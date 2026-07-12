
## 一、文档目标

本文档用于指导 Windows 用户安装和配置以下工具：

- CC Switch
- Claude Code
- Codex CLI
- Cherry Studio
- OpenCode
- VPN/代理工具
- WisArt 生图网站

主要用途：

| 工具 | 用途 |
|---|---|
| CC Switch | 集中管理第三方 API 线路 |
| Claude Code | 终端编程、项目分析和文件操作 |
| Codex CLI | 终端编程、项目分析和文件操作 |
| Cherry Studio | 日常对话、办公和模型测试 |
| OpenCode | 主线路不可用时的保底工具 |
| WisArt | AI 图片生成 |

---

# 二、重要使用原则

## 2.1 每次使用前必须检查模型健康度

每次使用 Claude Code、Codex 或 Cherry Studio 前，必须先打开对应线路的“**模型健康检测”页面。爆红，就用不了。

标准流程：

```text
打开模型健康检测页面
→ 找到当前配置的模型
→ 确认模型健康
→ 打开 CC Switch
→ 切换到对应线路
→ 启动 Claude Code、Codex 或 Cherry Studio
```

如果模型显示异常、离线、失败或高延迟：

1. 切换到其他健康线路；
2. 如果所有主线路（鸡毛，不跑路，hyb）均不可用，改用 OpenCode；

## 2.2 对话时直接提问

不要输入：你好

如果只想测试当前模型是否可用，统一输入：检查当前目录

这个测试可以同时检查：

- 模型是否响应；
- API 是否正常；
- CLI 工具调用是否正常；
- 当前目录读取是否正常。


# 三、第三方 API 线路

## 3.1 鸡毛

```text
名称：鸡毛
API URL：https://api.ark717.com
API Key：sk-XC3fRgJdXKjQKN9pqYE7nIvY51jmR0jEA8JgGQVKyUyfAJ5I
模型健康检测：https://pool.ark717.com/
```

使用前操作：

1. 打开模型健康检测页面；
2. 找到准备使用的模型；
3. 确认模型状态健康；
4. 在 CC Switch 中启用“鸡毛”；
5. 再启动 Claude Code 或 Codex。

---

## 3.2 不跑路

```text
名称：不跑路
API URL：https://runanytime.hxi.me/
API Key：sk-B31AekxrnEvszDX5TRZ2AwNPXq7fQjbcKsR6hTugQzEgkIdU
模型健康检测：https://stat.hxi.me/status/ai
```

使用前操作：

1. 打开模型健康检测页面；
2. 检查当前模型状态；
3. 模型健康后，在 CC Switch 中启用“不跑路”；
4. 再启动 Claude Code 或 Codex。

---

## 3.3 HYB

```text
名称：HYB
API URL：https://ai.hybgzs.com/
API Key：sk-doJc46tZw0qPeu5_gIxboYj0J4L2uHwVNJ_IR3cp3snX3xkx3NMurrl6V_c 
模型健康检测：https://ai.hybgzs.com/panel/model_health
```

使用前操作：

1. 打开模型健康检测页面；
2. 确认模型状态健康；
3. 在 CC Switch 中启用“HYB”；
4. 再启动 Claude Code 或 Codex。

### HYB 已知问题

HYB 可能出现以下情况：

```text
Cherry Studio 已连接，但无法正常对话；
终端中的 Claude Code 或 Codex 可以正常使用。
```

遇到这种情况不代表整条线路失效，应分别测试：

1. 在 Cherry Studio 中刷新对话；
2. 在 Cherry Studio 中重新选择模型；
3. 新建一个对话窗口；
4. 输入实际问题进行测试；
5. **如果仍然失败**，打开终端运行 Claude Code 或 Codex；
6. 输入：

```text
检查当前目录
```

如果终端可以正常返回，则说明：

- API 线路可能仍然正常；
- 问题可能只发生在 Cherry Studio；
- 可以暂时使用终端继续工作；
- 也可以切换其他服务商供 Cherry Studio 使用。

其他线路也可能出现“GUI 不可用但终端可用”或“终端不可用但 GUI 可用”的情况，因此需要分别测试，不能只根据一个客户端判断线路整体状态。

# 四、推荐使用顺序

建议按照以下顺序选择线路：

```text
检查线路健康度
→ 使用当前健康的主要线路
→ 出现问题后切换其他健康线路
→ 所有线路均不可用时使用 OpenCode
```

