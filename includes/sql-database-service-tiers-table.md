<!--
Used in:
sql-database-performance-guidance.md  
sql-database-resource-limits.md
sql-database-service-tiers.md  
-->

### <a name="basic-service-tier"></a>基本服务层
| **性能级别** | **基本** |
| --- | :---: |
| 最大 DTU | 5 |
| 最大数据库大小* |2 GB|
| 最大内存 OLTP 存储 |不适用 |
| 最大并发辅助进程数 |30 |
| 最大并发登录数 |30 |
| 最大并发会话数 |300 |
|||

### <a name="standard-service-tier"></a>标准服务层
| **性能级别** | **S0** | **S1** | **S2** | **S3** |
| --- |---:| ---:|---:|---:|---:|
| 最大 DTU | 10 | 20 | 50 | 100 |
| 最大数据库大小* | 250 GB| 250 GB | 250 GB | 250 GB |
| 最大内存 OLTP 存储 | 不适用 | 不适用 | 不适用 | 不适用 |
| 最大并发辅助进程数 | 60 | 90 | 120 | 200 |
| 最大并发登录数 | 60 | 90 | 120 | 200 |
| 最大并发会话数 |600 | 900 | 1200 | 2400 |
||||||

### <a name="premium-service-tier"></a>高级服务层 
| **性能级别** | **P1** | **P2** | **P4** | **P6** | **P11** | **P15** | 
| --- |---:|---:|---:|---:|---:|---:|
| 最大 DTU | 125 | 250 | 500 | 1000 | 1750 | 4000 |
| 最大数据库大小* | 500 GB | 500 GB | 500 GB | 500 GB | 4 TB* | 4 TB* |
| 最大内存中 OLTP 存储 | 1 GB | 2 GB | 4 GB | 8 GB | 14 GB | 32 GB |
| 最大并发辅助进程数 | 200 | 400 | 800 | 1600 | 2400 | 6400 |
| 最大并发登录数 | 200 | 400| 800| 1600| 2400| 6400 |
| 最大并发会话数 | 30000| 30000| 30000| 30000| 30000| 30000 |
|||||||

### <a name="premium-rs-service-tier"></a>高级 RS 服务层 
| **性能级别** | **PRS1** | **PRS2** | **PRS4** | **PRS6** |
| --- |---:|---:|---:|---:|---:|---:|
| 最大 DTU 数 | 125 | 250 | 500 | 1000 |
| 最大数据库大小* | 500 GB | 500 GB | 500 GB | 500 GB |
| 最大内存中 OLTP 存储 | 1 GB | 2 GB | 4 GB | 8 GB |
| 最大并发辅助进程数 | 200 | 400 | 800 | 1600 |
| 最大并发登录数 | 200 | 400| 800| 1600|
| 最大并发会话数 | 30000| 30000| 30000| 30000|
|||||||

\*最大数据库大小是指数据库中数据的最大大小。