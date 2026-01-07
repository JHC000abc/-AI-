# 如有合作意愿可直接评论联系我,或者联系邮箱:JHC000abc@gmail.com
<img width="1409" height="1477" alt="d85a26fbcb6d9e2b1780070a46d55586" src="https://github.com/user-attachments/assets/c23ddd3e-5c9c-481a-99c5-9e75a00960a4" />


`抖音AI神评助手 (v1.4.5 概率评论版)` 

该脚本是一个高度集成的 Tampermonkey 用户脚本，旨在通过拦截网络请求（Hook）和模拟用户操作，实现抖音网页版的自动化交互、数据抓取、媒体下载以及基于本地大语言模型（LLM）的智能评论功能。

### 🧩 插件功能深度分析列表

#### 1. 🤖 AI 智能评论系统 (核心功能)

* **本地 LLM 集成**：脚本通过 `GM_xmlhttpRequest` 与本地运行的 AI 服务进行通信。
* **⚠️ 依赖验证**：默认接口地址为 `http://127.0.0.1:11434/api/generate`，默认模型为 `deepseek-r1:1.5b`。**这意味着用户必须自行部署 Ollama 或类似兼容 API 的本地 AI 服务，否则该功能无法使用。**


* **上下文感知生成**：AI 的提示词（Prompt）不仅包含基础设定，还动态抓取了以下信息作为上下文：
* 视频纯净标题、提取的观点/话题。
* 作者昵称、系统标签、推荐词。
* **网友热评**（提取前15条有效评论作为参考，以融入社区氛围）。


* **🆕 概率控制机制 (v1.4.5 新增)**：
* 引入了 `commentRate`（评论概率）设置。
* 在每次尝试评论前生成 1-100 的随机数，只有当随机数小于等于设定值时才会请求 AI 并发送评论，否则跳过。


* **防重复与过滤**：
* 自动跳过已记录历史（Session History）的视频。
* 内置关键词过滤（如“逝世”、“R.I.P”、“广告”），命中关键词自动跳过并划走。



#### 2. 📡 网络数据嗅探与 API 劫持 (Hook 技术)

* **环境快照 (Env Snapshot)**：
* 自动捕获并更新关键的鉴权参数
* **⚠️ 风险提示**：脚本试图模拟真实用户的请求头。


* **XHR/Fetch 劫持**：
* 重写了多个原生接口。
* 从响应 JSON 中提取无水印视频地址、高码率流、作者详细信息等，并存储在全局 map 中。



#### 3. 📥 媒体与数据增强工具

* **画质增强下载**：
* 自动解析视频流中的 `bit_rate` 列表，排序并选择最高码率（通常为 1080p+）的视频地址。
* 提供“📥 下载视频”按钮，文件名自动格式化为 `时间_作者_标题.mp4`。


* **数据深挖与复制**：
* 提供“📋 复制分析”功能，一键将视频的深度信息复制到剪贴板。
* **主页深度分析**：若开启 `autoProfile`，脚本会额外请求作者主页接口，获取并缓存作者的**地区（精确到区县/IP）、年龄、粉丝数、获赞数、认证信息**等隐私数据。


* **封面提取**：提供直接打开当前视频高清封面的功能。

#### 4. ❤️ 自动化交互 (点赞/收藏)

* **概率交互**：
* 包含自动点赞和自动收藏功能。
* 受 `likeRate`（点赞/收藏概率）控制，模拟人类的不确定性操作，非 100% 触发。


* **手动/快捷键触发**：
* `Alt + L`：手动触发当前视频点赞。
* `Alt + C`：手动触发当前视频收藏。



#### 5. 🕹️ 自动化流程与导航控制

* **脉冲检测与自动下滑**：
* 使用定时器检测当前视频状态。
* 当 AI 评论完成、或判定跳过（广告/已看/概率未命中）后，经过随机延迟（`minTime` 到 `maxTime`）自动触发“方向键下”模拟下滑。
* **死锁恢复**：若页面卡在评论区无法获取数据，脚本包含强制划走的逻辑以防止死循环。


* **状态机可视化**：
* 悬浮球（`⚡`）颜色根据当前状态实时变化：
* 🟣 等待/空闲
* 🟡 分析中/请求 AI
* 🟢 成功/操作完成
* 🔴 错误/超时
* 🔵 跳过/广告




* **外部 App 拦截**：
* 通过重写方法**强制拦截并阻止抖音网页版尝试唤起桌面端应用或手机 App 的行为**。



#### 6. ⚙️ UI 界面与配置系统

* **悬浮控制球**：支持拖拽，显示当前倒计时或状态图标。
* **全功能设置面板**：
* **实时统计**：展示已分析、评论、点赞、收藏、广告跳过的数据统计。
* **参数热调整**：支持动态调整 API 地址、模型名称、提示词、各种概率阈值（评论/点赞概率）及时间延迟。
* **暗黑模式**：适配网页的暗色主题 UI。
* **配置持久化**：保存用户偏好。



