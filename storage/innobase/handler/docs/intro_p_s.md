**`data_lock_inspector`** 是 MySQL 从 **8.0.27** 开始提供的一套 **用于观察 InnoDB 行级锁（record lock）和事务锁等待情况的视图**，属于 **Performance Schema** 的一部分。

它的设计目的是让开发者和DBA能更清晰地看到 **当前事务持有哪些锁、等待哪些锁、锁冲突的关系图**。

---

# ✅ `data_lock_inspector` 的核心功能

`data_lock_inspector` 并不是一个表，而是一组系统视图，作用主要是：

### **1. 显示当前所有 InnoDB 锁**

包括：

* 行锁（Record Lock）
* 间隙锁（Gap Lock）
* 下一键锁（Next-key Lock）
* 表锁（意向锁）

### **2. 显示事务之间的锁冲突**

例如：

* 哪个事务持锁？
* 哪个事务正在等待锁？
* 等待的原因是什么？

### **3. 清晰展示等待图（Wait-for Graph）**

能帮助你定位：

* 死锁是谁造成的
* 哪些事务互相阻塞
* 谁阻塞了别人

---

# 📚 `data_lock_inspector` 提供的主要视图

官方包含如下 6 个视图：

## **1. data_locks**

显示当前系统所有锁。
关键字段：

* `ENGINE`（InnoDB）
* `ENGINE_TRANSACTION_ID`
* `OBJECT_SCHEMA`
* `OBJECT_NAME`
* `INDEX_NAME`
* `LOCK_TYPE`
* `LOCK_MODE`

## **2. data_lock_waits**

显示锁等待关系（等待者 → 持有者）。

字段：

* `REQUESTING_ENGINE_TRANSACTION_ID`
* `BLOCKING_ENGINE_TRANSACTION_ID`

这张表最重要，能看到谁阻塞谁。

## **3. data_locks_nowait**

只显示 **当前正在阻塞的锁**。

## **4. data_locks_all**

包括持锁 + 等待锁的所有条目。

## **5. data_lock_pages**

显示锁所在的数据页（用于深度分析）。

## **6. data_lock_event**

显示锁和具体 SQL 事件关联。

---

# 🔎 常见用法示例

### **查看所有锁**

```sql
SELECT * FROM performance_schema.data_locks;
```

### **查看锁等待关系（重点）**

```sql
SELECT * FROM performance_schema.data_lock_waits;
```

输出示例（示意）：

| REQUESTING_TXN | BLOCKING_TXN |
| -------------- | ------------ |
| 12345          | 67890        |

表示 **事务 12345 正在被 67890 阻塞**。

---

# 🧠 和旧版 InnoDB Lock Monitor 的区别

旧的：

```sql
SHOW ENGINE INNODB STATUS;
```

输出：

* 很乱
* 难解析
* 一次查询很多信息

`data_lock_inspector`：

* 结构化 SQL 视图
* 方便 join 查询
* 更详细（Next-key、Gap 锁都能看到）
* 更利于分析死锁与性能瓶颈

---

# 📌 为什么会引入 `data_lock_inspector`？

因为 MySQL 8.0 之前 **没有办法直接看到行级锁**，只能靠猜。

8.0.27 后：

* 所有行锁都能看到
* 锁等待关系可视化
* 和事务、线程、SQL 事件可关联

这是非常重要的调优工具。

---

# 📝 总结

`data_lock_inspector` 是 MySQL 8.0.27+ 中 Performance Schema 的锁分析组件，用于：

✔ 展示所有 InnoDB 行级锁
✔ 显示事务间的锁等待
✔ 帮助分析死锁与阻塞问题
✔ 提供结构化 SQL 视图，替代冗长的 InnoDB STATUS

是现代 MySQL 锁调试的核心工具。

---

如果你愿意，我可以继续帮你写：
🔍 如何用 data_lock_inspector 查死锁
📊 data_lock_inspector + sys schema 联合分析
🧪 故障案例：两个事务互锁的完整分析
