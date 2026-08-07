# 智能AI面试官平台

基于 Spring Boot + Spring AI + PostgreSQL + Redis 的智能面试辅助系统，提供简历解析、AI出题、多轮追问、知识库问答等能力。

## 技术栈

- **后端框架**: Java 21 + Spring Boot 3.2+ + Spring AI
- **数据库**: PostgreSQL + pgvector（向量检索）
- **缓存/消息队列**: Redis + Redisson（Stream异步任务）
- **实时通信**: WebSocket（AI流式响应）
- **文档解析**: Apache Tika（多格式简历解析）
- **AI模型**: DashScope、DeepSeek等多Provider统一调用

## 核心功能

### 简历管理模块
- **多格式解析**: 支持 PDF、DOCX、DOC、TXT 等简历格式
- **异步处理流**: 基于 Redis Stream 实现异步简历分析，支持实时进度查看
- **稳定性保障**: 内置分析失败自动重试（最多3次）与内容哈希重复检测
- **分析报告导出**: AI分析结果一键导出为结构化PDF报告

### 模拟面试模块
- **Skill驱动出题**: 内置10+面试方向（Java后端、前端、算法、系统设计等）
- **历史题目去重**: 出题时自动排除已问题目，避免重复考察
- **智能追问流**: 支持配置多轮智能追问，模拟真实面试场景
- **统一评估架构**: 文字面试和语音面试共用评估引擎

### 知识库模块（RAG）
- **向量检索**: 基于pgvector实现知识库向量化存储与相似度搜索
- **混合检索策略**: 向量检索 + 关键词检索，结果合并后统一排序
- **知识库题库面试**: 基于知识库内容自动生成面试题

### 语音面试模块
- **实时语音**: WebSocket双向通信，支持语音面试
- **语音识别/合成**: 集成Qwen3 ASR/TTS
- **流式响应**: AI回答实时推送到前端

## 项目结构

```
interview-guide/
├── app/                              # 后端应用
│   └── src/main/java/interview/guide/
│       ├── modules/
│       │   ├── resume/              # 简历模块
│       │   │   ├── service/         # 上传、解析、持久化
│       │   │   └── listener/        # Redis Stream消费者
│       │   ├── interview/           # 面试模块
│       │   │   ├── service/         # 出题、评估、持久化
│       │   │   └── controller/      # 面试接口
│       │   ├── knowledgebase/       # 知识库模块
│       │   │   ├── service/         # 向量化、问答生成
│       │   │   └── controller/      # 知识库接口
│       │   ├── voiceinterview/      # 语音面试模块
│       │   │   ├── service/         # ASR/TTS、面试服务
│       │   │   ├── handler/         # WebSocket处理器
│       │   │   └── listener/        # Stream生产者/消费者
│       │   └── llmprovider/         # LLM Provider模块
│       └── infrastructure/          # 基础设施
│           ├── file/                # 文件存储、文档解析
│           └── redis/               # Redis配置
├── frontend/                         # 前端应用（React + TypeScript）
├── docker/                           # Docker配置
└── docs/                             # 项目文档
```

## 技术亮点

### 1. RAG知识库 - 混合检索策略
```java
// 向量检索 + 关键词检索，结果合并排序
List<Document> vectorResults = vectorStore.similaritySearch(query);
List<Document> keywordResults = keywordSearch(query);
List<Document> merged = reciprocalRankFusion(vectorResults, keywordResults);
```

### 2. Redis Stream 异步任务队列
```java
// 简历分析异步处理
redisTemplate.opsForStream().add("resume:analyze", resumeTask);
// 消费者组消费
redisTemplate.opsForStream().read(consumer, StreamReadOptions.empty().count(1), "resume:analyze");
```

### 3. WebSocket 实时通信
```java
// AI流式响应实时推送
@ServerEndpoint("/interview/{sessionId}")
public class InterviewWebSocketHandler {
    // 处理语音/文字面试的实时双向通信
}
```

### 4. 多Provider统一调用
```java
// 统一调用层，支持DashScope、DeepSeek等多个大模型
public class LlmProviderRegistry {
    public ChatClient getProvider(String providerName) {
        return providers.get(providerName);
    }
}
```

## 快速启动

```bash
# 1. 克隆项目
git clone https://github.com/b3stellung/interview-guide.git

# 2. 启动基础设施
docker-compose up -d

# 3. 配置API Key
# 编辑 application.yml 配置 DashScope/DeepSeek API Key

# 4. 启动后端
./gradlew bootRun

# 5. 启动前端
cd frontend && pnpm install && pnpm dev
```

## 环境要求

- JDK 21+
- PostgreSQL 14+（pgvector扩展）
- Redis 6+
- Node.js 18+（前端）
