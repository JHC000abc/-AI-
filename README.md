# 如有合作意愿可直接评论联系我,或者联系邮箱:JHC000abc@gmail.com
<img width="1409" height="1477" alt="d85a26fbcb6d9e2b1780070a46d55586" src="https://github.com/user-attachments/assets/c23ddd3e-5c9c-481a-99c5-9e75a00960a4" />


根据您上传的 `demo.js` 文件（版本 v4.0.0），这是对该脚本功能的深度分析。对比旧版本（v1.4.5），此版本进行了底层重构，从单一的评论工具进化为集**数据采集、全自动运营、弹幕交互、外部下载**于一体的综合控制台。

以下是基于 v4.0.0 代码的完整功能深度分析报告：

---

# 抖音AI神评论助手 (v4.0.0 测试版 重构界面/全功能版)

该脚本是一个企业级的 Tampermonkey 用户脚本，代码量显著增加。它不仅重构了 UI 界面，还新增了对粉丝列表、搜索结果、弹幕数据的采集功能，支持 Excel 导出，并集成了 Motrix 下载器与更复杂的 AI 交互逻辑（支持在线/本地双模）。

## 🧩 插件功能深度分析列表

### 1. 🤖 AI 智能评论与回复系统 (双模引擎)

* **双源 AI 支持 (Refactored)**：
* 界面新增 Tab 切换，支持 **本地模型 (Ollama)** 和 **在线模型 (OpenAI 格式)**。
* **本地模式**：默认连接 `http://127.0.0.1:11434/api/generate`，默认模型 `deepseek-r1:1.5b`。
* **在线模式**：支持自定义 API 地址（如 DeepSeek 官方 API）和 Key (`Bearer Token`)。


* **智能上下文 (Context Aware)**：
* 脚本将视频标题、话题、作者、标签、推荐词以及**前15条热门评论**打包发送给 AI，确保生成的回复紧贴视频内容和评论区氛围。


* **自动回复 (Auto Reply) ⚠️**：
* 新增 `processReplyLogic` 函数。开启后，脚本不仅能发表一级评论，还能自动回复评论区中其他用户的评论（排除作者和自己）。
* **防重机制**：使用 `repliedCidsCache` 缓存已回复的评论 ID (`cid`)，防止对同一条评论重复回复。


* **概率控制**：
* 保留了 `commentRate`（评论概率），新增 `aiTimeout`（超时设置），避免 AI 响应慢卡死流程。



### 2. 📡 全维度数据嗅探与采集 (Excel 导出)

脚本集成了 `xlsx.full.min.js`，通过 Hook 核心 API 实现了多维度的数据“无感”采集与导出。

* **👥 粉丝数据录制 (`handleFollowerList`)**：
* **功能**：在用户浏览粉丝列表时，自动后台记录粉丝的 UID、昵称、粉丝数、作品数、甚至**年龄、IP属地、星座**（通过计算出生日期）。
* **导出**：支持一键导出为 Excel，文件名为 `抖音粉丝数据_ID_时间.xlsx`。


* **🔍 搜索数据录制 (`handleSearchList`)**：
* **功能**：在搜索页面，自动捕获搜索结果中的用户信息（UID、抖音号、企业认证信息等）。
* **导出**：支持导出搜索结果数据。


* **🎞️ 弹幕数据采集 (`handleDanmakuList`)**：
* **功能**：捕获视频播放时的弹幕流，记录弹幕内容、发送时间、点赞数、是否广告等。
* **导出**：支持批量导出弹幕数据。


* **📝 评论深度抓取 (`processExportLogic`)**：
* **后台抓取**：点击“抓取评论”后，脚本会在后台静默请求 API (`comment/list`)，支持递归抓取**子评论**。
* **数据详尽**：导出字段包含评论层级、父评论ID、用户主页链接、IP属地等。



### 3. 🚀 弹幕交互系统 (新增)

* **发送弹幕 (`tryProcessDanmaku`)**：
* 支持在视频播放时自动发送弹幕。
* **双模式**：
1. **固定文本**：发送如 "666" 等固定内容。
2. **AI 生成**：根据视频内容，让 AI 生成“神评”弹幕。


