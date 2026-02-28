# 环境配置

## OSS桶环境差异 ⚠️ 重要 [2025-02-26]

| 环境 | 桶名 | 访问地址 | 路径 |
|------|------|---------|------|
| **生产** | INC-DEBIT-ABS | http://inc-debit-abs-shenzhen-futian1.oss.sfcloud.local:8080 | /assetdata/fsp_data/YYYYMMDD.csv |
| **测试(SIT)** | INC-DEBIT-CCS | http://inc-debit-ccs-shenzhen-xili1.osssit.sfcloud.local:8080 | /assetdata/fsp_data/YYYYMMDD.csv |

**重点**：桶名和访问地址不同，但路径前缀相同！后续环境切换时要注意此差异。