线路选择表：

| 场景 | 建议操作 |
|---|---|
| 当前服务商和模型健康 | 继续使用当前线路 |
| 当前模型异常 | 在当前服务商中切换健康模型 |
| 当前服务商整体异常 | 在 CC Switch 中切换服务商 |
| Cherry Studio 无法对话 | 刷新、新建对话或改用终端 |
| 终端无法使用 | 测试 Cherry Studio或切换服务商 |
| 所有第三方线路异常 | 使用 OpenCode |
| 需要生成图片 | 使用 WisArt |

---

# 五、安装基础环境

## 5.1 安装 Node.js

打开 Node.js 官方网站：

```text
https://nodejs.org/
```

下载并安装 LTS 版本。

安装完成后，关闭并重新打开 PowerShell，执行：

```powershell
node --version
npm --version
```

只要能正常显示版本号，即表示安装成功。

### 截图建议

在文档中加入：

1. Node.js 下载页面；
2. Node.js 安装完成页面；
3. PowerShell 中的版本验证结果。

---

# 六、安装 Claude Code

打开 PowerShell，执行：

```powershell
npm install -g @anthropic-ai/claude-code
```

安装完成后验证：

```powershell
claude --version
```

能显示版本号即表示安装成功。

> Claude Code 不需要在终端里单独手动填写 URL 和 API Key。后续统一在 CC Switch 中配置。

### 截图建议

1. PowerShell 安装命令；
2. 安装完成结果；
3. `claude --version` 的输出结果。

---

# 七、安装 Codex CLI

打开 PowerShell，执行：

```powershell
npm install -g @openai/codex
```

验证安装：

```powershell
codex --version
```

能显示版本号即表示安装成功。

> Codex 不需要单独手动配置 URL 和 API Key。后续统一在 CC Switch 中配置。

### 截图建议

1. Codex 安装命令；
2. 安装完成结果；
3. `codex --version` 的输出结果。

---

# 八、安装和配置 CC Switch

## 8.1 安装 CC Switch

从 CC Switch 的官方发布页面下载 Windows 安装包：

```text
https://github.com/farion1231/cc-switch/releases
```

安装步骤：

1. 打开发布页面；
2. 下载 Windows 安装包；
3. 双击安装；
4. 安装完成后打开 CC Switch。

> 不要从第三方软件下载站、陌生网盘或不明群文件下载安装包。

---

## 8.2 添加 Claude Code 服务商

打开 CC Switch 后：

1. 进入 **Claude Code**；
2. 点击“添加服务商”；
3. 输入服务商名称；
4. 填入服务商的 URL；
5. 填入管理员提供的 API Key；
6. 保存配置；
7. 点击“启用”。

填写示例：

```text
服务商名称：鸡毛
URL：https://api.ark717.com
API Key：sk-********************
```

另外两个服务商使用相同方法添加：

```text
不跑路：https://runanytime.hxi.me/
HYB：https://ai.hybgzs.com/
```

> URL 是否保留末尾的 `/`，以服务商实际说明和已验证配置为准。不要自行增加 `/v1`。

---

## 8.3 添加 Codex 服务商

打开 CC Switch 后：

1. 进入 **Codex**；
2. 点击“添加服务商”；
3. 输入服务商名称；
4. 填入相应 URL；
5. 填入管理员提供的 API Key；
6. 保存配置；
7. 点击“启用”。

Claude Code 和 Codex 都通过 CC Switch 管理，终端中不需要再次手动填写 URL 和 Key。

---

## 8.4 切换线路

每次使用前：

1. 打开对应模型健康检测页面；
2. 确认模型健康；
3. 打开 CC Switch；
4. 选择健康的服务商；
5. 点击“启用”；
6. 关闭之前打开的终端；
7. 重新打开 PowerShell；
8. 启动 Claude Code 或 Codex。

启动 Claude Code：

```powershell
claude
```

启动 Codex：

```powershell
codex
```

如果只是测试，输入：

```text
检查当前目录
```

---

# 九、安装和配置 Cherry Studio

## 9.1 安装

从 Cherry Studio 官方网站或官方 GitHub Releases 下载 Windows 版本。

安装完成后打开 Cherry Studio。

---

## 9.2 添加服务商

操作步骤：

