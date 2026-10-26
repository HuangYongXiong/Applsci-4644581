# SA 盲评标注 Rubric（SYNTHETIC demo）

## 标注者资质（demo 设定）
- 标注员 A：6 年应急 GIS / 灾情图件制作经验，省级应急指挥中心助工；
- 标注员 B：4 年应急救援规划经验，高校资源与环境方向；
- 仲裁 E3：NLP/LLM 方向副教授。

## 四级判定
| 等级 | 定义 | 锚点 |
|---|---|---|
| Fully correct | 全部关键事实承诺正确：设施/村镇点名、空间关系（距离/方位/乡镇）正确，无编造、无关键遗漏 | 正确指出磨西镇附近卫生院、距离与所属乡镇 |
| Partially correct | 主体正确但一处非关键偏差（距离略偏、漏次要设施），无错位无编造 | 点名正确但漏一个备选安置点 |
| Incorrect | wrong_location / hallucinated_entity / vague_no_direct_answer / incomplete_key_facts | 设施安错乡镇或虚构救援队 |
| Unanswerable | 参考材料无答案 | 官方通报未覆盖 |

## 流程
A、B 独立盲判（系统匿名、行序打乱）；不一致交 E3 仲裁；相同叙述只判一次（去重 1600 行）；kappa=0.5213，一致 1354/1600，246 条仲裁。
