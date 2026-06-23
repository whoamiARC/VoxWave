# 舆澜 VoxWave

> 面向社会事件的舆情种子生成、模拟推演与风险研判系统。

[GitHub 仓库](https://github.com/whoamiARC/VoxWave)

舆澜将自然语言描述、事件标签和补充材料组织成可推演的舆情场景，并通过多智能体模拟生成传播路径、群体情绪、观点分化、风险拐点和处置建议。它适合用于校园、职场、公共安全、消费维权、社区治理、平台争议等社会议题的舆情预演。

## 功能亮点

- 舆情种子生成器：内置场景、角色、事件、行为、议题、平台、情绪、传播阶段、处置策略、风险类型和地域层级等多维种子。
- 自然语言提取：输入一段事件描述后，可自动匹配并转化为结构化种子词。
- 场景提示词编排：把已选种子、原始描述和补充资料组合成可直接用于图谱构建与仿真的提示词。
- 多智能体仿真：围绕不同角色生成发帖、评论、转发、质疑、回应和沉默等行为。
- 风险研判报告：输出舆情热度、关键节点、误读链条、二次传播点、风险等级与沟通建议。
- 极简工作台 UI：以“舆澜 / VoxWave”为品牌重新设计入口页、流程页和分析页。

## 技术栈

- Frontend: Vue 3, Vite, Vue Router, Vue I18n, D3
- Backend: Flask, Python 3.11
- LLM: OpenAI SDK compatible endpoint, default example uses DeepSeek
- Memory Graph: Zep Cloud
- Simulation: OASIS / CAMEL based social simulation runtime

## 快速开始

### 1. 安装依赖

```bash
npm install
cd frontend
npm install
cd ..
```

Python 后端建议使用虚拟环境：

```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
cd ..
```

### 2. 配置环境变量

复制示例配置：

```bash
copy .env.example .env
```

然后在 `.env` 中填入真实密钥：

```env
LLM_API_KEY=your_llm_api_key_here
LLM_BASE_URL=https://api.deepseek.com
LLM_MODEL_NAME=deepseek-v4-flash
ZEP_API_KEY=your_zep_api_key_here
```

注意：`.env` 已被 `.gitignore` 忽略，上传 GitHub 前不要提交任何真实 API Key。

### 3. 启动开发环境

分别启动后端和前端：

```bash
cd backend
.venv\Scripts\activate
python run.py
```

```bash
cd frontend
npm run dev
```

默认地址：

- Frontend: `http://localhost:3000`
- Backend: `http://localhost:5001`

也可以使用根目录脚本同时启动：

```bash
npm run dev
```

### 4. 构建前端

```bash
cd frontend
npm run build
```

## 使用流程

1. 在首页选择舆情种子，例如“职场、被裁员员工、匿名社区、发帖、监控软件、VPN、内网故障”。
2. 输入自然语言描述，或让系统从描述中提取种子词。
3. 上传可选补充资料，例如政策文件、公告、聊天记录整理或事件背景文本。
4. 生成图谱和仿真环境。
5. 设置模拟轮数并开始推演。
6. 查看报告，分析传播路径、观点分化、情绪峰值和处置建议。

## Docker

```bash
docker compose up --build
```

服务会暴露：

- `3000`: 前端页面
- `5001`: 后端 API

## GitHub 上传建议

仓库名建议使用：

```text
VoxWave
```

推荐首次提交：

```bash
git init
git add .
git commit -m "Initial VoxWave public opinion simulation workbench"
git branch -M main
git remote add origin https://github.com/whoamiARC/VoxWave.git
git push -u origin main
```

## 目录结构

```text
.
├── frontend/          # Vue 前端工作台
├── backend/           # Flask API 与仿真服务
├── locales/           # 中英文界面文案
├── static/            # 静态资源
├── docker-compose.yml
├── Dockerfile
└── README.md
```

## 致谢与许可

本项目基于 AGPL-3.0 许可发布，并在原多智能体仿真项目能力基础上面向舆情分析场景进行了 UI、流程和种子体系改造。仿真能力依赖 OASIS / CAMEL 等开源生态，感谢相关项目的贡献。
