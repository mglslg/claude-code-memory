# AI Cache Memory

> 热记忆，每次会话必读，上限100行。
> ⚠️ **超100行时：把内容【剪切转移】到 ram/，禁止在此文件内删减内容！**

## 工作效率提示 ⚡ 重要 [2026-02-27]

**有MCP数据库配置，SQL查询直接用MCP调用！**
- ✅ 使用 mcp__mcp_server_mysql__mysql_query 直接查库
- ❌ 不要用 bash 执行 mysql 命令
- MCP调用效率更高、更简洁

---

## 🚨 严重问题：MCP时区处理 [2026-02-27]

**MCP返回的时间格式都是UTC（协调世界时），需要转换为北京时间（UTC+8）**

### 时间格式识别
- `2026-02-26T09:01:23Z` ← 末尾**Z = UTC时间**，需要+8小时
- `2026-02-26T17:01:23+08:00` ← 末尾**+08:00 = 北京时间**，无需转换
- `2026-02-26 17:01:23` ← 无标志 = 本地时间

### 处理规则（必须执行）
**任何查询结果，若时间末尾有 `Z`，立即加8小时转换为北京时间，不能直接用UTC值进行分析。**

转换公式：北京时间 = UTC时间 + 8小时

例如：
```
UTC时间：2026-02-23T16:00:00.000Z
北京时间：2026-02-24 00:00:00 ✓ 必须使用此值进行业务分析
```

### 问题严重性
- 如果不转换，会导致时间判断偏差整整1天
- 这会导致对业务逻辑（如快照表日期、封包日期等）的分析完全错误
- **教训：曾误判封包时间有bug，实际是时区问题**

---

## 灰度表清单 <@灰度><@V5> [2026-02-26]

**灰度表列表**（共10张）：asset_pack_info_v5, asset_pack_info_attribute_v5, asset_info_v5, asset_data_pack_snapshot_v5, asset_holder_chg_history_v5, asset_data_v5, asset_info_chg_flow_v5, asset_into_pack_snapshot_v5, asset_transfer_file_history_v5, asset_pack_holder_chg_history_v5

**隔离原则**：灰度表与正式表完全隔离，不能灰度表JOIN原表，灰度表之间可互相JOIN

## 🚨 手动vs自动封包数据一致性问题 <@数据一致性><@关键> [2026-02-27]

**原则：** 手动封包产生的数据已完全正常，自动封包必须与之100%贴合！

**严重后果：**
- 这些数据会被**大数据报表抽走**
- 任何字段差异都会**污染整个数据链路**
- 会导致报表统计错误、业务决策失误
- **一旦出问题，影响范围极大**

**因此：** 自动封包的数据形态必须与手动封包完全一致！

---

## ✅ 已修复字段清单 [2026-02-27]

1. ✅ **asset_pack_info_v5.ASSET_HOLDER** → AssetAutoPackServiceV5.java:218
2. ✅ **asset_pack_info_attribute_v5.exp_date_end** → AssetAutoPackServiceV5.java:548-557
3. ✅ **asset_info_v5.pack_date** → AssetInfoV5ExtMapper.xml batchInsertAssetInfoFlow
4. ✅ **asset_holder_chg_history_v5.AFTER_ASSET_STATUS** → BasePackImportService.java:779 <@灰度检验>

详细灰度验证方案已转移到 `ram/workflows.md` 修复4章节

---

## 🚨 资产债转与核销记录 <@债转><@核销><@资产归属><@陈乐康> [2026-02-28]

**业务规则**：资产自转让日期次日起归属受让人

**与陈乐康的原文对话**：

**提问**：
```
乐康，想问下。asset_holder_chg_history 这张表的数据你是怎么使用的了，有没有用到transfer_day这个字段？
```

**陈乐康回答**：
```
-- 当资产包持有人变更时，需要调整核销记录
-- 将原持有人的核销记录以负数形式反向记录
-- 确保账务平衡
```

**大数据报表使用**：大数据通过该表的变更记录识别债转发生时间

---

## 📚 项目QA知识库 <@QA> [2026-02-28]

已记录3个常见问题：
1. asset_data.UPDATE_TIME的刷新时间（FspDataMapping调度）
2. LOAN_BALANCE vs CAP_BALANCE的区别（无区别）
3. 逻辑删除的必要性（不多不需要关注）

详见 `ram/qa.md`
