# 低保真原型说明

[原型图目录](../prototype/index.html) 提供 16 张重新绘制的灰色线框图，全部使用 SVG 图形和文字，没有嵌入原有 UI 图片。图上描述页面结构、操作、权限和异常状态；布局是实现前的建议，可以调整。

| 原型 | 主要职责 | 下一步 |
| --- | --- | --- |
| [登录与身份选择](../prototype/01-sign-in.svg) | 手机号验证码登录后再选择身份。 | 入驻资料 |
| [酒吧入驻](../prototype/02-venue-onboarding.svg) | 资质上传使用短时授权。 | 酒吧主页 / 发布需求 |
| [艺人入驻](../prototype/03-artist-onboarding.svg) | 报价同时显示单位和币种。 | 艺人主页 / 演出机会 |
| [艺人主页](../prototype/04-artist-profile.svg) | 只显示审核通过的公开资料。 | 发起邀约 |
| [演出机会列表](../prototype/05-gig-list.svg) | 有效需求分页展示。 | 需求详情 |
| [发布演出需求](../prototype/06-publish-gig.svg) | 结束时间晚于开始时间；支持跨午夜。 | 需求管理 / 候选人 |
| [需求详情与投递](../prototype/07-gig-detail.svg) | 艺人不能投递自己的酒吧需求。 | 候选人 / 洽谈 |
| [候选人管理](../prototype/08-candidates.svg) | 选择一人开始洽谈，不自动确认订单。 | 消息 / 合作条件 |
| [发出与回应邀约](../prototype/09-invitation.svg) | 只有指定艺人可以回应。 | 消息 / 合作条件 |
| [私信与通知](../prototype/10-messages.svg) | 消息保存在服务端，重连能补历史。 | 合作条件 |
| [商量最终合作条件](../prototype/11-terms.svg) | 预算与最终费用分别记录。 | 双方确认 |
| [双方确认合作](../prototype/12-confirm.svg) | 只允许确认当前版本。 | 演出订单 |
| [订单详情与变更](../prototype/13-order.svg) | 普通编辑不覆盖订单历史快照。 | 完成确认 / 变更回应 |
| [演出完成与争议](../prototype/14-completion.svg) | 演出结束后双方分别确认。 | 合作记录 |
| [商家会员](../prototype/15-membership.svg) | 一期先说明权益、记录购买意向。 | 会员权益 / 需求发布 |
| [只读运营看板](../prototype/16-operations.svg) | 一期不拆独立管理服务。 | 运营审核与异常处理 |

页面交互依照[产品流程](PRODUCT_SPEC.md)、[状态与权限](STATE_AND_ACCESS.md)和[接口约定](API_DESIGN.md)。接受邀约不会生成订单；双方确认同一版本才成单；完成、取消、变更和争议保留独立记录。会员收费前需定价格和支付规则，运营看板一期只读。

原有 UI 仅保存在本地，当前公开文件与下载包不包含原图或嵌入原图的参考板。
