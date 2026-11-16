# Paper Interpreter

将原 `paper-triage`、`research-interpreter` 和 `review-interpreter` 合并为一个论文阅读技能：从用户提供的论文 PDF 出发，依次完成材料处理、阅读初筛与分类，再按论文类型生成 Obsidian Markdown 解读。

## 安装

将仓库克隆到目标 agent 的技能目录，例如：

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/oroiteS/paper-interpreter.git ~/.agents/skills/paper-interpreter
```

目标目录按所用 agent 调整；若目录或软链已存在，先检查其指向和本地配置，避免覆盖。已有独立克隆可在仓库目录运行 `git pull` 更新。安装时保留 `SKILL.md` 与 `references/` 的相对结构。

## 使用方式

向支持该技能的 agent 提供论文 PDF 路径，或提供可获取 PDF 的明确 URL，并说明希望关注的问题。例如：

```text
请阅读并解读 /path/to/paper.pdf，重点关注方法和实验结果。
```

仅提供标题，或提供的路径无法定位时，技能会请求文件或可靠链接。URL 对应的论文身份和版本需要先核实。

## 工作流程

四个阶段由不同 agent **串行**执行，上一阶段完成并交接后才启动下一阶段：

1. **PDF 处理**：使用运行环境中适合处理已有 PDF 的技能，提取可追溯的正文、页码及必要图表，记录处理限制。
2. **初筛与分类**：检查论文主线、关键证据和公开材料，给出精读、选读、暂缓或信息不足的建议，并依据实际材料判断 `Review`、`Research` 或无法判断。默认从 Personalized LLM（pLLM）研究视角出发；用户本次指定其他目标时优先遵循。不开源或缺少算力不单独构成不推荐阅读的理由。
3. **深度解读**：按类型选择综述或原始研究路线，在解读正文开头原样保留初筛的阅读建议，由第三阶段 agent 写作并交付初稿。
4. **独立审阅**：调度者派遣不同的只读 agent 对照原论文核查解读内容，不评价投稿资格。审阅完成后，调度者将意见交回原第三阶段 agent，由其核对并修订，再执行公式后处理。

阅读建议并非自动中止条件：即使建议暂缓或信息不足，只要类型明确且材料足以支持解读，仍继续处理。类型无法判断时不强行选择路线；无法派遣 agent、关键材料不可用或审阅、后处理失败时会报告阻碍，而不将未完成步骤称为成功。

## 输出

相对于发起任务时的工作目录生成：

```text
./paper/markdown/{论文名}/
├── 解读.md
└── append/             # PDF 处理输出、图片、提取文本及其他需保留的副产物
```

所有阶段使用同一相对路径基准；用户提供的原始 PDF 保留在原位置，通过 URL 下载的 PDF 放入 `append/`。论文目录无法确定时先询问，不猜测其他保存位置。具体的路径与产物规则由 `SKILL.md` 统一规定，解读参考文档只规定阅读和写作要求。

解读包含对应论文类型的 YAML 元数据和正文结构。Review 路线侧重领域范围、分类体系、技术路线与开放问题；Research 路线侧重研究问题、方法、实验与批判性评估。关键结论应能追溯至原文，未知或未核实的内容会明确标注。

## 运行条件

- 运行环境须支持调度者分别派遣四个阶段 agent，并在审阅后继续联系原第三阶段 agent；阶段及审阅后的修订之间不能并行。
- 阶段一需要可用的 PDF 处理技能；解读前需要 `kimi-obsidian-markdown`。需要自绘图时还须使用 `nature-figure`；默认优先使用文字或合适的原论文图表。
- 公式后处理依赖学习库根目录的 `scripts/math.py`；该脚本不在本仓库中。脚本缺失或运行失败时会停止相应步骤并报告。
- 本技能包含默认 pLLM 阅读视角和上述学习库路径约定；按实际需求修改使用目标或配置。

## 仓库结构

```text
paper-interpreter/
├── README.md
├── SKILL.md
├── references/
│   ├── paper-triage.md
│   ├── research-interpreter.md
│   ├── review-interpreter.md
│   └── interpretation-review.md
└── LICENSE
```

各阶段的具体规则以 `SKILL.md` 和对应参考文档为准。

## 许可证

[MIT](LICENSE)
