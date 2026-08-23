

[求助，想自己写skills，应该从哪里学起？ - 开发调优 / 开发调优, Lv1 - LINUX DO](https://linux.do/t/topic/2777118)

 ### product-manager

  主要能力：
  - 编写产品需求文档（PRD）
  - 定义功能与非功能需求
  - 使用 MoSCoW、RICE、Kano 等方法进行需求优先级排序
  - 将需求拆分为 Epic 和 User Story
  - 编写验收标准
  - 审查和完善现有需求文档
  - 创建轻量级技术规格说明 (skills.sh (https://www.skills.sh/aj-geddes/claude-code-bmad-skills/product-manager?utm_source=openai))

  安装到 Codex：
  npx skills add https://github.com/aj-geddes/claude-code-bmad-skills --skill product-manager -g -a codex

  项目级安装则去掉 -g：
  npx skills add https://github.com/aj-geddes/claude-code-bmad-skills --skill product-manager -a codex



Skill 的 md 文件确实会被塞进模型的 context，模型基于概率去理解、决定要不要遵循、怎么应用。

所以确实存在"被长对话淹没""被模型忽略"的风险

Claude Code 用的缓解办法是渐进式加载(progressive disclosure)

不是一次性把所有 skill 塞进 context，而是先给模型一个简短的 skill 列表/描述，模型自己判断相关性后再主动去读取完整内容。

但这终究是**软约束** ——模型仍然可能不去读，或者读了但没完全照做。