1. 打开 Cherry Studio；
2. 进入“设置”；
3. 打开“模型服务”；
4. 添加服务商；
5. 填入服务商名称；
6. 填入 URL；
7. 填入管理员提供的 API Key；
8. 添加或选择对应模型；
9. 保存配置。

示例：

```text
服务商名称：鸡毛
API 地址：https://api.ark717.com
API Key：sk-********************
模型：选择健康检测页面中状态正常的模型
```

“不跑路”和“HYB”按照同样的方法添加。

## 9.3 测试模型

不要只输入“你好”，直接输入实际任务，例如：

```text
总结下面这段文字的主要内容。
```

或者：

```text
帮我写一份本周工作总结模板。
```

需要注意：

```text
检查当前目录
```

更适合在 Claude Code、Codex 或 OpenCode 等终端工具中测试。Cherry Studio通常无法直接读取电脑当前目录，除非已经启用对应的文件或工具能力。

### 截图建议

1. Cherry Studio 设置入口；
2. 模型服务入口；
3. 添加服务商页面；
4. URL 填写位置；
5. API Key 填写位置；
6. 模型添加位置；
7. 新建对话和选择模型的位置。

---

# 十、安装 OpenCode

OpenCode 作为主线路全部出现问题时的最后保障。

## 10.1 安装命令

确认 Node.js 已安装后，在 PowerShell 中执行：

```powershell
npm install -g opencode-ai
```

验证安装：

```powershell
opencode --version
```

启动 OpenCode：

```powershell
opencode
```

如果安装命令因版本更新失效，应查看 OpenCode 官方安装说明：

```text
https://opencode.ai/docs/
```

## 10.2 使用说明

当前内部可用配置提供免费的 DeepSeek V4 Flash，用于：

- 日常办公；
- 文本整理；
- 内容总结；
- 简单代码处理；
- 常规项目分析；
- 主线路故障时的临时工作。

> “免费模型”的名称、额度和可用性可能发生变化，应以 OpenCode 当时显示的可用模型及内部实际配置为准。

使用前进入需要处理的目录：

```powershell
cd "你的项目目录"
opencode
```

启动后直接输入任务，例如：

```text
检查当前目录
```

或者：

```text
分析当前目录的项目结构，并说明如何启动。
```

## 10.3 使用优先级

```text
主要 API 线路正常
→ 使用 Claude Code、Codex 或 Cherry Studio

主要 API 线路异常
→ 切换其他健康线路

所有主要 API 线路异常
→ 使用 OpenCode
```

---

# 十一、图片生成

需要生成图片时，使用：

```text
https://wisart.kuaileshifu.com/
```

操作步骤：

1. 使用浏览器打开网站；
2. 登录账号；
3. 进入图片生成页面；
4. 输入具体的图片描述；
5. 选择尺寸和相关参数；
6. 提交生成；
7. 下载并保存图片。

提示词不要只写：

```text
生成一张好看的图
```

建议写清楚：

```text
主体 + 场景 + 风格 + 构图 + 光线 + 色彩 + 尺寸 + 不需要的内容
```

示例：

```text
生成一张现代科技办公场景，桌面上放置笔记本电脑和咖啡，
蓝紫色科技感灯光，简洁构图，写实风格，16:9，
画面中不要出现文字和水印。
```

---

# 十二、VPN或代理使用

如果当前线路需要 VPN，按以下顺序操作：

1. 打开公司允许使用的 VPN 或代理工具；
2. 切换到可用节点；
3. 确认节点连接成功；
4. 打开模型健康检测页面；
5. 确认目标模型健康；
6. 打开 CC Switch；
7. 启用目标服务商；
8. 重新打开终端；
9. 启动 Claude Code 或 Codex。

如果对话失败，可以先尝试切换节点。

切换节点后建议：

1. 完全退出当前终端程序；
2. 关闭并重新打开 PowerShell；
3. 刷新 Cherry Studio对话；
4. 必要时新建对话；
5. 再次提交实际问题。

---

# 十三、常见问题处理

## 13.1 推荐的统一排查顺序

遇到问题时，按照以下顺序处理：

```text
第一步：检查模型健康度
第二步：确认 CC Switch已启用正确服务商
第三步：检查 URL和 API Key
第四步：切换 VPN/代理节点
第五步：刷新或新建对话
第六步：重启终端和客户端
第七步：切换其他服务商
第八步：使用 OpenCode保底
```

