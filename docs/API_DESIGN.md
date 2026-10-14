# 接口约定

以下是一期业务 API 的设计，不是现成的服务地址。统一前缀 `/api/v1`，JSON 用 camelCase。公开主键为 UUID，金额是整数分，日期时间用带偏移的 RFC 3339，并保留 timezone。

## 通用规则

需要登录的请求带 `Authorization: Bearer <session>`，会话在服务端核对。公开浏览接口只返回已公开且审核通过的数据。操作者 ID 从会话取得，不能让请求体替用户指定发起方。

成功结果含 data 与 requestId；失败含 error.code、error.message、error.details 与 requestId。列表使用 cursor、limit，默认 20、最大 100，返回 nextCursor，排序固定并带 ID 作为末级排序。

创建投递、邀约、确认、消息、订单动作和会员购买意向带 `Idempotency-Key`。服务端按账号、操作、幂等键和请求摘要保存结果；同键同内容返回原结果，同键不同内容返回 409。幂等记录保留时间由实现配置，至少覆盖客户端重试周期；唯一约束仍作为最终防线。

修改资料和需求携带 expectedVersion。修改合作条件创建新版本，不能覆盖旧版本。确认携带 expectedTermsVersion；订单变更携带 expectedOrderVersion。

## 登录、资料和审核

| 方法与路径 | 输入 | 结果和规则 |
| --- | --- | --- |
| POST /auth/challenges | phone, purpose | challengeId, expiresAt；频率限制，不回显验证码 |
| POST /auth/sessions | challengeId, code | session, expiresAt, account；短信服务商待选 |
| DELETE /auth/session | 无 | 当前会话失效 |
| GET /me | 无 | 自己的身份、账号和资料状态 |
| PUT /me/venue | expectedVersion, name, city, address, description, licenseMediaId | 自己的酒吧资料；私有证件不回到公开接口 |
| PUT /me/artist | expectedVersion, name, city, styles, introduction, rateUnit, rateMinor, mediaIds | 自己的艺人资料；媒体用途和所有者核对 |
| POST /me/venue/review | expectedVersion | 提交酒吧审核 |
| POST /me/artist/review | expectedVersion | 提交艺人审核 |
| GET /artists | city, style, cursor, limit | 公开艺人列表；不返回手机号和私有资料 |
| GET /artists/{id} | 无 | 公开艺人主页和已可用作品 |
| GET /venues/{id} | 无 | 公开酒吧主页 |
| GET /me/artist/calendar | from, to | 本人档期说明与订单占用 |
| POST /me/artist/calendar | startsAt, endsAt, kind, note | 增加本人档期说明；不覆盖订单占用 |
| DELETE /me/artist/calendar/{id} | expectedVersion | 仅删除本人手工录入的说明 |

## 需求、投递和邀约

| 方法与路径 | 输入 | 结果和规则 |
| --- | --- | --- |
| GET /demands | city, style, from, to, cursor, limit | OPEN 且可公开的需求 |
| POST /demands | startsAt, endsAt, timezone, budgetMinor, artistType, requirements | 本人酒吧的需求；审核、会员额度和时间检查 |
| PATCH /demands/{id} | expectedVersion, 可修改字段 | 本人且仍可修改的需求；不能改已确认订单 |
| POST /demands/{id}/close | expectedVersion, reason | 关闭本人需求，停止新投递 |
| POST /demands/{id}/applications | message | 当前艺人的投递；已有记录时返回同一记录 |
| GET /demands/{id}/applications | cursor, limit | 需求所属酒吧可查看投递 |
| POST /applications/{id}/withdraw | reason | 当前投递艺人撤回 |
| POST /applications/{id}/select | 无 | 需求所属酒吧建立或返回洽谈 |
| POST /applications/{id}/reject | reason | 需求所属酒吧拒绝投递 |
| POST /invitations | artistId, proposedTerms | 本人酒吧发起邀约；邀请不是订单 |
| GET /me/invitations | direction, cursor, limit | 自己发出或收到的邀约 |
| POST /invitations/{id}/respond | decision: ACCEPT / REJECT, message | 收件艺人回应；接受时建立洽谈 |
| POST /invitations/{id}/withdraw | reason | 发件酒吧撤回未完成邀约 |

## 条件和订单

