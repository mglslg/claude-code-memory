# 踩坑记录

## updFromAssetData 只同步 ASSET_HOLDER，不同步 MAX_FIVE_LEVEL [2025-02-26]

**错误假设**：我曾错误地认为 MAX_FIVE_LEVEL 是从 updFromAssetData 同步的

**真实情况**：updFromAssetData 的XML代码如下：
```xml
<update id="updFromAssetData">
    UPDATE ${tableName} bi
    INNER JOIN asset_data ad ON bi.LOAN_NO = ad.LOAN_NO
    AND ad.CAP_SOURCE='FSP'
    SET bi.ASSET_HOLDER = ad.ASSET_HOLDER
</update>
```
只同步 ASSET_HOLDER！

**教训**：
- ❌ 不要在没有验证代码的情况下假设字段来源
- ✅ 始终要看实际的SQL/XML代码，不能凭印象说话
- ✅ 遇到同步问题，先检查源数据有没有，再检查SQL逻辑是否正确

---

## asset_info.pack_date 的"错误"设计 <@asset_info><@pack_date><@设计缺陷> [2026-02-27]

**现象**：asset_info表中pack_date字段的统计口径与其他所有表不一致
- asset_pack_info.pack_date = T-1（预封包选择的日期）
- asset_info.pack_date = T 日（当天操作日期，NOT T-1）
- 其他所有表.pack_date = T-1

**根本原因**：原本应该写成T-1（与其他表保持一致），但实际代码写成了T日

**业务缘何成立**：大数据系统恰好把asset_info.pack_date当作操作日D的锚点，用D-1来筛选其他数据，虽然"错了"但反而能对上！吴欣建确认无"指定某日封包"场景，仅有"选择前一天"场景，因此这个Bug从未暴露。

**当前决策**：保持现状，"将错就错"，以保证与历史数据的一致性

**⚠️ 警告**：涉及自动封包、跨版本迁移、数据修复场景时，这个细节会成为关键线索！
