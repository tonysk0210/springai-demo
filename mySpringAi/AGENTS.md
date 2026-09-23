# 儲存庫指南

## 專案結構與模組組織

本專案是以 Java 25、Spring Boot 與 Spring AI 建置的單一模組 Maven 應用程式。正式程式碼位於 `src/main/java/com/example/mySpringAi`，並依職責分為 `controller`、`service`、`repo`、`entity`、`dto`、`payload`、`config`、`advisor`、`tools` 與 `util`。執行期設定、提示詞範本及範例文件位於 `src/main/resources`；StringTemplate 提示詞應放在 `promptTemplate/`。測試位於 `src/test/java`，其套件結構應對應正式程式碼。Docker 服務定義於 `compose.yml`。請勿提交產生的資料及建置輸出，例如 `target/`、`logs/`、`h2db/`、`audio-output/` 與 `image-output/`。

## 建置、測試與開發指令

請使用 Maven Wrapper，確保所有貢獻者使用相同的 Maven 版本：

- `./mvnw spring-boot:run`（Windows：`.\mvnw.cmd spring-boot:run`）：在本機啟動 API，並可能啟動已設定的 Compose 相依服務。
- `./mvnw test`：執行完整測試套件。
- `./mvnw -Dtest=AudioControllerTest test`：開發期間只執行指定的測試類別。
- `./mvnw clean package`：編譯、測試，並在 `target/` 產生可執行 JAR。
- `docker compose up -d`：啟動 Qdrant、Redis、Jaeger、Prometheus 與 Grafana；使用完畢後執行 `docker compose down`。

## 程式碼風格與命名慣例

採用四個空格縮排及標準 Java 格式。套件名稱應使用小寫，但須保留現有根套件拼法 `com.example.mySpringAi`。類別與 record 使用 PascalCase，方法與欄位使用 camelCase，常數使用 `UPPER_SNAKE_CASE`。遵循既有的 `*Controller`、`*Service`、`*Repository`、`*Config`、`*Dto` 與 `*Payload` 後綴。優先使用建構式注入；專案目前已使用 Lombok 的 `@RequiredArgsConstructor`。本專案未設定自動格式化或靜態檢查工具，因此提交前應參照鄰近程式碼並整理 import。

## 測試準則

測試透過 `spring-boot-starter-test` 使用 JUnit 5、Mockito 與 AssertJ。測試類別命名為 `*Test`，測試方法應描述可觀察行為，例如 `textToSpeechReturnsInlineMp3Bytes`。新增分支或錯誤處理時，應加入聚焦的單元測試；只有需要驗證應用程式 wiring 時才使用 `@SpringBootTest`。目前未強制設定覆蓋率門檻。

## Commit 與 Pull Request 準則

近期歷史多使用 `update` 等過短訊息；請改用簡潔、命令式且能描述變更的主旨，例如 `Add audio format validation`。每個 commit 應只涵蓋一項邏輯變更。Pull request 必須說明目的與行為差異、列出驗證指令、連結相關 issue，並標示設定或 Compose 異動。API 變更應附上請求與回應範例；只有涉及可見輸出或監控儀表板時才需提供截圖。

## 安全性與設定

機密資料應透過 `OPENAI_API_KEY` 等環境變數提供。將 `src/main/resources/api.properties` 視為僅供本機使用的機密設定；切勿提交真實憑證、產生的媒體、資料庫檔案或日誌。
