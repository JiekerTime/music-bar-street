# 数据设计

以 PostgreSQL 为业务数据来源。下面是实现前的表设计，不是已经执行过的迁移文件。主键建议用 UUID；金额用整数分，时间点用 timestamptz，同时保留双方约定的时区。

[数据关系图](../design/data-model.svg) · [双方确认时序](../design/confirmation-sequence.svg)

## 身份、资料和媒体

| 表 | 主要字段 | 约束 |
| --- | --- | --- |
| accounts | id, phone, status, created_at | phone 唯一；停用后不能发起新合作 |
| venue_profiles | id, account_id, name, city, address, description, license_media_id, review_status | 一期 account_id 唯一；资质资源为私有 |
| artist_profiles | id, account_id, name, city, styles, introduction, rate_unit, rate_minor, review_status | 一期 account_id 唯一；报价保留计价单位 |
| artist_calendar_entries | id, artist_id, starts_at, ends_at, kind, note | starts_at < ends_at；kind 区分不可用和可用说明 |
| media_assets | id, owner_account_id, purpose, provider, provider_id, access_level, upload_status, review_status, size_bytes, mime_type | 供应商资源标识唯一；用途和所有者不能由客户端事后改写 |
| artist_media | artist_id, media_id, sort_order | 组合唯一；只能绑定本人且用途正确的资源 |

账号可以具备酒吧和艺人两种资料，当前操作身份与权限由服务端核对。多人管理一个酒吧可在以后增加成员关系，不用提前建设团队权限系统。

## 从需求到订单

| 表 | 主要字段 | 约束 |
| --- | --- | --- |
| demands | id, venue_id, starts_at, ends_at, timezone, budget_minor, artist_type, requirements, status, version | 时间和金额有效；version 用于防止旧页面覆盖修改 |
| applications | id, demand_id, artist_id, message, status | demand_id + artist_id 唯一；重复投递返回已有记录 |
| invitations | id, venue_id, artist_id, proposed_terms, status, expires_at | 艺人只能回应发给自己的邀约 |
| negotiations | id, venue_id, artist_id, application_id, invitation_id, current_terms_version, status, conversation_id | application_id / invitation_id 必须且只能有一个非空；来源唯一 |
| agreement_versions | id, negotiation_id, version, starts_at, ends_at, timezone, price_unit, quantity, unit_price_minor, total_minor, currency, requirements, cancellation_terms | negotiation_id + version 唯一；版本保存后不可覆盖 |
| agreement_confirmations | id, agreement_version_id, account_id, party, confirmed_at | agreement_version_id + party 唯一；操作者必须是这次合作对应一方 |
| performance_orders | id, negotiation_id, agreement_version_id, venue_id, artist_id, terms_snapshot, status, version, completed_at | negotiation_id 唯一；只引用双方确认过的同一版本 |
| artist_reservations | id, order_id, artist_id, period, active | order_id 唯一；有效占用不能重叠 |
| order_events | id, order_id, actor_id, event_type, payload, occurred_at | 追加记录；保存完成、取消、变更和争议结果 |

agreement_versions 记录谈定的条件，demands.budget_minor 仍是原预算。订单里的快照另外保留双方当时的名称、场地与条件，避免后来修改资料改变历史订单。

需求关闭或已约满后，不能新投递。已有洽谈在第二方确认时仍要复查需求、账号状态与剩余名额。一个需求一期按一份演出订单处理；如果要多位艺人，需要先增加名额规则。

## 消息、会员和费用

| 表 | 主要字段 | 约束 |
| --- | --- | --- |
| conversations | id, negotiation_id, venue_account_id, artist_account_id, status | negotiation_id 唯一；双方是唯一普通成员 |
| messages | id, conversation_id, sender_id, client_message_id, body, created_at | conversation_id + sender_id + client_message_id 唯一；消息内容不可由另一方改写 |
| read_cursors | conversation_id, account_id, last_message_id | 组合唯一；只能更新自己的游标 |
| membership_plans | id, name, price_minor, duration_days, entitlements, active | 套餐变化不改已经购买的权益快照 |
| membership_orders | id, venue_id, plan_snapshot, amount_minor, status, provider_payment_id, paid_at | 支付标识唯一；待支付不能生效 |
| memberships | id, venue_id, membership_order_id, entitlements_snapshot, starts_at, ends_at, status | 购买订单唯一；时间有效；到期按时间计算 |
| fee_records | id, order_id, basis_minor, rate_bps, estimated_minor, receivable_minor, received_minor, status | order_id 唯一；统计、应收与实收分别保存 |
| audit_logs | id, operator_id, action, resource_type, resource_id, reason, created_at | 追加记录；不保存密钥、验证码和完整私密内容 |

会员表是购买和权益的设计；实际支付服务商和价格未定，暂不接真实收款。服务费计算比例保存在每单快照里，后续改比例不能重算历史订单。

## 必须放进事务的操作

第二方确认时，锁定 negotiation 和当前条件版本，确认请求携带的版本仍有效。保存确认记录、创建订单、占用艺人档期、关闭需求的剩余名额、追加订单事件在同一个事务里完成。

如果档期冲突，这次事务整体回滚，不能留下“已确认但没有档期”的订单。并发的第二次确认要返回已存在的同一份订单。

订单完成、取消、争议裁定和双方同意的变更，同样在事务中更新状态、档期、费用状态和事件。变更既要保留旧快照，也要重新检查新档期；失败时旧订单和旧占用继续有效。

## 索引和约束

- demand：按状态、城市和开始时间筛选；application：需求与艺人唯一索引。
- negotiation：来源唯一；agreement_version：洽谈与版本号唯一；order：洽谈唯一。
- messages：conversation_id + id 游标索引；order_events：order_id + occurred_at 索引。
- reservations：使用 `[starts_at, ends_at)` 范围，首尾相接不算冲突；有效记录使用 GiST 排斥约束。

PostgreSQL 支持 tstzrange 和组合排斥约束，标量 UUID 与范围组合可用 btree_gist。实现示例：

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;
ALTER TABLE artist_reservations
  ADD CONSTRAINT no_active_artist_overlap
  EXCLUDE USING gist (artist_id WITH =, period WITH &&)
  WHERE (active);
```

这是约束示例，实施时还要建表、非空约束和范围有效性检查。[PostgreSQL 官方范围类型说明](https://www.postgresql.org/docs/17/rangetypes.html)解释了范围及排斥约束的用法。
