# CLAUDE.md

本文件為 Claude Code (claude.ai/code) 在此儲存庫中工作時提供指引。

## Monorepo 結構

本專案為全端 Spring AI 展示專案，包含兩個獨立子專案：

- `mySpringAi/` — Spring Boot 4.1.0 + Spring AI 2.0.0 後端（Java 25、Maven）。詳細後端指引請參閱 `mySpringAi/CLAUDE.md`。
- `mySpringAi-ui/` — React 19 + Vite 8 前端。詳細前端指引請參閱 `mySpringAi-ui/CLAUDE.md`。

兩個子專案獨立開發。前端透過 Vite 設定將 `/api/*` 代理至後端 `:8080`，開發環境無需設定 CORS。

## 常用指令

### 後端（`mySpringAi/`）
```bash
./mvnw spring-boot:run                          # 啟動 API，監聽 :8080（自動啟動 Docker 服務）
./mvnw test                                     # 執行所有測試
./mvnw -Dtest=AudioControllerTest test          # 執行單一測試類別
./mvnw clean package                            # 建置 JAR
```

### 前端（`mySpringAi-ui/`）
```bash
npm run dev       # 開發伺服器，監聽 :5173
npm run build
npm run lint
```

### 基礎設施（`mySpringAi/`）
```bash
docker compose up                        # 啟動 Qdrant、Redis、Prometheus、Grafana、Jaeger
```

## 架構說明

### 後端

每個功能為一個獨立的 Controller，對應一個預先設定好的 `ChatClient` Bean。`config/` 套件負責組合這些 Bean，組合元素包含：LLM 供應商（OpenAI 或 Ollama）+ 記憶體策略（無 / 記憶體內 / JDBC/H2）+ Advisor 鏈。

**Advisor 鏈** — Controller 透過在 `ChatClient` 呼叫上附加 Advisor 來組合行為，執行順序會影響結果：
- `MessageChatMemoryAdvisor` — 依 `userName` header + `conversationId` 限定範圍
- `RetrievalAugmentationAdvisor` — 從 Qdrant 進行 RAG 檢索
- `SemanticCacheAdvisor` — Redis 或 Qdrant 語意快取
- `TokenUsageAuditAdvisor` / `PrettyLoggerAdvisor` — 可觀測性

**RAG 變體**（`RagController`）：基礎版（rag-collection）、PDF 版（pdf-collection + 查詢改寫）、Web 版（Tavily 搜尋）、進階版（檢索前查詢翻譯 + 檢索後 PII 遮罩）。

**對話隔離**：HTTP header `userName` 作為 `conversationId`，依使用者與端點限定對話記憶體範圍。

**檔案輸出**：生成圖片 → `image-output/`（對外路徑 `/generated-images/**`），音訊 → `audio-output/`。

**API 金鑰**：將 `api.properties` 放置於 `mySpringAi/src/main/resources/`（已加入 .gitignore）。必要金鑰：`spring.ai.openai.api-key`、`spring.ai.tavily.api-key`（Web RAG 使用）。

### 前端

導覽選單完全由 `src/config/apiRoutes.js` 資料驅動。新增展示頁面需執行以下步驟：
1. 在 `apiRoutes.js` 新增項目
2. 在 `App.jsx` 新增路由
3. 建立薄包裝頁面元件，將 `endpoint`/`title`/`description` 傳入 `ChatBox`

全域狀態（使用者名稱 + 各端點訊息歷史）存放於 `DemoContext`，並同步至 `sessionStorage`。

## 關鍵埠號

| 服務 | 網址 |
|---|---|
| 後端 API | http://localhost:8080 |
| H2 主控台 | http://localhost:8080/h2-console |
| 前端 | http://localhost:5173 |
| Qdrant | http://localhost:6333 |
| Redis Insight | http://localhost:8001 |
| Grafana | http://localhost:3000 (admin/admin) |
| Jaeger | http://localhost:16686 |
