# 数据库运维/DBA 手册（Vibe Coding + OpenSpec）

面向 DBA 的独立手册，覆盖 MySQL、PostgreSQL、Oracle、人大金仓（KingbaseES）、达梦（DM）。聚焦日常巡检、变更、备份/恢复与性能排查，可结合 Codex/Claude/Cursor 快速生成脚本与 Runbook。

## 前置
- 读 `openspec/AGENTS.md`、`openspec/project.md`；用 `openspec list` 了解变更与影响的数据库。
- 收集拓扑与关键信息：版本/主备/HA 方式、端口、角色账户、备份/归档位置、RPO/RTO。
- 本地可用客户端：`mysql`、`psql`、`sqlplus`、`ksql`、`disql` 及对应备份工具（mysqldump/pg_dump/RMAN/ks_dump/dmrman 等）。
- 全程记录：执行窗口、命令、耗时、产物位置、验证结果、回滚入口。

## 协作工作流（建议顺序）
- AI 协同：每一步要求助手先列假设/风险与验证方式，再给命令或补丁；输出需包含窗口/阈值/回滚口令，优先在从库或备份演练。
1) 对齐需求与风险  
   - 提问：“总结 <change-id>/<库名> 的变更、受影响表/索引、峰值时段、允许的停机/锁表时间。”
2) 建立基线与报警线  
   - 输出：连接/复制状态、QPS/TPS、锁等待、表空间/磁盘、关键参数与慢 SQL TopN。
   - AI 协同：让助手给出采集命令及阈值，并标注需人工确认的基线。
3) 备份策略确定  
   - 选择物理/逻辑备份，标注 RPO/RTO、备份路径、校验方式（checksum、恢复演练窗口）。
   - AI 协同：要求输出完整命令+校验步骤，并附恢复演练计划。
4) 变更脚本评审  
   - 在从库/备份副本演练；使用 `EXPLAIN`/`EXPLAIN ANALYZE`，必要时采用在线变更工具（pt-osc/gh-ost、分批 DDL）。
   - AI 协同：让助手先估算锁表/时长与影响面，再生成最小可回退的脚本。
5) 执行与监控  
   - 在维护窗口执行；实时观察复制延迟、锁/等待、IO/CPU；超阈值立即暂停并进入回滚预案。
   - AI 协同：要求输出观测命令与阈值表，异常时生成回滚/降级步骤。
6) 验证与交付  
   - 验证数据量/行数/校验 SQL、业务关键查询耗时、复制/HA 恢复；更新 Runbook 与交付材料。
   - AI 协同：让助手生成“验证脚本 + 对账清单”，并列未决风险。

## 通用检查与操作模板
- 连接探活：确认主备均可连，认证、TLS、超时配置正确。
- 复制/容灾：查看复制延迟、同步位点、归档/日志堆积，必要时预热/解压归档。
- 存储/容量：表空间/磁盘使用率、膨胀（膨胀表/索引统计）、自动扩容策略。
- 备份/恢复：标记类型（全量/增量/归档）、保留期、校验方式；定期演练恢复到新实例。
- DDL/Schema 变更：上线前在备份副本跑 schema diff、估算锁时间；MySQL 优先在线 DDL，PG 关注长事务与锁冲突。
- 慢 SQL/性能：开启慢日志或等效统计（MySQL slow log、pg_stat_statements、AWR/TKProf、DM/Kingbase 动态视图）；限制查询重写在单文件补丁内。
- 安全/权限：最小权限账户、审计开关、默认账户处理、密码/证书轮转。

## 数据库速查

### MySQL
- 连接与健康：`mysql -h <host> -P 3306 -u <user> -p<pass> -e "SHOW GLOBAL STATUS LIKE 'Threads_connected';"`。
- 复制：`SHOW REPLICA STATUS\G`（或 `SHOW SLAVE STATUS\G`），主库位点 `SHOW MASTER STATUS;`。
- 慢 SQL：开启 `slow_query_log=ON`，设置 `long_query_time`，汇总 `mysqldumpslow -s t <slow.log>`。
- 备份/恢复：逻辑 `mysqldump --single-transaction --master-data=2 <db> > backup.sql`；大库用物理备份（如 xtrabackup），恢复后跑 `CHECKSUM TABLE` 或比对行数。
- 在线变更：优先 `pt-online-schema-change`/`gh-ost`；释放长事务后再执行 DDL。

