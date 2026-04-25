# SuperBizAgent

基于 Spring Boot + Spring AI 的企业级智能业务代理系统，包含 RAG 智能问答和 AIOps 智能运维两大模块。

## 技术栈

- Java 17, Spring Boot 3.2.0
- Spring AI 1.1.0 + Spring AI Alibaba 1.1.0.0-RC2
- 阿里云 DashScope（qwen3-max 模型）
- Milvus 2.6.10 向量数据库
- Maven 构建

## 构建与运行

```bash
# 启动向量数据库
docker compose up -d -f vector-database.yml

# 构建并运行
mvn clean install
mvn spring-boot:run

# 或一键启动（含文档上传）
make init
```

必须设置环境变量：
```bash
export DASHSCOPE_API_KEY=your-api-key
```

服务端口：`9900`

## 项目结构

```
src/main/java/org/example/
├── controller/       # HTTP 接口（ChatController, FileUploadController）
├── service/          # 业务逻辑（ChatService, AiOpsService, RagService, Vector*）
├── agent/tool/       # Agent 工具（DateTimeTools, InternalDocsTools, QueryMetricsTools, QueryLogsTools）
├── config/           # 配置类（Milvus, DashScope, Web 等）
├── dto/              # 数据传输对象
└── constant/         # 常量
src/main/resources/
├── static/           # Web 界面（index.html, app.js, styles.css）
└── application.yml   # 应用配置
aiops-docs/           # 运维知识库文档
```

## 核心 API

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/chat` | 普通对话 |
| POST | `/api/chat_stream` | SSE 流式对话 |
| POST | `/api/ai_ops` | AIOps 运维分析（SSE） |
| POST | `/api/chat/clear` | 清空会话 |
| POST | `/api/upload` | 上传文件并向量化 |
| GET  | `/milvus/health` | Milvus 健康检查 |

请求体示例：
```json
{ "Id": "session-123", "Question": "你的问题" }
```

## 架构说明

- **RAG 模块**：文档向量化存入 Milvus → 查询时检索相关片段 → 拼入 prompt 调用 DashScope
- **AIOps 模块**：Planner-Executor-Replanner 架构，Agent 自动调用工具（告警查询、日志分析、文档检索）生成运维报告
- **会话管理**：基于 sessionId 维护多轮对话上下文，支持自动清理
