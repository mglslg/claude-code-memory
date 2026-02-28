# 工作流程与已完成任务 [2026-02-27]

## 手动封包完整流程 <@手动封包><@回测> [2026-02-27]

### Step 1：生成预封包（generatePrePackageExcel）
```bash
curl -X POST http://localhost:8080/abs/rest/prePackage/generatePrePackageExcel \
  -H "Content-Type: application/json" \
  -d '{
    "expireDateTo": "2026-09-01",
    "holder": "深圳市顺丰合丰小额贷款有限公司",
    "inPoolBalanceMax": 200000,
    "includeSubKindNo": "50041,50042,500414",
    "loanSts": "DEFR,USED",
    "maxFiveLevel": "1,2",
    "overDays": "5",
    "packageAmt": 5000,
    "packageDate": "2026-02-24 00:00:00",
    "productType": "5004,5009",
    "projectAssetHolder": "中信证券-顺丰小贷第3期资产支持专项计划",
    "projectId": "1068",
    "sumGraceOvdueDaysMax": 30,
    "sumGraceOvdueTimesMax": 3,
    "sumGraceOvdueTimesMin": 0
  }'
```

### Step 2：下载预封包文件
```bash
curl -X GET "http://localhost:8080/abs/rest/prePackage/downloadPackage?fileName=PACKID.csv" \
  -o PACKID.csv
```
**说明**：PACKID为Step1生成的资产包ID（如2602260000）

### Step 3：导入封包（importPackage/import）
```bash
curl -X POST 'http://localhost:8080/abs/rest/importPackage/import' \
  -F 'file=@PACKID_import.csv' \
  -F 'forceMode=false'
```
**关键**：
- file参数：修改后的导入CSV文件（PACKID_import.csv）
- forceMode：false=严格模式，true=强制覆盖

---

- ✅ 对比生产数据与测试环境手动封包数据的字段结构 → 完全一致
- ✅ 识别MCP时区转换问题 → UTC需+8小时转北京时间
- ✅ 确认手动封包时间规律正确 → 快照表日期、操作日期符合预期
- ✅ 修复asset_pack_info_v5.ASSET_HOLDER字段（硬编码XFXD）
- ✅ 修复asset_pack_info_attribute_v5.exp_date_end日期格式（YYYY-MM-DD→YYYYMMDD）
- ✅ 修复asset_info_v5.pack_date NULL问题（添加SQL字段）
- ✅ 整理AI记忆，压缩冷数据到ram/

## 修复清单 [2026-02-27]

### 修复1：ASSET_HOLDER错误
**文件**：AssetAutoPackServiceV5.java:218
**原因**：自动封包使用了projectInfo.getAssetHolder()（受让方），应该用出让方
**修复**：硬编码`YgdLicenseTypeEnum.XFXD.getKey()`保证与手动封包一致

### 修复2：exp_date_end格式不一致
**文件**：AssetAutoPackServiceV5.java:548-557 buildAutoPackBean方法
**原因**：手动封包调用handleDateFmt转换YYYY-MM-DD→YYYYMMDD，自动封包未转换
**修复**：添加DateUtil.formatDate调用进行格式转换，包含异常处理

### 修复3：pack_date为NULL
**文件**：AssetInfoV5ExtMapper.xml batchInsertAssetInfoFlow语句
**原因**：代码调用assetInfo.setPackDate(now)但SQL INSERT语句未包含该字段
**修复**：添加PACK_DATE字段到INSERT字段列表和VALUES占位符列表，及ON DUPLICATE KEY UPDATE子句

## 修复4：AFTER_ASSET_STATUS状态值错误 <@灰度验证>

**文件**：BasePackImportService.java:779
**原因**：数据库status更新后，内存对象未同步，导致插入历史时使用旧值0
**修复**：添加`assetPackInfo.setStatus(AssetStatusEnum.ALREADY_PACK.typeCode);`同步内存值为1

**历史遗留问题**：
- 生产数据目前全部为0（错误状态，应该为1）
- 原因：BasePackImportService中更新数据库后未同步内存对象status

**修复影响说明**：
- ✅ 新增数据：灰度后的新records会正确记录为1
- ✅ 历史数据：保持为0（不修改旧数据）
- ✅ 系统影响：零影响，该字段仅作审计记录，无业务逻辑依赖

**灰度期间验证方案**：
```sql
-- 检查灰度新增记录是否正确变为1
SELECT COUNT(*) as cnt, AFTER_ASSET_STATUS
FROM asset_holder_chg_history_v5
WHERE CREATE_TIME >= '灰度上线时间'
GROUP BY AFTER_ASSET_STATUS;
-- 预期：新增记录的AFTER_ASSET_STATUS = 1
```

**验证目标**：新增记录AFTER_ASSET_STATUS = 1（旧数据保持0）
**验证周期**：灰度上线后3-7天执行验证
**风险等级**：零风险，纯数据一致性修复，无业务逻辑影响

---