| 方法与路径 | 输入 | 结果和规则 |
| --- | --- | --- |
| GET /negotiations/{id} | 无 | 参与方可读当前条件、确认情况、消息会话 |
| POST /negotiations/{id}/agreement-versions | expectedTermsVersion, terms | 参与方提新条件；清空当前确认，旧版本保留 |
| POST /negotiations/{id}/confirm | expectedTermsVersion | 记录本人确认；第二方确认时事务内创建订单并占用档期 |
| POST /negotiations/{id}/cancel | reason | 订单前终止洽谈并留记录 |
| GET /me/orders | status, cursor, limit | 自己参与的订单 |
| GET /orders/{id} | 无 | 参与方订单快照、状态和历史事件 |
| POST /orders/{id}/complete | expectedOrderVersion | 演出结束后记录本人完成；另一方确认后完成 |
| POST /orders/{id}/cancellation-requests | expectedOrderVersion, reason | 发起取消请求；暂不释放档期 |
| POST /orders/{id}/cancellation-requests/{requestId}/respond | decision, reason | 另一方回应；同意后取消，分歧进入争议 |
| POST /orders/{id}/change-requests | expectedOrderVersion, terms, reason | 提出改期或改价，不直接改订单 |
| POST /orders/{id}/change-requests/{requestId}/respond | decision | 另一方同意后事务内替换当前版本与档期，保留旧记录 |
| POST /orders/{id}/disputes | expectedOrderVersion, reason | 记录争议，不自动扣款或裁定 |

取消、变更和争议的子记录可使用 order_events 保存提案和回应，具体表拆分可在实现时确定。每个提案有独立 UUID，只允许回应尚待回应的同一提案；过期订单版本不能覆盖新结果。

一次确认请求：

```json
{"expectedTermsVersion": 3}
```

一方已确认、另一方还没确认：

```json
{
  "data": {
    "termsVersion": 3,
    "confirmations": {"venue": true, "artist": false},
    "order": null
  },
  "requestId": "example-request-1"
}
```

双方确认后返回同一份订单 ID。档期冲突返回 409 ARTIST_UNAVAILABLE，整次事务回滚。

一份条件示例：

```json
{
  "startsAt": "2026-11-07T23:00:00+08:00",
  "endsAt": "2026-11-08T01:00:00+08:00",
  "timezone": "Asia/Shanghai",
  "priceUnit": "PER_GIG",
  "quantity": "1",
  "unitPriceMinor": 120000,
  "totalMinor": 120000,
  "currency": "CNY",
  "requirements": "双人演出，场地提供音响；彩排时间另行确认。",
  "cancellationTerms": "取消与改期先协商，结果保存在订单记录中。"
}
```

quantity 是最多两位小数的正十进制字符串；服务端用十进制计算 unitPriceMinor × quantity，四舍五入到分，核对 totalMinor。PER_GIG 的 quantity 按场数，PER_HOUR 按已约定收费小时数，不能在 UI 里静默切单位。

## 媒体、消息、会员与运营

| 方法与路径 | 输入 | 结果和规则 |
| --- | --- | --- |
| POST /media/upload-intents | purpose, filename, contentType, sizeBytes | mediaId, 短期上传授权；客户端不传目标 Bucket |
| POST /media/{id}/upload-complete | 无 | 核对对象后更新本人资源上传状态 |
| GET /media/{id}/access | 无 | 根据公开状态或本人/审核权限返回短期地址或播放授权 |
| POST /callbacks/media/{provider} | 供应商事件 | 核对来源与事件 ID，重复处理返回原结果 |
| GET /conversations/{id}/messages | cursor, limit | 仅参与方，按游标读取 |
| POST /conversations/{id}/messages | clientMessageId, body | 仅参与方发文字消息，长度 1–2000 字符 |
| PUT /conversations/{id}/read-cursor | lastMessageId | 只更新自己读到的会话消息 |
| GET /membership/plans | 无 | 套餐、权益及当前购买方式 |
| POST /membership/purchase-intents | planId | 未接支付时登记意向；不能假装开通会员 |
| GET /me/membership | 无 | 本人酒吧的权益和有效期 |
| POST /membership/orders | planId | 支付接通后才启用，服务端锁定价格快照 |
| POST /callbacks/payments/{provider} | 供应商支付事件 | 启用后核实支付事实，一次开通 |
| GET /ops/reviews | type, cursor, limit | 审核权限；不对普通账号开放 |
| POST /ops/reviews/{id}/decisions | decision, reason | 审核权限，保存审计 |
| POST /ops/resources/{type}/{id}/suspend | reason | 受控下架，不删除历史订单 |
| GET /ops/dashboard | from, to | 只读权限，明确统计口径 |
| POST /ops/orders/{id}/dispute-decisions | expectedOrderVersion, decision, reason | 争议处理权限；保存裁定记录，不代替支付操作 |

媒体和支付回调不靠普通登录会话授权，要按供应商验证方式核对。运营身份由受控配置或授权记录管理，客户端不能注册成为运营。

## HTTP 与业务错误

400 输入不合法；401 未登录或会话失效；403 无权限；404 资源不存在或不可见；409 版本、档期、名额或幂等冲突；422 当前状态不允许操作；429 频率限制；503 外部服务暂不可用。

服务端错误不回传堆栈和凭证。上传授权或支付服务失败要能重新发起同一业务记录，不产生一排重复订单。当前只有接口设计，还没有实现这些 HTTP 接口。
