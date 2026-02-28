# 项目QA文档 [2026-02-28]

## Q1: asset_data表的UPDATE_TIME何时更新？ <@asset_data><@调度>

**问题**：`select t.ASSET_HOLDER,t.CREATE_TIME,t.UPDATE_TIME from asset_data t where t.SUB_NAME is null;` 中的UPDATE_TIME什么时候被写入的？每天早上8:00都会被刷新？

**答案**：早上8:00由FspDataMapping调度任务刷新

**含义**：asset_data表中SUB_NAME为null的记录，每天早上8:00会被FspDataMapping调度重新更新时间戳

---

## Q2: LOAN_BALANCE vs CAP_BALANCE的区别？ <@asset_data><@字段>

**问题**：`select t.LOAN_BALANCE,t.CAP_BALANCE from asset_data t where t.SUB_NAME='slg测试项目1' order by t.UPDATE_TIME desc;` 这两个字段有什么区别？

**答案**：没有区别，生产数据两个字段的值完全一样

**含义**：LOAN_BALANCE和CAP_BALANCE在生产环境中是一致的，可以互换使用

---

## Q3: 系统逻辑删除的必要性 <@删除策略><@架构>

**问题**：系统的逻辑删除到底有多少？有哪些是逻辑删除、哪些是真实删除？逻辑删除是否有必要？

**答案**：逻辑删除的数量不多，不用太在意，不是重点关注的内容

**含义**：
- 逻辑删除用得不多
- 逻辑删除的必要性不大
- 不需要花力气去优化或重构删除策略

---