* **概率与冷却**：受 `danmakuRate` 控制，并包含随机冷却时间 (`nextDanmakuCooldown`)，模拟真实用户观影节奏。



### 4. 📥 增强型媒体下载与 Motrix 集成

* **Motrix RPC 支持 (`sendToMotrix`)**：
* **功能**：除了浏览器直接下载，新增推送到 Motrix 下载器的功能。
* **原理**：通过 JSON-RPC 协议 (`aria2.addUri`) 将无水印高码率视频地址发送到本地端口 `16800`。这解决了浏览器下载大文件慢或不稳定的问题。


* **视频源解析**：
* 自动从 JSON 响应或 DOM (`recoverContextFromDOM`) 中提取最高码率的视频流地址。
* **Blob 解密兜底**：如果 API 抓取失败，尝试从 `<video>` 标签提取 Blob URL（虽然 Blob 较难直接下载，但脚本做了兼容性尝试）。



### 5. 🕹️ 自动化流程与内存管理 (企业级优化)

* **内存守护进程 (`startMemoryDaemon`)**：
* **功能**：定期清理过期的任务记录 (`activeScrapingSet`)、历史会话 (`sessionHistory`) 和视频缓存 (`dy_video_map`)。
* **目的**：防止长时间挂机导致浏览器内存溢出崩溃。


* **自动化状态机**：
* **脉冲检测**：`startBurstCheck` 持续监测视频切换。
* **自动下滑**：任务（评论/点赞/录制）完成后，根据设定的 `minTime`/`maxTime` 随机延迟后触发 `ArrowDown` 模拟下滑。
* **状态可视化**：悬浮球颜色对应不同状态（🟣空闲 🟡分析 🟢成功 🔴错误 🔵跳过 🔘暂停）。


* **防风控策略**：
* **随机间隔**：所有操作（评论、点赞、下滑）均加入随机抖动。
* **拦截唤起**：`blockExternalApps` 阻止网页唤起抖音客户端，保持网页端稳定运行。



### 6. ⚙️ 全新 UI 界面与统计看板

* **悬浮球与面板**：
* 可拖拽的悬浮球（⚡）。
* 全新的设置面板，包含 **高级设置折叠**、**暗黑模式**、**Tab 分页**。


* **实时数据看板**：
* 实时显示：已分析数、已评论数、已回复数、点赞/收藏数、广告跳过数、屏蔽数。
* 提供“清除所有统计数据”按钮。



---

## ⚠️ 风险提示与使用建议 (基于 v4.0.0 代码分析)

1. **高频采集风险**：
* 脚本新增的 **粉丝列表录制** 和 **搜索数据录制** 涉及高频访问敏感接口 (`user/follower/list`)。这极易触发抖音的 **滑块验证码** 或 **IP 封禁**。
* *建议*：不要长时间开启“粉丝录制”功能，仅在需要分析特定账号时手动开启。


2. **AI 回复功能风险**：
* `autoReplyBg`（后台自动回复）功能会扫描评论区并对他人评论进行回复。这种行为不仅容易被判定为机器人，还可能引起社区用户的反感（被举报骚扰）。
* *建议*：谨慎开启“自动回复”，并确保 AI 提示词（Prompt）足够友善且相关性强。


3. **Cookies 与 Token 依赖**：
* 脚本完全依赖 `msToken`、`a_bogus`、`verifyFp` 和 `cookie`。如果抖音升级加密算法（如更新 X-Bogus 生成逻辑），脚本的网络请求（评论、点赞、数据抓取）将全部失效（返回状态码 1024 或 8）。


4. **Motrix 配置**：
* 若使用 Motrix 下载，必须确保 Motrix 软件已启动，且在脚本设置中填写正确的 RPC 授权密钥（Token），否则请求会失败。



---

### 总结