---

### ⚠️ 重要风险与限制说明 (基于代码分析)

1. **AI 服务依赖**：代码中硬编码了 API 请求格式为 `{"model": ..., "prompt": ..., "stream": false}`。这主要适配 Ollama 接口。**如果用户没有配置本地 AI 服务，AI 评论功能将直接报错或超时。**
2. **账号安全**：虽然脚本尝试通过增加随机延迟来模拟人工，但高频的自动化评论、点赞和数据抓取（尤其是主页深挖功能）极易触发抖音的风控机制，导致**账号禁言或封号**。
3. **Cookie 有效性**：脚本依赖页面现有的 Cookie 

---

### 总结

这是一个功能极其强大的**灰产级/极客级**脚本，它不仅仅是一个简单的评论工具，更是一个集**数据采集、自动化运营、AI 内容生成**于一体的综合系统。v1.4.5 版本通过加入“评论概率”控制，进一步增强了模拟真人的隐蔽性。


基于您提供的 `抖音AI神评助手 (v1.4.5 概率评论版)` 脚本代码，以下是对其功能的详细深度分析与总结列表。

该脚本是一个高度集成的 Tampermonkey 用户脚本，旨在通过拦截网络请求（Hook）和模拟用户操作，实现抖音网页版的自动化交互、数据抓取、媒体下载以及基于本地大语言模型（LLM）的智能评论功能。

### 🧩 插件功能深度分析列表

#### 1. 🤖 AI 智能评论系统 (核心功能)

* **本地 LLM 集成**：脚本通过 `GM_xmlhttpRequest` 与本地运行的 AI 服务进行通信。
* **⚠️ 依赖验证**：默认接口地址为 `http://127.0.0.1:11434/api/generate`，默认模型为 `deepseek-r1:1.5b`。**这意味着用户必须自行部署 Ollama 或类似兼容 API 的本地 AI 服务，否则该功能无法使用。**


* **上下文感知生成**：AI 的提示词（Prompt）不仅包含基础设定，还动态抓取了以下信息作为上下文：
* 视频纯净标题、提取的观点/话题。
* 作者昵称、系统标签、推荐词。
* **网友热评**（提取前15条有效评论作为参考，以融入社区氛围）。


* **🆕 概率控制机制 (v1.4.5 新增)**：
* 引入了 `commentRate`（评论概率）设置。
* 在每次尝试评论前生成 1-100 的随机数，只有当随机数小于等于设定值时才会请求 AI 并发送评论，否则跳过。


* **防重复与过滤**：
* 自动跳过已记录历史（Session History）的视频。
* 内置关键词过滤（如“逝世”、“R.I.P”、“广告”），命中关键词自动跳过并划走。



#### 2. 📡 网络数据嗅探与 API 劫持 (Hook 技术)

* **环境快照 (Env Snapshot)**：
* 自动捕获并更新关键的鉴权参数：`msToken`、`X-Bogus`、`_signature`、`sid_guard` 和 Cookie。
* **⚠️ 风险提示**：脚本试图通过 `handleCommentList` 和 `captureEnvSnapshot` 模拟真实用户的请求头。**如果抖音更新了签名算法或加密逻辑，这些硬编码的参数抓取逻辑可能会失效，甚至导致账号风控。**


* **XHR/Fetch 劫持**：
* 重写了 `XMLHttpRequest` 和 `fetch`，监听 `comment/list`（评论列表）、`feed`（视频流）等接口。
* 从响应 JSON 中提取无水印视频地址、高码率流、作者详细信息等，并存储在全局 `dy_video_map` 中。



#### 3. 📥 媒体与数据增强工具

* **画质增强下载**：
* 自动解析视频流中的 `bit_rate` 列表，排序并选择最高码率（通常为 1080p+）的视频地址。
* 提供“📥 下载视频”按钮，文件名自动格式化为 `时间_作者_标题.mp4`。


* **数据深挖与复制**：
* 提供“📋 复制分析”功能，一键将视频的深度信息复制到剪贴板。
* **主页深度分析**：若开启 `autoProfile`，脚本会额外请求作者主页接口，获取并缓存作者的**地区（精确到区县/IP）、年龄、粉丝数、获赞数、认证信息**等隐私数据。


* **封面提取**：提供直接打开当前视频高清封面的功能。

#### 4. ❤️ 自动化交互 (点赞/收藏)

* **概率交互**：
* 包含自动点赞和自动收藏功能。
* 受 `likeRate`（点赞/收藏概率）控制，模拟人类的不确定性操作，非 100% 触发。


* **手动/快捷键触发**：
* `Alt + L`：手动触发当前视频点赞。
* `Alt + C`：手动触发当前视频收藏。



#### 5. 🕹️ 自动化流程与导航控制

* **脉冲检测与自动下滑**：
* 使用定时器检测当前视频状态。
* 当 AI 评论完成、或判定跳过（广告/已看/概率未命中）后，经过随机延迟（`minTime` 到 `maxTime`）自动触发“方向键下”模拟下滑。
* **死锁恢复**：若页面卡在评论区无法获取数据，脚本包含强制划走的逻辑以防止死循环。