---

## 13.2 对话没有响应或响应失败

可以依次尝试：

### 方法一：切换节点

1. 打开 VPN 或代理工具；
2. 切换其他可用节点；
3. 重新打开健康检测页面；
4. 再次确认模型状态；
5. 重试对话。

### 方法二：刷新对话

Cherry Studio 中可以：

1. 点击重新生成；
2. 刷新当前对话；
3. 新建对话；
4. 重新选择模型；
5. 直接输入实际问题。

### 方法三：重启工具

1. 退出 Claude Code、Codex 或 OpenCode；
2. 关闭 PowerShell；
3. 重新在 CC Switch 中启用服务商；
4. 重新打开 PowerShell；
5. 再次启动工具。

### 方法四：切换服务商

如果当前服务商的模型健康状态异常：

1. 打开其他服务商的健康检测页面；
2. 找到健康模型；
3. 在 CC Switch 中切换到对应服务商；
4. 重启终端；
5. 再次测试。

---

## 13.3 出现 404

常见错误：

```text
404 Not Found
Model Not Found
The requested resource was not found
```

一般表示 API 请求地址、接口路径或模型存在问题，可能原因包括：

- 服务商 API 暂时异常；
- URL 填写错误；
- 接口路径不存在；
- 模型名称错误；
- 模型已下线；
- 服务商路由发生变化；
- 多写或少写了 `/v1`。

处理步骤：

1. 打开模型健康检测页面；
2. 检查当前模型是否健康；
3. 对照内部配置检查 URL；
4. 不要自行添加或删除 `/v1`；
5. 重新选择健康模型；
6. 切换其他服务商；
7. 向管理员反馈。

反馈时提供：

```text
服务商名称：
使用工具：
使用模型：
发生时间：
完整错误信息：
健康检测结果：
```

不要在反馈中发送完整 API Key。

---

## 13.4 出现 503

常见错误：

```text
503 Service Unavailable
Service temporarily unavailable
Upstream unavailable
```

503 通常表示服务暂时不可用、上游模型异常、线路拥堵，也可能和当前配置或节点有关。

处理步骤：

1. 查看模型健康度；
2. 确认当前模型是否在线；
3. 刷新或新建对话；
4. 切换 VPN/代理节点；
5. 在 CC Switch 中重新启用服务商；
6. 重启终端；
7. 切换其他健康模型；
8. 切换其他服务商；
9. 全部不可用时使用 OpenCode。

> 不能仅凭 503 就确定一定是本机配置错误。它也可能是第三方服务商或上游模型临时故障。

---

## 13.5 出现 401

常见错误：

```text
401 Unauthorized
Invalid API Key
```

可能原因：

- API Key 输入错误；
- API Key 前后存在空格；
- Key 已失效；
- Key 已被撤销；
- Key 不属于当前服务商；
- Key 没有相应权限。

处理步骤：

1. 重新复制管理员提供的 Key；
2. 检查是否存在前后空格；
3. 检查是否填入正确服务商；
4. 保存并重新启用；
5. 重启终端；
6. 仍然失败时联系管理员更换 Key。

---

## 13.6 出现 429

常见错误：

```text
429 Too Many Requests
Rate Limit
Insufficient Quota
```

可能原因：

- 当前线路请求过多；
- 服务商限流；
- 账户额度不足；
- 模型暂时拥堵；
- 同一个 Key 同时使用人数过多。

处理：

1. 等待一段时间后再试；
2. 查看模型健康度；
3. 切换其他模型；
4. 切换其他服务商；
5. 使用 OpenCode保底。

---

## 13.7 Cherry Studio连接成功，但不能对话

这种情况在 HYB 中出现过，其他服务商也可能发生。

处理步骤：

1. 确认选择了正确模型；
2. 检查模型健康度；
3. 点击刷新或重新生成；
4. 新建对话；
5. 重新选择模型；
6. 切换 VPN/代理节点；
7. 在终端中启动 Claude Code 或 Codex；
8. 输入：

```text
检查当前目录
```

判断方式：

| 测试结果 | 可能情况 |
|---|---|
| Cherry Studio失败，终端成功 | Cherry Studio兼容性或当前会话异常 |
| Cherry Studio成功，终端失败 | CC Switch配置、CLI配置或工具调用异常 |
| 两边都失败 | 服务商、模型、网络或 Key可能异常 |
| 切换服务商后恢复 | 原服务商临时异常 |

