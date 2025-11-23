**I_S 表（INFORMATION_SCHEMA 表）本身不存储“真正的数据”**，它是 **MySQL 服务器动态生成的一组虚拟系统表**，用于展示：

📌 **数据库的元数据（metadata）**
📌 **引擎内部状态（InnoDB / MyISAM 等）**
📌 **性能统计信息（部分是从 Performance Schema 衍生）**

也就是说：
**I_S 表是“读的时候现算出来的”，不占物理磁盘空间，也不会持久化。**

---

# ⭐ I_S 表的几大分类（你必须弄清楚）

INFORMATION_SCHEMA 的内容可以按来源分成 **三大类**：

---

# **① 元数据表（Metadata Tables）**

这些表记录数据库结构，由 MySQL 系统直接维护：

| I_S 表          | 内容是什么                |
| -------------- | -------------------- |
| **SCHEMATA**   | 数据库列表                |
| **TABLES**     | 每个库的表、表类型、行数估计、文件大小等 |
| **COLUMNS**    | 每个表的列字段结构            |
| **STATISTICS** | 索引信息                 |
| **TRIGGERS**   | 触发器信息                |
| **ROUTINES**   | 存储过程、函数信息            |
| **VIEWS**      | 视图定义                 |

用途：查看数据库结构、备份前的分析、DDL 可视化。

---

# **② InnoDB 内部状态（InnoDB-specific I_S 表）**

这些是你刚才发的头文件 `i_s.h` 实现的表，来自 InnoDB。

它们展示 InnoDB **内部数据结构**，例如：

| I_S 表                      | 存放内容                              |
| -------------------------- | --------------------------------- |
| **INNODB_TRX**             | 活跃事务：trx_id、事务状态、锁信息、undo log     |
| **INNODB_LOCKS**（旧版）       | 事务锁记录（后来被 data_lock_inspector 取代） |
| **INNODB_BUFFER_PAGE**     | Buffer Pool 中每个 page 的类型、LSN 等    |
| **INNODB_BUFFER_PAGE_LRU** | BP LRU 链表结构                       |
| **INNODB_TABLES**          | InnoDB 表信息（空间 ID、row format、统计信息） |
| **INNODB_INDEXES**         | 索引 B+ 树元信息                        |
| **INNODB_COLUMNS**         | InnoDB 内部列结构（比 COLUMNS 更底层）       |
| **INNODB_TABLESPACES**     | 表空间元数据                            |
| **INNODB_FT_CONFIG**       | 全文索引（FTS）配置                       |
| **INNODB_METRICS**         | InnoDB 各组件的计数器（类似 P_S，但更底层）       |

📌 **重点：这些表数据全部来自 InnoDB 内存结构，不写磁盘，不持久化。**

常用于：

* 锁分析
* Buffer pool 可视化
* 表空间结构分析
* 索引结构理解
* FTS 搜索调试

---

# **③ 服务器统计类（Server-level Info）**

来自 MySQL server 层，而不是引擎层：

| I_S 表                           | 内容                     |
| ------------------------------- | ---------------------- |
| **GLOBAL_STATUS**               | SHOW GLOBAL STATUS 同源  |
| **GLOBAL_VARIABLES**            | 服务器配置项                 |
| **USER_PRIVILEGES**             | 权限系统                   |
| **PROCESSLIST**                 | 所有线程的当前状态（旧版使用）        |
| **CHARACTER_SETS / COLLATIONS** | 字符集与比较规则               |
| **ENGINES**                     | 所有存储引擎状态               |
| **PLUGINS**                     | 插件注册情况                 |
| **OPTIMIZER_TRACE**             | 查询优化器的执行路径（8.0 默认 off） |

这些基本是 server 层 API 动态生成的。

---

# 🔥 总结一句话

**INFORMATION_SCHEMA = 数据库结构 + InnoDB 内部结构 + MySQL server 内部状态（全部是虚拟表）**

没有一条数据是持久化的，都是 SQL 查询时实时从引擎内存结构“渲染”出来的。

---

# 🌟 你可能更关心的一点

### **I_S 跟 Performance Schema 有何区别？**

| 项目   | INFORMATION_SCHEMA         | PERFORMANCE_SCHEMA     |
| ---- | -------------------------- | ---------------------- |
| 类型   | 元数据、结构、状态                  | 性能事件、高频统计              |
| 数据来源 | MySQL server & InnoDB 内部结构 | 预定义的内存 instrumentation |
| 是否动态 | 是 – 读时生成                   | 是 – 事件采样计数器            |
| 开销   | 较低                         | 可以配置开销（默认低）            |

Performance Schema 能追踪：

* SQL 延迟
* Lock Wait 事件
* IO 事件
* Thread 事件

但 I_S 更多是结构、元数据视角。

---

# 📘 你现在研究的 InnoDB I_S 表，到底有什么特点？

你看到的 `i_s.h` 声明的那些表：

✔ 不是真实存储的表
✔ 也不会写磁盘
✔ 是通过插件（plugin）动态注册
✔ 是 InnoDB 内存数据结构的只读视图
✔ SQL 层只负责展示，数据由 InnoDB handler 填充

类似于“把 InnoDB 内核的数据结构暴露成 SQL 表”。

---