* **状态机可视化**：
* 悬浮球（`⚡`）颜色根据当前状态实时变化：
* 🟣 等待/空闲
* 🟡 分析中/请求 AI
* 🟢 成功/操作完成
* 🔴 错误/超时
* 🔵 跳过/广告




* **外部 App 拦截**：
* 通过重写 `window.open` 和 `document.createElement('iframe')`，**强制拦截并阻止抖音网页版尝试唤起桌面端应用或手机 App 的行为**。



#### 6. ⚙️ UI 界面与配置系统

* **悬浮控制球**：支持拖拽，显示当前倒计时或状态图标。
* **全功能设置面板**：
* **实时统计**：展示已分析、评论、点赞、收藏、广告跳过的数据统计。
* **参数热调整**：支持动态调整 API 地址、模型名称、提示词、各种概率阈值（评论/点赞概率）及时间延迟。
* **暗黑模式**：适配网页的暗色主题 UI。
* **配置持久化**：使用 `GM_setValue` / `GM_getValue` 保存用户偏好。



---

### ⚠️ 重要风险与限制说明 (基于代码分析)

1. **AI 服务依赖**：代码中硬编码了 API 请求格式为 `{"model": ..., "prompt": ..., "stream": false}`。这主要适配 Ollama 接口。**如果用户没有配置本地 AI 服务，AI 评论功能将直接报错或超时。**
2. **账号安全**：虽然脚本尝试通过 `comment_send_celltime` 增加随机延迟来模拟人工，但高频的自动化评论、点赞和数据抓取（尤其是主页深挖功能）极易触发抖音的风控机制，导致**账号禁言或封号**。
3. **DOM 结构依赖**：脚本依赖特定的 DOM 属性（如 `data-e2e="comment-list"`）。**一旦抖音更新网页结构（类名混淆或重构），自动打开评论区和检测逻辑将彻底失效。**
4. **Cookie 有效性**：脚本依赖页面现有的 Cookie 和 `msToken`。如果 Session 过期或签名参数获取失败，所有写操作（评论、点赞）都会返回 API 错误（如状态码 8 或 1024）。

---

### 总结

这是一个功能极其强大的**极客级**脚本，它不仅仅是一个简单的评论工具，更是一个集**数据采集、自动化运营、AI 内容生成**于一体的综合系统。v1.4.5 版本通过加入“评论概率”控制，进一步增强了模拟真人的隐蔽性。


<figure style="text-align: center; margin: 20px 0;">
  <img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/9cea10c1-4726-4ec5-849d-7b1f5ecf041a" style="max-width: 100%; height: auto;" />
  
  <figcaption style="margin-top: 10px; color: #555; font-weight: bold; font-size: 16px;">
    正常模式(悬浮球收起)
  </figcaption>
</figure>


<figure style="text-align: center; margin: 20px 0;">
  <img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/989b3b42-7fe1-48ec-9798-67cdb9154b11" style="max-width: 100%; height: auto;" />
  
  <figcaption style="margin-top: 10px; color: #555; font-weight: bold; font-size: 16px;">
    悬浮球基础配置
  </figcaption>
</figure>


<figure style="text-align: center; margin: 20px 0;">
  <img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/2ec02c0a-70c0-4a7a-b671-15c75c03e757" style="max-width: 100%; height: auto;" />
  
  <figcaption style="margin-top: 10px; color: #555; font-weight: bold; font-size: 16px;">
    悬浮球高级配置
  </figcaption>
</figure>



<img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/a21ed82b-12f2-4162-9b27-fd789669c5dc" />
<img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/8eda9095-4c67-4425-995b-af97312760c5" />
<img width="3812" height="1928" alt="image" src="https://github.com/user-attachments/assets/26afb3c1-4326-4edb-ae33-fad9c8bb998e" />


## 增加了获取全部评论的能力

<img width="3602" height="2099" alt="image" src="https://github.com/user-attachments/assets/a876d9bc-6b58-4455-b010-ecfe78985e87" />
<img width="3841" height="1866" alt="image" src="https://github.com/user-attachments/assets/58daa094-f71f-4f80-b3dc-88f6bf962817" />
<img width="3841" height="1866" alt="image" src="https://github.com/user-attachments/assets/01ab6bb6-9c82-45d7-9ef2-a895a7d55e1c" />
<img width="3841" height="1866" alt="image" src="https://github.com/user-attachments/assets/32ae4065-fce7-43e4-92f6-7f592ca77d19" />
<img width="3841" height="1866" alt="image" src="https://github.com/user-attachments/assets/156ef88e-819f-4af6-b9ab-c1efe4bf912a" />
<img width="3340" height="1006" alt="image" src="https://github.com/user-attachments/assets/6accfdb1-d5bd-4ba1-8bfd-cc5e0c93a446" />