---

## 13.8 CC Switch切换后仍使用旧线路

处理步骤：

1. 退出 Claude Code 或 Codex；
2. 打开 CC Switch；
3. 确认目标服务商显示为“已启用”；
4. 再点击一次启用；
5. 关闭所有 PowerShell窗口；
6. 重新打开 PowerShell；
7. 再次启动 CLI。

不要在 Claude Code 或 Codex 正在运行时反复切换线路并继续使用旧会话。新人统一按“切换后重启终端”操作最稳妥。

---

## 13.9 命令无法识别

错误示例：

```text
claude 不是内部或外部命令
codex 不是内部或外部命令
opencode 不是内部或外部命令
```

先关闭并重新打开 PowerShell，然后验证：

```powershell
claude --version
codex --version
opencode --version
```

仍然失败时重新安装：

```powershell
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g opencode-ai
```

检查已安装的全局工具：

```powershell
npm list -g --depth=0
```

---

# 十四、完整操作流程

普通用户每次使用时，只需要按照下面的流程操作。

## Claude Code或 Codex

```text
1. 打开服务商模型健康检测页面
2. 确认准备使用的模型健康
3. 如果需要，连接 VPN并选择可用节点
4. 打开 CC Switch
5. 启用健康的服务商
6. 打开项目文件夹
7. 在项目文件夹中打开 PowerShell
8. 运行 claude 或 codex
9. 直接输入具体任务
```

Claude Code：

```powershell
claude
```

Codex：

```powershell
codex
```

测试命令：

```text
检查当前目录
```

## Cherry Studio

```text
1. 检查模型健康度
2. 打开 Cherry Studio
3. 选择对应服务商和健康模型
4. 新建对话
5. 直接输入实际问题
6. 如果失败，刷新、新建对话或切换节点
7. 仍然失败时改用终端或切换服务商
```

## 所有主线路均失败

```text
1. 打开项目目录
2. 在当前目录打开 PowerShell
3. 运行 opencode
4. 选择可用的免费模型
5. 直接输入任务
```

---







# 十五、验收检查表

## 安装检查

- [ ] Node.js 已安装
- [ ] npm 可以正常使用
- [ ] Claude Code 已安装
- [ ] Codex CLI 已安装
- [ ] CC Switch 已安装
- [ ] Cherry Studio 已安装
- [ ] OpenCode 已安装
- [ ] VPN或代理工具已安装

## 配置检查

- [ ] CC Switch 已添加“鸡毛”
- [ ] CC Switch 已添加“不跑路”
- [ ] CC Switch 已添加“HYB”
- [ ] Claude Code 配置可切换
- [ ] Codex 配置可切换
- [ ] Cherry Studio 已添加所需服务商
- [ ] 用户知道健康检测页面的位置
- [ ] 文档和截图中没有完整 API Key

## 功能检查

- [ ] 使用前已检查模型健康度
- [ ] Claude Code 可以执行“检查当前目录”
- [ ] Codex 可以执行“检查当前目录”
- [ ] Cherry Studio 可以回答实际问题
- [ ] 用户知道如何切换服务商
- [ ] 用户知道如何切换 VPN节点
- [ ] 用户知道 404 和 503 的基本处理方法
- [ ] 主线路失败时可以启动 OpenCode
- [ ] 可以打开 WisArt生成图片

---

# 十六、问题反馈模板

遇到无法解决的问题时，复制以下内容提交：

```text
【发生时间】
YYYY-MM-DD HH:mm

【使用工具】
Claude Code / Codex / Cherry Studio / OpenCode

【当前服务商】
鸡毛 / 不跑路 / HYB

【当前模型】
填写完整模型名称

【模型健康状态】
正常 / 异常 / 未检查

【VPN或代理】
未开启 / 已开启

【是否切换过节点】
是 / 否

【错误代码】
401 / 404 / 429 / 503 / 其他

【完整错误信息】
粘贴错误文本

【终端测试结果】
“检查当前目录”成功 / 失败 / 未测试

【已尝试操作】
1. 刷新或新建对话
2. 切换节点
3. 重启终端
4. 切换服务商
5. 使用OpenCode

【截图】
必须遮挡API Key、账号和内部敏感信息。
```
