# 音乐酒吧街

[English](README.en.md) · [低保真原型](https://jiekertime.github.io/music-bar-street/prototype/) · [页面职责](https://jiekertime.github.io/music-bar-street/ui/) · [Figma 文件](https://www.figma.com/community/file/1692143979739836987)

音乐酒吧街是我想做的一款演出撮合 App：酒吧发演出需求，艺人看机会、投递；酒吧也可以看作品，直接邀请合适的艺人。双方谈好时间、费用和演出要求，再确认合作。

现在先把商业思路、产品流程、原型和后端设计放出来。有人愿意拿去做，我很高兴；想一起做，也可以联系我。

目前有完整的 [设计文档](docs/README.md)、[10 张产品与后端设计图](design/index.html)、16 张低保真原型和页面职责说明，App 的业务后端还没写。现有 UI 仅保存在本地，公开资料用重新绘制的灰色线框图说明页面结构。

## 先看什么

1. [看低保真原型](prototype/index.html)：从入驻走到找人、商量条件、确认和完成。
2. [看看商业思路](docs/BUSINESS_MODEL.md)：为什么做、准备怎么收费，还有哪些事没验证。
3. [看产品流程](docs/PRODUCT_SPEC.md)：一期做什么，投递、邀约和订单怎么衔接。
4. [看设计文档](docs/README.md) 和 [设计图](design/index.html)，了解系统、数据、状态、权限与接口。
5. [读页面职责](ui/index.html) 或复制 [Figma 文件](https://www.figma.com/community/file/1692143979739836987)，再按自己的需求改。
6. 准备开发时，再看 [后端设计](docs/IMPLEMENTATION_GUIDE.md)。

[下载全部资料](https://github.com/JiekerTime/music-bar-street/archive/refs/heads/main.zip) · [联系我](https://github.com/JiekerTime/music-bar-street/issues/new/choose)

## 一场合作怎么发生

```text
酒吧发需求 → 艺人投递 → 酒吧选人 ─┐
                                 ├─→ 商量合作条件
酒吧直接邀请艺人 → 艺人愿意聊 ────┘
                         ↓
          谈好日期、时段、场地和费用
                         ↓
              双方确认同一份条件
                         ↓
                  生成演出订单
                         ↓
             演出结束，双方确认完成
```

预算和最后谈定的费用分开记录。条件改了，之前的确认作废，双方重新确认。第一版让酒吧和艺人自行结算演出费，平台先保存合作记录。

会员和 1% 服务费是最初的收费想法，还没有验证。直播、打赏和钱包是一开始的扩展想法，一期先不做。

## 拿去做，或一起做

你可以自行修改、开发和商用，不需要先联系我，也不需要给我分成。文档、原型与设计图 用 **CC BY 4.0**，网页源码用 **Apache-2.0**；使用时保留相应署名和许可，细节见 [许可说明](LICENSING.md)。

如果你手里有酒吧或艺人资源，愿意尝试落地；或者想补设计、写代码，可以在 [Issues](https://github.com/JiekerTime/music-bar-street/issues) 留言。说下你在哪个城市、能做哪部分、想先试什么就行。实际合伙的投入和分工再一起谈。

其他资料：[怎么试点](docs/ADOPTION.md) · [参与方式](CONTRIBUTING.md) · [页面目录](docs/UI_MAP.md) · [发布记录](docs/RELEASE_PLAN.md)。

作者：Ruoyi / [JiekerTime](https://github.com/JiekerTime)。[Product Hunt](https://www.producthunt.com/products/music-bar-street?launch=music-bar-street) 已提交，安排在 2026 年 10 月 15 日 15:01（北京时间）发布。
