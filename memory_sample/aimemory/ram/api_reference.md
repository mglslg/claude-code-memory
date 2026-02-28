# API参考

## 核心测试接口 ⭐⭐⭐ [2025-02-26]

| 接口路径 | 类位置 | 功能 | 数据版本 |
|---------|-------|------|---------|
| POST /rest/sysmaintain/doSnapshotSync | com.sf.fintech.abs.sys.web.SysMaintainController#doSnapshotSync | 创建快照表、同步borrower_info、同步fsp_loan_info | 通用 |
| POST /rest/sysmaintain/doFspDataMappingJob | com.sf.fintech.abs.sys.web.SysMaintainController#doFspDataMappingJob | **同步旧数据** asset_data、备份、刷新产品小类 | V4（正式表） |
| POST /rest/sysmaintain/doFspAssetDataSyncV5Job | com.sf.fintech.abs.sys.web.SysMaintainController#doFspAssetDataSyncV5Job | **同步灰度新版本** asset_data_v5 | V5（灰度表） |

**核心区别**：
- doFspDataMappingJob 更新的是 **asset_data（正式表）**
- doFspAssetDataSyncV5Job 更新的是 **asset_data_v5（灰度表）**
- 两个接口互不影响，灰度期间需要分别运行

---

## DataSynchronizeService执行步骤 ⚠️ 已变更 [2025-02-26]

快照同步调度DataSynchronizeJob中，autoImportData()方法执行的步骤**仅有3步**：

1. **batchInsert** - 从OSS CSV批量插入asset_snapshot_yyyymmdd快照表
2. **updFromBorrowerInfo** - 从borrower_info同步QUALITY_LEVEL风险标签到快照表
3. **updFromFspLoanInfo** - 从fsp_loan_info同步4期新增的10个字段

**已废弃**：updFromAssetData
- 原因：asset_snapshot表中ASSET_HOLDER字段完全没被用到
- 为了避免v5灰度期间与asset_data表的数据不一致，完全剥离对asset_data的依赖
- 代码位置：DataSynchronizeServiceImpl.java:515-521（已注释）
