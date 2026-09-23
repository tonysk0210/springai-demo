# CLAUDE.md

本文件為 Claude Code (claude.ai/code) 在此儲存庫中工作時提供指引。

## 儲存庫結構

本專案為 monorepo，包含兩個獨立子專案：
- `mySpringAi/` — Spring Boot 4.1.0 + Spring AI 2.0.0 後端（Java 25、Maven）
- `mySpringAi-ui/` — React 19 + Vite 8 前端

## 常用指令

### 後端（`mySpringAi/`）
```bash
# 啟動（監聽 :8080，自動透過 compose.yml 啟動 Docker 服務）
./mvnw spring-boot:run

# 打包建置
./mvnw clean package

# 執行測試
./mvnw test
```

### 前端（`mySpringAi-ui/`）
```bash
npm run dev      # 開發伺服器，監聽 :5173，將 /api 代理至 :8080
npm run build
npm run lint
```

### 基礎設施
```bash
# 在 mySpringAi/ 目錄下執行，啟動 Qdrant、Redis、Prometheus、Grafana、Jaeger
docker compose up
```

## 架構說明

### 後端

後端透過各自獨立的 Controller，展示多種 Spring AI 整合模式，每個 Controller 都有對應的 `ChatClient` Bean 配置。

**ChatClient Bean**（`config/`）是架構核心，每個 Bean 組合了特定的模型 + 記憶體 + Advisor 堆疊：
- `openaiCCNoMem` — 無狀態 OpenAI 客戶端
- `openaiCCInMemory` — 記憶體內對話歷史
- `openaiCCJdbcMemory` — H2 持久化對話歷史（預設聊天）
- `ollamaCCJdbcMemory` — 本地 Ollama 模型搭配 JDBC 記憶體
- `openaiCCNoMemRedisCache` / `openaiCCNoMemQdrantCache` — 語意快取
- `openaiCCJdbcMemoryWithToolCalling` — 透過 MCP 執行工具呼叫

**Advisor 鏈模式**：Controller 透過在 `ChatClient` 呼叫鏈上附加 Advisor 來組合行為——`MessageChatMemoryAdvisor`、`RetrievalAugmentationAdvisor`、`SemanticCacheAdvisor`、`TokenUsageAuditAdvisor`、`PrettyLoggerAdvisor`。Advisor 的順序會影響執行結果。

**RAG 管線**（`controller/RagController`）：多種策略透過 `RetrievalAugmentationAdvisor` 串接。進階版本支援檢索前查詢改寫（中文→英文翻譯）及檢索後 PII 遮罩。向量資料庫使用 Qdrant（`rag-collection`、`pdf-collection`、`caching-collection`）。

**關鍵設定檔：**
- `src/main/resources/application.properties` — LLM 端點、H2/Qdrant/Redis 設定、檔案上傳限制、OTel 取樣率
- `src/main/resources/api.properties`（選用，已加入 .gitignore）— OpenAI 與 Tavily 的 API 金鑰

**對話身份識別**：Controller 讀取 HTTP header `userName` 來區隔對話記憶體範圍（搭配 `MessageChatMemoryAdvisor` 的 `conversationId` 使用）。

**檔案輸出**：生成圖片 → `image-output/`（對外提供路徑為 `/generated-images/**`），音訊 → `audio-output/`。

### 前端

`DemoContext`（`context/`）存放全域狀態：`userName` 與 `messagesByUserAndEndpoint`（持久化至 `sessionStorage`）。

大多數頁面是薄包裝層，僅將 API 路徑傳入共用的 `ChatBox` 元件。API 路徑統一定義於 `src/config/apiRoutes.js`；各端點的測試說明文件位於 `src/config/apiTestGuides.js`。

Vite 將所有 `/api` 請求代理至 `:8080`，開發環境無需設定 CORS。

## 關鍵埠號與開發網址

| 服務 | 網址 |
|---|---|
| 後端 API | http://localhost:8080 |
| H2 主控台 | http://localhost:8080/h2-console |
| Actuator / Prometheus | http://localhost:8080/actuator/prometheus |
| 前端開發伺服器 | http://localhost:5173 |
| Qdrant UI | http://localhost:6333 |
| Redis Insight | http://localhost:8001 |
| Grafana | http://localhost:3000 |
| Jaeger UI | http://localhost:16686 |