v4.0.0 版本是一个质的飞跃，它不再局限于“自动刷视频”，而是变成了一个**全能型的抖音网页端数据分析与运营工具**。它适合运营人员进行竞品数据分析（抓取评论/粉丝）、内容创作者进行自动化活跃账号（自动回复/弹幕），但同时也带来了更高的账号风控风险。


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




# 更新日志
## 2026.01.08
1. 增加了 Motrix 辅助下载大视频能力(不开启默认下载半小时左右视频无压力,开启后可以下载2小时以上超大型视频)
2. 增加对于在线大模型的支持能力
3. 支持用户个人主页视频的自动回复能力(回复所有评论用户,包括子评论),增加容错,连续超过三次,自动失败
4. 增加父评论,子评论全面抓取能力,增加容错,连续超过三次,自动失败

## 2026.01.10
1. 配置面板界面布局全面优化，同类型整理到一起
2. 修复子评论乱回复问题（直接去掉了）
3. 增加接口数据录制功能（目前支持搜索作者，关注自己的粉丝，视频弹幕）
4. 增加弹幕回复能力（支持独立配置大模型提示词，独立设置概率。还支持固定文本回复模式）
5. 优化接口请求冷却时间（降低禁言概率）

### 全部功能展示
<img width="829" height="1554" alt="image" src="https://github.com/user-attachments/assets/1a836d71-d5fd-47eb-bca3-b9b7fa946e6f" />

<img width="829" height="1554" alt="image" src="https://github.com/user-attachments/assets/342e8273-2aff-44bf-b3e3-ca32f2e7c5ef" />

<img width="829" height="1554" alt="image" src="https://github.com/user-attachments/assets/0a8eaecf-9a9c-4534-a8b1-8f7ba9b70287" />

<img width="829" height="1554" alt="image" src="https://github.com/user-attachments/assets/e9bbd8c8-8080-4664-bb31-59b715172c1a" />






# 备注:
1. Motrix 安装
   中文官网:https://motrix.app/zh-CN/
   
   <img width="3700" height="2054" alt="image" src="https://github.com/user-attachments/assets/e4d71dac-4b2c-4ee8-b65d-73426762af2d" />
   
   英文官网:https://motrix.app/
   
   <img width="3700" height="2054" alt="image" src="https://github.com/user-attachments/assets/49da79cc-3010-4547-8467-98b859963ec3" />

   下载完成,安装包啥都不用额外配置,一路下一步就行了

   <img width="2084" height="1577" alt="image" src="https://github.com/user-attachments/assets/ee006b9e-0d1a-4c43-919a-6bd56a4d4610" />

   在进阶设置里,可以获取到RPC端口,以及授权密钥
   
   <img width="2084" height="1577" alt="image" src="https://github.com/user-attachments/assets/2130341e-2298-4942-a206-5d4052e4a8ef" />

   填入插件里,保存即可
   
   <img width="624" height="413" alt="image" src="https://github.com/user-attachments/assets/0b15cbd3-c0a2-40c4-b2bb-2c98fef3ff08" />

2. 下载安装油候脚本(谷歌浏览器为例)

   官网:https://www.tampermonkey.net/index.php?browser=chrome&locale=zh
   
   <img width="2013" height="818" alt="image" src="https://github.com/user-attachments/assets/48325be6-145b-43cf-8baa-a0fa05cf3e06" />
   
   一般装第一个就行,打不开自己行办法(能看到这的,不用教吧)


   
3. 安装谷歌浏览器
  官网:https://www.google.com/intl/zh-CN/chrome/
  打不开的自行查找方案(能看到这的,不用教吧)
  啥都不用改,全都下一步就行了

4. 脚本导入
   
   添加新脚本
   
   <img width="625" height="761" alt="image" src="https://github.com/user-attachments/assets/35ec38ad-0572-4ce3-a0f2-58ef45d33193" />
   
   全选,删掉,然后把我给你的文件里的内容全都粘贴过来保存就行了
   
   <img width="3825" height="1926" alt="image" src="https://github.com/user-attachments/assets/e6c9f9d1-53f1-48c3-a3b8-371e4d4e8012" />
   