### PostgreSQL
- 连接：`psql "host=<host> port=5432 dbname=<db> user=<user> sslmode=require"`。
- 复制/Lag：`select application_name,state,sync_state,write_lag,replay_lag from pg_stat_replication;`，从库延迟 `select now() - pg_last_xact_replay_timestamp();`。
- 统计/维护：`ANALYZE <table>; VACUUM (VERBOSE, ANALYZE) <table>;`，碎片严重时 `VACUUM FULL`（需维护窗口）。
- 慢 SQL：启用 `pg_stat_statements`，`select query, total_time, calls from pg_stat_statements order by total_time desc limit 20;`；必要时打开 auto_explain。
- 备份/恢复：逻辑 `pg_dump -Fc -f db.dump <db>` / `pg_restore -d <db> db.dump`；物理 `pg_basebackup -D /data/pg_base -X stream -P`。

### Oracle
- 连接：`sqlplus <user>/<pass>@//<host>:1521/<service>`。
- 健康：`select instance_name,status from v$instance;`，`select name,open_mode,database_role from v$database;`，表空间 `select tablespace_name, used_percent from dba_tablespace_usage_metrics;`。
- 计划/性能：`explain plan for <SQL>;` 后 `select * from table(dbms_xplan.display());`；抓取 AWR/ASH、TKProf 分析。
- 备份：RMAN `backup database plus archivelog;`，校验 `list backup summary;`；逻辑导出 `expdp <user>/<pass> directory=DATA_PUMP_DIR dumpfile=exp.dmp logfile=exp.log schemas=<schema>`，恢复用 `impdp`。
- 变更：在备库演练并确认归档/闪回可用，必要时启用 DDL Lock Timeout。

### 人大金仓（KingbaseES）
- 说明：PG 兼容，默认端口常为 54321，常用客户端 `ksql`，运维视图与 PG 类似。
- 连接：`ksql -h <host> -p 54321 -U <user> -d <db>`。
- 复制/Lag：复用 `pg_stat_replication`、`pg_is_in_recovery()`，延迟判定与 PG 相同。
- 维护与性能：`VACUUM`/`ANALYZE`、`EXPLAIN (ANALYZE, BUFFERS)`；慢 SQL 可启用 `log_min_duration_statement` 或 `pg_stat_statements`。
- 备份/恢复：逻辑备份使用官方工具（如 `ks_dump`/`ks_restore`，与 `pg_dump`/`pg_restore` 用法相似）；物理备份按官方容灾/归档工具执行并记录恢复窗口。

### 达梦（DM）
- 连接：`disql SYSDBA/<pass>@<host>:5236`（按实际端口/账户调整）。
- 健康：`select * from v$instance;`，存储查看 `select * from v$datafile;`，归档模式 `select * from v$dm_ini where para_name='ARCH_INI';`。
- 备份/恢复：`dmrman` 全量备份 `backup database full to '/dm/backup';`，校验 `list backupset;`；逻辑导出 `dexp user/<pass> file=exp.dmp log=exp.log owner=<schema>`，导入用 `dimp`。
- 性能：可结合 `sp_btrace`/`v$sql_area` 查看热点 SQL；长事务、锁信息可查 `v$lock` 等视图。

## 常用提示语
- “生成 <DB>/<实例> 的巡检清单与命令，覆盖连接、复制、表空间/磁盘、慢 SQL TopN、审计/安全。”
- “为 <DDL/变更> 生成上线方案：评估锁表时间、备份与回滚步骤、监控指标与阈值、验证 SQL。”
- “把 <备份类型/路径> 的恢复演练写成 Runbook，含耗时预估、校验命令、失败时的兜底措施。”

## 交付清单
- 基线与变更前后对比：连接/复制/容量/性能指标。
- 备份/恢复记录：命令、路径、校验结果、RPO/RTO 证明。
- 变更脚本与执行日志：包含窗口、耗时、影响范围、回滚入口。
- 验证结果与未决风险：业务关键查询、行数/校验 SQL、剩余告警。
