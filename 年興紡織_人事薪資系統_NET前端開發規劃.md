# 年興紡織人事薪資系統 .NET 前端開發計劃書

> **版本：** v2.0  
> **日期：** 2025-01-26  
> **作者：** 開發團隊  
> **基於：** 年興紡織_人事薪資系統_規格書_v_2.md

---

## 📋 目錄

1. [專案概述](#專案概述)
2. [整體系統架構與流程](#整體系統架構與流程)
3. [技術堆疊選型](#技術堆疊選型)
4. [十大功能模組流程設計](#十大功能模組流程設計)
5. [AS400 DB2 整合架構](#as400-db2-整合架構)
6. [系統資料流程](#系統資料流程)
7. [用戶操作流程](#用戶操作流程)
8. [所需材料與資源清單](#所需材料與資源清單)
9. [風險控制](#風險控制)
10. [成功標準](#成功標準)

---

## 🎯 專案概述

### 專案目標
基於年興紡織現有人事薪資系統規格書，開發全新的 **.NET 前端應用程式**，保持所有原有功能模組完整性，並確保與 **AS400 DB2** 系統的無縫整合。

### 核心需求
- ✅ **完整功能對應**：實作規格書中的 M01-M10 所有模組
- ✅ **現代化介面**：採用 Blazor Server 技術提供現代化用戶體驗
- ✅ **資料整合**：與 AS400 DB2/400 資料庫完全整合
- ✅ **效能優化**：支援 10k+ 員工的高效能處理
- ✅ **安全合規**：符合 SOX、ISO 27001、個資法等規範

---

## 🏗️ 整體系統架構與流程

### 系統整體架構圖

```mermaid
graph TB
    subgraph "用戶端層"
        UA["HR人員<br/>Chrome/Edge瀏覽器"]
        UB["部門主管<br/>審核介面"]
        UC["財務人員<br/>核准介面"]
        UD["一般員工<br/>查詢介面"]
    end
    
    subgraph ".NET前端層"
        WEB["Blazor Server<br/>統一入口"]
        API["Web API Gateway<br/>服務閘道"]
        SIG["SignalR Hub<br/>即時推播"]
        AUTH["認證授權中心<br/>JWT+Session"]
    end
    
    subgraph "業務服務層"
        M01["M01 基礎資料管理"]
        M02["M02 考勤資料處理"]
        M03["M03 薪資參數設定"]
        M04["M04 薪資計算批次"]
        M05["M05 薪資審核核准"]
        M06["M06 報表與匯出"]
        M07["M07 檔案交換中心"]
        M08["M08 安全與權限"]
        M09["M09 稽核與日誌"]
        M10["M10 系統設定監控"]
    end
    
    subgraph "資料整合層"
        CACHE["Redis快取<br/>Session/權限"]
        DB2CONN["DB2連接池<br/>連線管理"]
        FILEPROC["檔案處理<br/>匯入匯出"]
    end
    
    subgraph "AS400 IBM i"
        DB2["DB2/400資料庫<br/>員工/薪資/考勤"]
        RPG["RPG批次程式<br/>薪資計算引擎"]
        IFS["IFS檔案系統<br/>DAT/CSV/PDF"]
        SFTP["SFTP伺服器<br/>檔案交換"]
    end
    
    subgraph "外部系統"
        BANK["銀行SFTP<br/>匯款檔傳輸"]
        GOV["政府系統<br/>稅務申報"]
        ERP["ERP系統<br/>GL分錄"]
    end
    
    UA --> WEB
    UB --> WEB
    UC --> WEB
    UD --> WEB
    
    WEB --> AUTH
    WEB --> API
    API --> SIG
    
    AUTH --> M08
    API --> M01
    API --> M02
    API --> M03
    API --> M04
    API --> M05
    API --> M06
    API --> M07
    API --> M09
    API --> M10
    
    M01 --> CACHE
    M02 --> CACHE
    M01 --> DB2CONN
    M02 --> DB2CONN
    M03 --> DB2CONN
    M04 --> DB2CONN
    M05 --> DB2CONN
    M06 --> FILEPROC
    M07 --> FILEPROC
    
    DB2CONN --> DB2
    M04 --> RPG
    FILEPROC --> IFS
    M07 --> SFTP
    
    SFTP --> BANK
    M06 --> GOV
    M06 --> ERP
```

### 系統總體業務流程圖

```mermaid
flowchart TD
    START([系統啟動]) --> LOGIN[用戶登入驗證]
    LOGIN --> ROLE{權限角色判斷}
    
    ROLE -->|HR人員| HR_FLOW[人事資料維護流程]
    ROLE -->|部門主管| DEPT_FLOW[部門審核流程]
    ROLE -->|財務人員| FIN_FLOW[財務核准流程]
    ROLE -->|一般員工| EMP_FLOW[員工查詢流程]
    
    HR_FLOW --> EMP_MAINT[員工資料維護]
    HR_FLOW --> ATT_IMPORT[考勤資料匯入]
    HR_FLOW --> PAY_PARAM[薪資參數設定]
    
    EMP_MAINT --> AUDIT_LOG[稽核日誌記錄]
    ATT_IMPORT --> ATT_VALID[考勤資料驗證]
    PAY_PARAM --> PARAM_HIST[參數歷史記錄]
    
    ATT_VALID --> BATCH_READY{批次計算準備就緒?}
    PARAM_HIST --> BATCH_READY
    
    BATCH_READY -->|是| PAY_BATCH[啟動薪資批次計算]
    BATCH_READY -->|否| WAIT_DATA[等待資料完整]
    
    PAY_BATCH --> BATCH_CALC[RPG批次計算]
    BATCH_CALC --> CALC_RESULT[計算結果產生]
    
    CALC_RESULT --> DEPT_REVIEW[部門主管審核]
    DEPT_FLOW --> DEPT_REVIEW
    DEPT_REVIEW --> DEPT_APPROVE{部門審核通過?}
    
    DEPT_APPROVE -->|通過| HR_REVIEW[HR複核]
    DEPT_APPROVE -->|退回| BATCH_ADJUST[批次調整]
    
    HR_REVIEW --> FIN_APPROVE[財務最終核准]
    FIN_FLOW --> FIN_APPROVE
    FIN_APPROVE --> FIN_OK{財務核准通過?}
    
    FIN_OK -->|通過| GENERATE_OUTPUT[產生輸出檔案]
    FIN_OK -->|退回| BATCH_ADJUST
    
    GENERATE_OUTPUT --> PAYSLIP[薪資單PDF]
    GENERATE_OUTPUT --> BANK_FILE[銀行匯款檔]
    GENERATE_OUTPUT --> TAX_FILE[稅務申報檔]
    GENERATE_OUTPUT --> GL_POST[ERP GL分錄]
    
    PAYSLIP --> EMP_QUERY[員工查詢薪資單]
    EMP_FLOW --> EMP_QUERY
    
    BANK_FILE --> SFTP_SEND[SFTP傳送銀行]
    TAX_FILE --> GOV_REPORT[政府申報]
    GL_POST --> ERP_INT[ERP整合]
    
    SFTP_SEND --> BANK_ACK[銀行回饋確認]
    BANK_ACK --> PROCESS_COMPLETE[流程完成]
    
    BATCH_ADJUST --> WAIT_DATA
    AUDIT_LOG --> COMPLIANCE[合規稽核]
    
    PROCESS_COMPLETE --> ARCHIVE[資料歸檔]
    ARCHIVE --> NEXT_CYCLE[下期循環]
    NEXT_CYCLE --> WAIT_DATA
```

### 資料流向架構圖

```mermaid
graph LR
    subgraph "資料來源"
        A1[員工異動表單]
        A2[門禁打卡DAT檔]
        A3[請假加班申請]
        A4[薪資參數設定]
    end
    
    subgraph "資料處理"
        B1[員工主檔更新]
        B2[考勤資料驗證]
        B3[參數版本控制]
        B4[批次計算觸發]
    end
    
    subgraph "計算引擎"
        C1[基本薪資計算]
        C2[加班費計算]
        C3[各項扣款計算]
        C4[淨薪資計算]
    end
    
    subgraph "審核流程"
        D1[部門主管審核]
        D2[HR複核確認]
        D3[財務最終核准]
    end
    
    subgraph "輸出產生"
        E1[員工薪資單PDF]
        E2[銀行匯款檔案]
        E3[稅務申報XML]
        E4[ERP GL分錄]
        E5[稽核報表]
    end
    
    subgraph "外部傳輸"
        F1[銀行SFTP]
        F2[政府稅務系統]
        F3[ERP財務系統]
    end
    
    A1 --> B1
    A2 --> B2
    A3 --> B2
    A4 --> B3
    
    B1 --> B4
    B2 --> B4
    B3 --> B4
    
    B4 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4
    
    C4 --> D1
    D1 --> D2
    D2 --> D3
    
    D3 --> E1
    D3 --> E2
    D3 --> E3
    D3 --> E4
    D3 --> E5
    
    E2 --> F1
    E3 --> F2
    E4 --> F3
```

---

## 🛠️ 技術堆疊選型

### 核心技術框架

| 層級 | 技術選型 | 版本 | 說明 |
|------|----------|------|------|
| **前端框架** | Blazor Server | .NET 8 | 伺服器端渲染，豐富互動體驗 |
| **API框架** | ASP.NET Core Web API | .NET 8 | RESTful API + OpenAPI 文檔 |
| **即時通訊** | SignalR | .NET 8 | 批次進度、審核通知即時推送 |
| **ORM框架** | Entity Framework Core | 8.0 | 物件關聯對應，支援 DB2 Provider |
| **DB2連接** | IBM.Data.DB2.Core | 最新版 | 原生 DB2 連接驅動程式 |
| **快取系統** | Redis | 7.x | Session 管理、權限快取 |
| **認證授權** | ASP.NET Core Identity | .NET 8 | JWT + Cookie 混合認證 |

### 輔助工具與套件

| 用途 | 工具/套件 | 版本 | 說明 |
|------|-----------|------|------|
| **工作排程** | Hangfire | 1.8+ | 背景任務與排程處理 |
| **檔案處理** | NPOI / ClosedXML | 最新版 | Excel 匯入匯出處理 |
| **PDF產生** | QuestPDF | 最新版 | 高效能薪資單 PDF 產生 |
| **日誌系統** | Serilog + Seq | 最新版 | 結構化日誌記錄與查詢 |
| **監控系統** | Application Insights | 最新版 | 應用程式效能監控 |
| **測試框架** | xUnit + Moq | 最新版 | 單元測試與模擬框架 |
| **API文檔** | Swagger/OpenAPI | .NET 8 | 自動化 API 文檔產生 |

### 開發工具建議

```yaml
IDE: Visual Studio 2022 Enterprise
版控: Git + Azure DevOps / GitHub
CI/CD: Azure DevOps Pipelines
容器: Docker + Kubernetes
監控: Application Insights + Grafana
```

---

## 📦 十大功能模組流程設計

### M01 - 基礎資料管理模組

#### 🎯 核心功能
- 員工主檔維護 (支援歷史版本追蹤)
- 組織架構樹狀管理
- 職級與薪級設定
- 批次匯入匯出功能
- 智能搜尋整合

#### 📊 模組業務流程圖

```mermaid
flowchart TD
    START([M01基礎資料管理]) --> LOGIN_CHECK{權限驗證}
    LOGIN_CHECK -->|HR Editor| MAIN_MENU[主功能選單]
    LOGIN_CHECK -->|無權限| ACCESS_DENY[存取拒絕]
    
    MAIN_MENU --> EMP_MAINT[員工資料維護]
    MAIN_MENU --> ORG_MAINT[組織架構維護]
    MAIN_MENU --> GRADE_MAINT[職級薪級維護]
    MAIN_MENU --> BULK_IMPORT[批次匯入作業]
    MAIN_MENU --> SEARCH_EMP[員工搜尋查詢]
    
    EMP_MAINT --> EMP_NEW[新增員工]
    EMP_MAINT --> EMP_EDIT[修改員工]
    EMP_MAINT --> EMP_TERM[員工離職]
    EMP_MAINT --> EMP_HIST[歷史異動查詢]
    
    EMP_NEW --> VALID_DATA{資料驗證}
    EMP_EDIT --> VALID_DATA
    VALID_DATA -->|通過| GEN_EMPID[產生員工編號]
    VALID_DATA -->|失敗| SHOW_ERROR[顯示錯誤訊息]
    
    GEN_EMPID --> ENCRYPT_SENS[敏感資料加密]
    ENCRYPT_SENS --> SAVE_DB[儲存至DB2]
    SAVE_DB --> AUDIT_LOG[記錄稽核日誌]
    AUDIT_LOG --> NOTIFY_DOWN[通知下游系統]
    
    ORG_MAINT --> ORG_CREATE[建立部門]
    ORG_MAINT --> ORG_MOVE[部門搬遷]
    ORG_MAINT --> ORG_DELETE[刪除部門]
    
    ORG_MOVE --> CHECK_CHILD{檢查子部門}
    CHECK_CHILD -->|有子部門| CASCADE_UPDATE[連動更新]
    CHECK_CHILD -->|無子部門| DIRECT_UPDATE[直接更新]
    
    BULK_IMPORT --> UPLOAD_FILE[上傳CSV檔案]
    UPLOAD_FILE --> PARSE_CSV[解析CSV內容]
    PARSE_CSV --> BATCH_VALID[批次資料驗證]
    BATCH_VALID --> SHOW_PREVIEW[顯示預覽結果]
    SHOW_PREVIEW --> CONFIRM_IMPORT{確認匯入}
    CONFIRM_IMPORT -->|確認| BATCH_PROCESS[批次處理]
    CONFIRM_IMPORT -->|取消| CANCEL_IMPORT[取消匯入]
    
    SEARCH_EMP --> INPUT_CRITERIA[輸入搜尋條件]
    INPUT_CRITERIA --> ES_SEARCH[ElasticSearch搜尋]
    ES_SEARCH --> SHOW_RESULTS[顯示搜尋結果]
    SHOW_RESULTS --> SELECT_EMP[選擇員工]
    SELECT_EMP --> VIEW_DETAIL[檢視詳細資料]
```

#### 📋 員工維護詳細流程

```mermaid
sequenceDiagram
    participant User as HR人員
    participant Web as Blazor前端
    participant API as Web API
    participant Auth as 權限驗證
    participant DB2 as DB2資料庫
    participant Audit as 稽核服務
    
    User->>Web: 開啟員工維護頁面
    Web->>Auth: 驗證權限(ROLE_HR_EDITOR)
    Auth-->>Web: 權限確認
    
    User->>Web: 輸入員工資料
    Web->>API: POST /api/employees
    API->>API: 資料驗證(身份證號格式等)
    
    alt 驗證通過
        API->>DB2: 檢查身份證號重複
        DB2-->>API: 查詢結果
        
        alt 無重複
            API->>API: 產生員工編號(雪花算法)
            API->>API: 敏感資料加密(身份證號/銀行帳號)
            API->>DB2: INSERT EMPLOYEE_MASTER
            DB2-->>API: 儲存成功
            API->>Audit: 記錄新增日誌
            API-->>Web: 成功回應
            Web-->>User: 顯示成功訊息
        else 身份證號重複
            API-->>Web: 錯誤:重複身份證號
            Web-->>User: 顯示錯誤訊息
        end
    else 驗證失敗
        API-->>Web: 驗證錯誤清單
        Web-->>User: 顯示驗證錯誤
    end
```

#### 🏗️ 組織架構管理流程

```mermaid
graph TD
    ORG_START([組織架構管理]) --> TREE_VIEW[載入組織樹]
    TREE_VIEW --> USER_ACTION{使用者操作}
    
    USER_ACTION -->|新增部門| ADD_DEPT[新增部門]
    USER_ACTION -->|編輯部門| EDIT_DEPT[編輯部門]
    USER_ACTION -->|刪除部門| DEL_DEPT[刪除部門]
    USER_ACTION -->|搬遷部門| MOVE_DEPT[搬遷部門]
    
    ADD_DEPT --> SELECT_PARENT[選擇上級部門]
    SELECT_PARENT --> INPUT_INFO[輸入部門資訊]
    INPUT_INFO --> VALID_ORG{驗證組織資料}
    VALID_ORG -->|通過| CHECK_DEPTH{檢查組織深度}
    VALID_ORG -->|失敗| ORG_ERROR[顯示錯誤]
    
    CHECK_DEPTH -->|≤10層| CREATE_ORG[建立部門]
    CHECK_DEPTH -->|>10層| DEPTH_ERROR[深度超限錯誤]
    
    CREATE_ORG --> UPDATE_CLOSURE[更新Closure Table]
    UPDATE_CLOSURE --> REGEN_COST[重新產生成本中心]
    
    MOVE_DEPT --> SELECT_NEW_PARENT[選擇新上級]
    SELECT_NEW_PARENT --> CHECK_LOOP{檢查循環依賴}
    CHECK_LOOP -->|無循環| MOVE_CONFIRM[確認搬遷]
    CHECK_LOOP -->|有循環| LOOP_ERROR[循環依賴錯誤]
    
    MOVE_CONFIRM --> COUNT_AFFECT{影響員工數}
    COUNT_AFFECT -->|<1000| AUTO_MOVE[自動搬遷]
    COUNT_AFFECT -->|≥1000| MANUAL_CONFIRM[手動二次確認]
    
    DEL_DEPT --> CHECK_EMP{檢查部門員工}
    CHECK_EMP -->|有員工| CANNOT_DELETE[無法刪除]
    CHECK_EMP -->|無員工| CHECK_SUB{檢查子部門}
    CHECK_SUB -->|有子部門| CASCADE_DEL[連動刪除確認]
    CHECK_SUB -->|無子部門| SAFE_DELETE[安全刪除]
```

#### 📥 批次匯入作業流程

```mermaid
flowchart TD
    IMPORT_START([批次匯入作業]) --> UPLOAD[上傳CSV檔案]
    UPLOAD --> FILE_CHECK{檔案檢查}
    
    FILE_CHECK -->|格式正確| PARSE[解析CSV內容]
    FILE_CHECK -->|格式錯誤| FORMAT_ERROR[格式錯誤]
    
    PARSE --> ROW_VALIDATE[逐列資料驗證]
    ROW_VALIDATE --> CREATE_SUMMARY[建立驗證摘要]
    
    CREATE_SUMMARY --> SHOW_PREVIEW[顯示預覽畫面]
    SHOW_PREVIEW --> PREVIEW_TABLE[預覽表格]
    PREVIEW_TABLE --> ERROR_LIST[錯誤清單]
    
    ERROR_LIST --> USER_DECISION{使用者決定}
    USER_DECISION -->|修正後重新上傳| UPLOAD
    USER_DECISION -->|忽略錯誤繼續| CONFIRM_IMPORT
    USER_DECISION -->|取消作業| CANCEL_IMPORT[取消匯入]
    
    CONFIRM_IMPORT --> BATCH_PROCESS[批次處理]
    BATCH_PROCESS --> PROGRESS_UPDATE[進度更新]
    PROGRESS_UPDATE --> PROCESS_ROW[處理單列資料]
    
    PROCESS_ROW --> ENCRYPT_DATA[加密敏感資料]
    ENCRYPT_DATA --> INSERT_DB[插入資料庫]
    INSERT_DB --> UPDATE_COUNT[更新處理計數]
    UPDATE_COUNT --> MORE_ROWS{還有資料?}
    
    MORE_ROWS -->|是| PROCESS_ROW
    MORE_ROWS -->|否| IMPORT_COMPLETE[匯入完成]
    
    IMPORT_COMPLETE --> SEND_EMAIL[發送完成通知]
    SEND_EMAIL --> AUDIT_RECORD[記錄稽核日誌]
```

#### 📊 資料流與整合點

| 資料表 | 主要欄位 | 整合關係 | 備註 |
|--------|----------|----------|------|
| **EMPLOYEE_MASTER** | EMP_ID, ID_NO, BANK_ACC | → M02考勤模組<br/>→ M04薪資計算 | 員工主檔，支援Temporal版本 |
| **ORG_UNIT** | ORG_ID, PARENT_ID, COST_CENTER | → M05審核流程<br/>→ M08權限控制 | 組織架構，Closure Table設計 |
| **GRADE_BAND** | GRADE_ID, MIN_PAY, MAX_PAY | → M03薪資參數<br/>→ M04薪資計算 | 職級薪級對照表 |
| **EMP_ATTACHMENT** | ATTACH_ID, FILE_PATH | → IFS檔案系統 | 員工附件(照片、證件) |

---

### M02 - 考勤資料處理模組

#### 🎯 核心功能
- DAT 檔案匯入與解析
- 考勤異常檢核與處理
- AI 智能補登建議
- 即時考勤儀表板
- 加班時數計算與審核

#### 📊 考勤處理主流程圖

```mermaid
flowchart TD
    START([M02考勤資料處理]) --> AUTH_CHECK{權限驗證}
    AUTH_CHECK -->|HR/考勤管理員| MAIN_MENU[考勤處理選單]
    AUTH_CHECK -->|無權限| ACCESS_DENY[存取拒絕]
    
    MAIN_MENU --> DAT_IMPORT[DAT檔案匯入]
    MAIN_MENU --> ATT_DASHBOARD[考勤儀表板]
    MAIN_MENU --> EXCEPTION_HANDLE[異常處理]
    MAIN_MENU --> AI_COMPLETION[AI智能補登]
    MAIN_MENU --> OT_CALC[加班時數計算]
    MAIN_MENU --> MONTHLY_CLOSE[月結作業]
    
    DAT_IMPORT --> UPLOAD_DAT[上傳DAT檔案]
    UPLOAD_DAT --> PARSE_DAT[解析DAT格式]
    PARSE_DAT --> CONVERT_CHARSET[轉換字元編碼]
    CONVERT_CHARSET --> VALIDATE_FORMAT{格式驗證}
    
    VALIDATE_FORMAT -->|通過| BULK_INSERT[大量插入DB]
    VALIDATE_FORMAT -->|失敗| FORMAT_ERROR[格式錯誤報告]
    
    BULK_INSERT --> AUTO_VALIDATE[自動驗證規則]
    AUTO_VALIDATE --> GENERATE_EXCEPTIONS[產生異常清單]
    GENERATE_EXCEPTIONS --> NOTIFY_ADMIN[通知管理員]
    
    ATT_DASHBOARD --> LOAD_REALTIME[載入即時資料]
    LOAD_REALTIME --> DEPT_SUMMARY[部門考勤摘要]
    LOAD_REALTIME --> EXCEPTION_COUNT[異常統計]
    LOAD_REALTIME --> TREND_CHART[趨勢圖表]
    
    EXCEPTION_HANDLE --> FILTER_EXCEPTION[篩選異常條件]
    FILTER_EXCEPTION --> SHOW_LIST[顯示異常清單]
    SHOW_LIST --> SELECT_ITEM[選擇異常項目]
    SELECT_ITEM --> MANUAL_FIX[人工修正]
    SELECT_ITEM --> AI_SUGGEST[AI建議]
    
    AI_COMPLETION --> SELECT_EMP[選擇員工]
    SELECT_EMP --> LOAD_HISTORY[載入歷史資料]
    LOAD_HISTORY --> ML_PREDICT[機器學習預測]
    ML_PREDICT --> SHOW_SUGGESTION[顯示建議時間]
    SHOW_SUGGESTION --> CONFIRM_AI{確認採用AI建議}
    
    CONFIRM_AI -->|確認| APPLY_SUGGESTION[套用建議]
    CONFIRM_AI -->|拒絕| MANUAL_INPUT[手動輸入]
    
    OT_CALC --> CALC_DAILY_OT[計算每日加班]
    CALC_DAILY_OT --> APPLY_RATES[套用加班費率]
    APPLY_RATES --> CHECK_LIMITS[檢查加班上限]
    CHECK_LIMITS --> ALERT_VIOLATION{是否超過上限}
    
    ALERT_VIOLATION -->|超過| SEND_ALERT[發送告警]
    ALERT_VIOLATION -->|正常| UPDATE_OT_SUM[更新加班彙總]
```

#### 📁 DAT檔案匯入詳細流程

```mermaid
sequenceDiagram
    participant User as 考勤管理員
    participant Web as Blazor前端
    participant API as 匯入API
    participant Parser as DAT解析器
    participant DB2 as DB2資料庫
    participant Validator as 驗證引擎
    participant Notify as 通知服務
    
    User->>Web: 選擇DAT檔案上傳
    Web->>API: POST /api/attendance/import
    API->>Parser: 解析DAT檔案
    
    Parser->>Parser: 檢查檔案格式(EBCDIC)
    Parser->>Parser: 轉換為UTF-8
    Parser->>Parser: 逐行解析考勤記錄
    
    alt 解析成功
        Parser-->>API: 回傳解析結果
        API->>DB2: 批次插入ATTEND_TXN
        
        loop 每100筆提交一次
            API->>DB2: COMMIT WORK
        end
        
        API->>Validator: 觸發驗證流程
        Validator->>DB2: 執行驗證規則
        Validator->>DB2: 插入異常記錄
        
        Validator-->>API: 驗證完成統計
        API->>Notify: 發送完成通知
        API-->>Web: 匯入成功(含異常統計)
        
    else 解析失敗
        Parser-->>API: 錯誤:檔案格式不正確
        API-->>Web: 匯入失敗訊息
    end
    
    Web-->>User: 顯示匯入結果
```

#### 🔍 考勤異常檢核流程

```mermaid
graph TD
    VALIDATION_START([考勤驗證開始]) --> LOAD_RULES[載入驗證規則]
    LOAD_RULES --> GET_ATTENDANCE[取得考勤資料]
    
    GET_ATTENDANCE --> RULE_ENGINE{驗證規則引擎}
    
    RULE_ENGINE --> CHECK_MISSING[檢查缺卡]
    RULE_ENGINE --> CHECK_LATE[檢查遲到]
    RULE_ENGINE --> CHECK_EARLY[檢查早退]
    RULE_ENGINE --> CHECK_OVERTIME[檢查異常加班]
    RULE_ENGINE --> CHECK_DUPLICATE[檢查重複打卡]
    RULE_ENGINE --> CHECK_WEEKEND[檢查假日出勤]
    
    CHECK_MISSING --> MISSING_FOUND{發現缺卡}
    MISSING_FOUND -->|是| CREATE_MISSING_EXC[建立缺卡異常]
    MISSING_FOUND -->|否| NEXT_CHECK1[下一項檢查]
    
    CHECK_LATE --> LATE_FOUND{發現遲到}
    LATE_FOUND -->|是| CALC_LATE_TIME[計算遲到時間]
    LATE_FOUND -->|否| NEXT_CHECK2[下一項檢查]
    
    CALC_LATE_TIME --> LATE_THRESHOLD{超過容許時間}
    LATE_THRESHOLD -->|是| CREATE_LATE_EXC[建立遲到異常]
    LATE_THRESHOLD -->|否| NEXT_CHECK2
    
    CHECK_OVERTIME --> OT_FOUND{發現加班}
    OT_FOUND -->|是| CHECK_APPROVAL{檢查加班核准}
    OT_FOUND -->|否| NEXT_CHECK3[下一項檢查]
    
    CHECK_APPROVAL -->|未核准| CREATE_OT_EXC[建立加班異常]
    CHECK_APPROVAL -->|已核准| CALC_OT_PAY[計算加班費]
    
    CHECK_DUPLICATE --> DUP_FOUND{發現重複}
    DUP_FOUND -->|是| AUTO_MERGE{自動合併}
    DUP_FOUND -->|否| NEXT_CHECK4[下一項檢查]
    
    AUTO_MERGE -->|成功| NEXT_CHECK4
    AUTO_MERGE -->|失敗| CREATE_DUP_EXC[建立重複異常]
    
    CREATE_MISSING_EXC --> EXCEPTION_DB[寫入異常表]
    CREATE_LATE_EXC --> EXCEPTION_DB
    CREATE_OT_EXC --> EXCEPTION_DB
    CREATE_DUP_EXC --> EXCEPTION_DB
    
    EXCEPTION_DB --> EMAIL_ALERT[發送郵件告警]
    EMAIL_ALERT --> DASHBOARD_UPDATE[更新儀表板]
```

#### 🤖 AI智能補登流程

```mermaid
flowchart TD
    AI_START([AI智能補登]) --> SELECT_TARGET[選擇目標員工]
    SELECT_TARGET --> DATE_RANGE[指定日期範圍]
    DATE_RANGE --> LOAD_PROFILE[載入員工檔案]
    
    LOAD_PROFILE --> GET_HISTORY[取得歷史考勤]
    GET_HISTORY --> FEATURE_EXTRACT[特徵提取]
    
    FEATURE_EXTRACT --> BASIC_FEATURES[基本特徵]
    FEATURE_EXTRACT --> TIME_FEATURES[時間特徵]
    FEATURE_EXTRACT --> PATTERN_FEATURES[模式特徵]
    
    BASIC_FEATURES --> DEPT_ID[部門代碼]
    BASIC_FEATURES --> JOB_LEVEL[職級]
    BASIC_FEATURES --> WORK_TYPE[工作型態]
    
    TIME_FEATURES --> DAY_OF_WEEK[星期]
    TIME_FEATURES --> MONTH_SEASON[月份季節]
    TIME_FEATURES --> HOLIDAY_FLAG[假日標記]
    
    PATTERN_FEATURES --> AVG_IN_TIME[平均上班時間]
    PATTERN_FEATURES --> AVG_OUT_TIME[平均下班時間]
    PATTERN_FEATURES --> VARIANCE[時間變異度]
    
    DEPT_ID --> ML_MODEL[機器學習模型]
    JOB_LEVEL --> ML_MODEL
    WORK_TYPE --> ML_MODEL
    DAY_OF_WEEK --> ML_MODEL
    MONTH_SEASON --> ML_MODEL
    HOLIDAY_FLAG --> ML_MODEL
    AVG_IN_TIME --> ML_MODEL
    AVG_OUT_TIME --> ML_MODEL
    VARIANCE --> ML_MODEL
    
    ML_MODEL --> PREDICT_IN[預測上班時間]
    ML_MODEL --> PREDICT_OUT[預測下班時間]
    ML_MODEL --> CONFIDENCE[計算信心度]
    
    PREDICT_IN --> SUGGESTION_UI[顯示建議介面]
    PREDICT_OUT --> SUGGESTION_UI
    CONFIDENCE --> SUGGESTION_UI
    
    SUGGESTION_UI --> USER_REVIEW{使用者檢視}
    USER_REVIEW -->|接受| APPLY_AI[套用AI建議]
    USER_REVIEW -->|修改| MANUAL_ADJUST[手動調整]
    USER_REVIEW -->|拒絕| DISCARD[放棄建議]
    
    APPLY_AI --> UPDATE_ATTEND[更新考勤記錄]
    MANUAL_ADJUST --> UPDATE_ATTEND
    UPDATE_ATTEND --> RETRAIN_MODEL[回饋訓練模型]
```

#### 📊 考勤儀表板即時更新流程

```mermaid
sequenceDiagram
    participant Dashboard as 考勤儀表板
    participant SignalR as SignalR Hub
    participant Service as 考勤服務
    participant DB2 as DB2資料庫
    participant Scheduler as 排程器
    
    Dashboard->>SignalR: 連接Hub
    SignalR-->>Dashboard: 連接確認
    
    loop 每5分鐘更新
        Scheduler->>Service: 觸發資料更新
        Service->>DB2: 查詢最新考勤統計
        DB2-->>Service: 回傳統計資料
        
        Service->>Service: 計算KPI指標
        Service->>SignalR: 推送更新資料
        SignalR->>Dashboard: 即時更新圖表
    end
    
    alt 有新異常產生
        Service->>SignalR: 推送異常告警
        SignalR->>Dashboard: 顯示紅色提醒
        Dashboard->>Dashboard: 播放提示音
    end
    
    Note over Dashboard: 使用者可看到：<br/>1.即時出勤人數<br/>2.異常統計<br/>3.部門排行<br/>4.趨勢圖表
```

#### 📊 資料流與整合點

| 資料表 | 主要欄位 | 整合關係 | 備註 |
|--------|----------|----------|------|
| **ATTEND_TXN** | EMP_ID, DATE, IN_TIME, OUT_TIME | ← M01員工主檔<br/>→ M04薪資計算 | 考勤交易表，按月分割 |
| **ATTEND_EXCEPTION** | EMP_ID, DATE, EXCEPTION_TYPE | → M05審核流程 | 異常記錄表 |
| **OT_SUMMARY** | EMP_ID, DATE, OT_HOURS, OT_PAY | → M04薪資計算 | 加班彙總表 |
| **ATTEND_RULES** | RULE_ID, RULE_TYPE, PARAMETERS | ← M03參數設定 | 驗證規則設定 |

---

### M03 - 薪資參數設定模組

#### 🎯 核心功能
- 薪資項目主檔維護
- 稅率表管理 (支援 XML 匯入)
- 社保費率設定
- 公式編輯器 (DSL 支援)
- 參數版本控制

#### 📊 參數設定主流程圖

```mermaid
flowchart TD
    START([M03薪資參數設定]) --> AUTH_CHECK{權限驗證}
    AUTH_CHECK -->|參數管理員| PARAM_MENU[參數設定選單]
    AUTH_CHECK -->|無權限| ACCESS_DENY[存取拒絕]
    
    PARAM_MENU --> PAY_ITEM[薪資項目設定]
    PARAM_MENU --> TAX_BRACKET[稅率表管理]
    PARAM_MENU --> INSURANCE[社保費率設定]
    PARAM_MENU --> FORMULA_EDITOR[公式編輯器]
    PARAM_MENU --> VERSION_CONTROL[參數版本控制]
    PARAM_MENU --> SIMULATION[薪資試算]
    
    PAY_ITEM --> ITEM_TYPE[選擇項目類型]
    ITEM_TYPE --> ALLOWANCE[津貼補助]
    ITEM_TYPE --> DEDUCTION[扣款項目]
    ITEM_TYPE --> NON_TAX[免稅項目]
    
    ALLOWANCE --> ITEM_CRUD[項目CRUD操作]
    DEDUCTION --> ITEM_CRUD
    NON_TAX --> ITEM_CRUD
    
    ITEM_CRUD --> SET_PROPERTIES[設定項目屬性]
    SET_PROPERTIES --> TAX_FLAG[是否計稅]
    SET_PROPERTIES --> INS_FLAG[是否計保]
    SET_PROPERTIES --> CALC_METHOD[計算方式]
    
    TAX_BRACKET --> UPLOAD_XML[上傳稅率XML]
    TAX_BRACKET --> MANUAL_INPUT[手動輸入]
    
    UPLOAD_XML --> PARSE_XML[解析XML格式]
    PARSE_XML --> VALIDATE_SCHEMA{XSD驗證}
    VALIDATE_SCHEMA -->|通過| PREVIEW_TAX[預覽稅率表]
    VALIDATE_SCHEMA -->|失敗| XML_ERROR[XML格式錯誤]
    
    PREVIEW_TAX --> CONFIRM_IMPORT{確認匯入}
    CONFIRM_IMPORT -->|確認| IMPORT_TAX[匯入稅率表]
    CONFIRM_IMPORT -->|取消| CANCEL_IMPORT[取消匯入]
    
    INSURANCE --> INS_TYPE[保險類型選擇]
    INS_TYPE --> LABOR_INS[勞保]
    INS_TYPE --> HEALTH_INS[健保]
    INS_TYPE --> PENSION[勞退]
    
    LABOR_INS --> SET_RATE[設定費率]
    HEALTH_INS --> SET_RATE
    PENSION --> SET_RATE
    
    SET_RATE --> EFFECTIVE_DATE[設定生效日期]
    EFFECTIVE_DATE --> VALIDATE_OVERLAP{檢查重疊}
    VALIDATE_OVERLAP -->|無重疊| SAVE_PARAM[儲存參數]
    VALIDATE_OVERLAP -->|有重疊| OVERLAP_ERROR[日期重疊錯誤]
    
    FORMULA_EDITOR --> SELECT_TEMPLATE[選擇公式範本]
    SELECT_TEMPLATE --> CUSTOM_FORMULA[自訂公式]
    CUSTOM_FORMULA --> DSL_INPUT[DSL語法輸入]
    DSL_INPUT --> SYNTAX_CHECK{語法檢查}
    
    SYNTAX_CHECK -->|正確| TEST_FORMULA[測試公式]
    SYNTAX_CHECK -->|錯誤| SYNTAX_ERROR[語法錯誤]
    
    TEST_FORMULA --> MOCK_DATA[模擬資料測試]
    MOCK_DATA --> FORMULA_RESULT[顯示計算結果]
    FORMULA_RESULT --> SAVE_FORMULA{儲存公式}
    
    VERSION_CONTROL --> VIEW_HISTORY[檢視版本歷史]
    VIEW_HISTORY --> SELECT_VERSION[選擇版本]
    SELECT_VERSION --> COMPARE_VERSION[版本比較]
    SELECT_VERSION --> ROLLBACK[版本回滾]
```

#### 📋 稅率表管理詳細流程

```mermaid
sequenceDiagram
    participant User as 稅務管理員
    participant Web as Blazor前端
    participant API as 參數API
    participant Parser as XML解析器
    participant Validator as 稅率驗證器
    participant DB2 as DB2資料庫
    participant Audit as 稽核服務
    
    User->>Web: 上傳稅率XML檔案
    Web->>API: POST /api/tax/import
    API->>Parser: 解析XML內容
    
    Parser->>Parser: 檢查XML格式
    Parser->>Parser: XSD Schema驗證
    
    alt XML格式正確
        Parser-->>API: 解析成功，回傳稅率資料
        API->>Validator: 驗證稅率級距
        
        Validator->>Validator: 檢查級距連續性
        Validator->>Validator: 檢查費率合理性
        Validator->>Validator: 檢查重複年度
        
        alt 驗證通過
            Validator-->>API: 驗證成功
            API-->>Web: 顯示稅率預覽表
            Web-->>User: 確認匯入？
            
            User->>Web: 確認匯入
            Web->>API: POST /api/tax/confirm
            
            API->>DB2: 備份舊版本稅率表
            API->>DB2: 插入新版本稅率
            API->>Audit: 記錄參數變更日誌
            
            API-->>Web: 匯入成功
            Web-->>User: 顯示成功訊息
            
        else 驗證失敗
            Validator-->>API: 驗證錯誤清單
            API-->>Web: 顯示驗證錯誤
            Web-->>User: 修正後重新上傳
        end
        
    else XML格式錯誤
        Parser-->>API: XML解析失敗
        API-->>Web: 格式錯誤訊息
        Web-->>User: 檢查XML格式
    end
```

#### 🧮 公式編輯器流程

```mermaid
graph TD
    FORMULA_START([公式編輯器]) --> TEMPLATE_SELECT[選擇公式範本]
    
    TEMPLATE_SELECT --> BASIC_PAY[基本薪資公式]
    TEMPLATE_SELECT --> OT_PAY[加班費公式]
    TEMPLATE_SELECT --> DEDUCTION[扣款公式]
    TEMPLATE_SELECT --> CUSTOM[自訂公式]
    
    BASIC_PAY --> LOAD_TEMPLATE[載入範本]
    OT_PAY --> LOAD_TEMPLATE
    DEDUCTION --> LOAD_TEMPLATE
    CUSTOM --> BLANK_EDITOR[空白編輯器]
    
    LOAD_TEMPLATE --> EDIT_FORMULA[編輯公式]
    BLANK_EDITOR --> EDIT_FORMULA
    
    EDIT_FORMULA --> DSL_SYNTAX[DSL語法輸入]
    DSL_SYNTAX --> AVAILABLE_VARS[可用變數清單]
    DSL_SYNTAX --> FUNCTION_LIST[函式清單]
    
    AVAILABLE_VARS --> BASIC_SALARY[基本薪資]
    AVAILABLE_VARS --> OT_HOURS[加班時數]
    AVAILABLE_VARS --> ALLOWANCE[各項津貼]
    AVAILABLE_VARS --> TAX_RATE[稅率]
    
    FUNCTION_LIST --> MATH_FUNC[數學函式]
    FUNCTION_LIST --> DATE_FUNC[日期函式]
    FUNCTION_LIST --> LOGIC_FUNC[邏輯函式]
    
    MATH_FUNC --> ROUND_FUNC[ROUND四捨五入]
    MATH_FUNC --> MAX_MIN[MAX/MIN]
    
    DSL_SYNTAX --> REAL_TIME_CHECK[即時語法檢查]
    REAL_TIME_CHECK --> SYNTAX_OK{語法正確?}
    
    SYNTAX_OK -->|正確| HIGHLIGHT_OK[綠色標記]
    SYNTAX_OK -->|錯誤| HIGHLIGHT_ERROR[紅色標記]
    
    HIGHLIGHT_ERROR --> SHOW_ERROR_MSG[顯示錯誤訊息]
    SHOW_ERROR_MSG --> FIX_SYNTAX[修正語法]
    FIX_SYNTAX --> DSL_SYNTAX
    
    HIGHLIGHT_OK --> TEST_BUTTON[測試公式按鈕]
    TEST_BUTTON --> INPUT_TEST_DATA[輸入測試資料]
    INPUT_TEST_DATA --> EXECUTE_FORMULA[執行公式計算]
    EXECUTE_FORMULA --> SHOW_RESULT[顯示計算結果]
    
    SHOW_RESULT --> RESULT_OK{結果正確?}
    RESULT_OK -->|正確| SAVE_FORMULA[儲存公式]
    RESULT_OK -->|錯誤| EDIT_FORMULA
    
    SAVE_FORMULA --> VERSION_COMMENT[版本說明]
    VERSION_COMMENT --> COMMIT_FORMULA[提交公式]
```

#### 📊 薪資試算功能流程

```mermaid
flowchart TD
    SIM_START([薪資試算]) --> SELECT_EMP[選擇試算員工]
    SELECT_EMP --> SELECT_PERIOD[選擇試算期間]
    SELECT_PERIOD --> LOAD_BASIC[載入基本資料]
    
    LOAD_BASIC --> EMP_INFO[員工基本資訊]
    LOAD_BASIC --> ATTEND_DATA[考勤資料]
    LOAD_BASIC --> PARAM_SET[參數設定]
    
    EMP_INFO --> BASE_SALARY[基本薪資]
    EMP_INFO --> JOB_GRADE[職級]
    EMP_INFO --> DEPT_CODE[部門代碼]
    
    ATTEND_DATA --> WORK_DAYS[出勤天數]
    ATTEND_DATA --> OT_HOURS[加班時數]
    ATTEND_DATA --> LEAVE_DAYS[請假天數]
    
    PARAM_SET --> TAX_TABLE[稅率表]
    PARAM_SET --> INS_RATE[保險費率]
    PARAM_SET --> ALLOWANCE_RULE[津貼規則]
    
    BASE_SALARY --> CALC_GROSS[計算應發薪資]
    WORK_DAYS --> CALC_GROSS
    OT_HOURS --> CALC_GROSS
    ALLOWANCE_RULE --> CALC_GROSS
    
    CALC_GROSS --> GROSS_TOTAL[應發總額]
    
    GROSS_TOTAL --> CALC_TAX[計算所得稅]
    TAX_TABLE --> CALC_TAX
    CALC_TAX --> TAX_AMOUNT[所得稅額]
    
    GROSS_TOTAL --> CALC_INS[計算保險費]
    INS_RATE --> CALC_INS
    CALC_INS --> LABOR_INS[勞保費]
    CALC_INS --> HEALTH_INS[健保費]
    CALC_INS --> PENSION_CONT[勞退提撥]
    
    TAX_AMOUNT --> CALC_NET[計算實發薪資]
    LABOR_INS --> CALC_NET
    HEALTH_INS --> CALC_NET
    PENSION_CONT --> CALC_NET
    
    CALC_NET --> NET_SALARY[實發薪資]
    NET_SALARY --> SHOW_DETAIL[顯示明細表]
    
    SHOW_DETAIL --> BREAKDOWN[薪資明細]
    BREAKDOWN --> GROSS_ITEMS[應發項目]
    BREAKDOWN --> DEDUCT_ITEMS[扣款項目]
    BREAKDOWN --> NET_AMOUNT[實發金額]
    
    SHOW_DETAIL --> COMPARE_LAST{與上期比較}
    COMPARE_LAST -->|是| VARIANCE_ANALYSIS[差異分析]
    COMPARE_LAST -->|否| EXPORT_RESULT[匯出結果]
    
    VARIANCE_ANALYSIS --> DIFF_REPORT[差異報告]
    DIFF_REPORT --> EXPORT_RESULT
```

#### 📊 資料流與整合點

| 資料表 | 主要欄位 | 整合關係 | 備註 |
|--------|----------|----------|------|
| **PAYROLL_ITEM** | ITEM_ID, TYPE, TAX_FLAG, FORMULA_DSL | → M04薪資計算<br/>→ M06報表輸出 | 薪資項目主檔 |
| **PAY_PARAM_HIST** | PARAM_ID, VALUE, EFF_DATE, EXP_DATE | → M04薪資計算 | 參數歷史表(Temporal) |
| **TAX_BRACKET** | YEAR, MIN_AMT, MAX_AMT, RATE | → M04薪資計算 | 累進稅率表 |
| **FORMULA_VERSION** | FORMULA_ID, VERSION, DSL_CODE | → M04薪資計算 | 公式版本控制 |

---

### M04 - 薪資計算批次模組

#### 🎯 核心功能
- 高效能批次薪資計算
- Checkpoint 重跑機制
- 並行處理支援
- 即時進度監控
- 差異分析與比較

#### 📊 薪資批次計算主流程圖

```mermaid
flowchart TD
    START([M04薪資計算批次]) --> AUTH_CHECK{權限驗證}
    AUTH_CHECK -->|薪資管理員| BATCH_MENU[批次作業選單]
    AUTH_CHECK -->|無權限| ACCESS_DENY[存取拒絕]
    
    BATCH_MENU --> START_BATCH[啟動新批次]
    BATCH_MENU --> MONITOR_BATCH[監控現有批次]
    BATCH_MENU --> RESTART_BATCH[重跑失敗批次]
    BATCH_MENU --> COMPARE_BATCH[批次差異比較]
    BATCH_MENU --> ARCHIVE_BATCH[批次歷史查詢]
    
    START_BATCH --> SELECT_PERIOD[選擇計算期間]
    SELECT_PERIOD --> PRE_CHECK[前置檢查]
    
    PRE_CHECK --> CHECK_DATA{資料完整性檢查}
    CHECK_DATA --> CHECK_EMP[員工主檔]
    CHECK_DATA --> CHECK_ATTEND[考勤資料]
    CHECK_DATA --> CHECK_PARAM[薪資參數]
    
    CHECK_EMP --> EMP_READY{員工資料就緒}
    CHECK_ATTEND --> ATT_READY{考勤資料就緒}
    CHECK_PARAM --> PARAM_READY{參數資料就緒}
    
    EMP_READY -->|否| DATA_MISSING[資料缺失告警]
    ATT_READY -->|否| DATA_MISSING
    PARAM_READY -->|否| DATA_MISSING
    
    EMP_READY -->|是| ALL_READY{全部資料就緒}
    ATT_READY -->|是| ALL_READY
    PARAM_READY -->|是| ALL_READY
    
    ALL_READY -->|是| LOCK_PERIOD[鎖定計算期間]
    ALL_READY -->|否| FIX_DATA[修正資料]
    
    LOCK_PERIOD --> CREATE_BATCH[建立批次作業]
    CREATE_BATCH --> LOAD_EMPLOYEES[載入員工清單]
    LOAD_EMPLOYEES --> SPLIT_CHUNKS[分割處理批次]
    
    SPLIT_CHUNKS --> PARALLEL_CALC[並行計算處理]
    PARALLEL_CALC --> CALC_GROSS[計算應發薪資]
    PARALLEL_CALC --> CALC_DEDUCT[計算各項扣款]
    PARALLEL_CALC --> CALC_NET[計算實發薪資]
    
    CALC_GROSS --> GROSS_RESULT[應發結果]
    CALC_DEDUCT --> DEDUCT_RESULT[扣款結果]
    CALC_NET --> NET_RESULT[實發結果]
    
    GROSS_RESULT --> CHECKPOINT{Checkpoint檢查點}
    DEDUCT_RESULT --> CHECKPOINT
    NET_RESULT --> CHECKPOINT
    
    CHECKPOINT -->|每500筆| SAVE_PROGRESS[儲存進度]
    CHECKPOINT -->|繼續| MORE_EMP{還有員工待處理}
    
    MORE_EMP -->|是| PARALLEL_CALC
    MORE_EMP -->|否| BATCH_COMPLETE[批次計算完成]
    
    SAVE_PROGRESS --> REALTIME_UPDATE[即時進度更新]
    REALTIME_UPDATE --> MORE_EMP
    
    BATCH_COMPLETE --> GENERATE_SUMMARY[產生批次摘要]
    GENERATE_SUMMARY --> VARIANCE_CHECK[差異檢查]
    VARIANCE_CHECK --> BATCH_READY[批次就緒審核]
    
    DATA_MISSING --> NOTIFY_ADMIN[通知系統管理員]
    FIX_DATA --> PRE_CHECK
```

#### 🔄 批次重跑機制流程

```mermaid
sequenceDiagram
    participant Admin as 系統管理員
    participant Web as 監控介面
    participant Engine as 批次引擎
    participant DB2 as DB2資料庫
    participant RPG as RPG程式
    participant Monitor as 進度監控
    
    Admin->>Web: 選擇失敗批次重跑
    Web->>Engine: 檢查批次狀態
    Engine->>DB2: 查詢最後Checkpoint
    DB2-->>Engine: 回傳Checkpoint資訊
    
    Engine->>Engine: 驗證資料完整性
    alt 資料完整
        Engine->>DB2: 回滾至Checkpoint
        Engine->>RPG: 重新啟動批次Job
        RPG-->>Engine: Job啟動成功
        
        loop 批次處理
            RPG->>DB2: 處理員工薪資
            RPG->>Monitor: 更新進度
            Monitor->>Web: 即時進度推送
            Web->>Admin: 顯示進度條
        end
        
        RPG->>DB2: 批次完成
        RPG-->>Engine: 完成通知
        Engine-->>Web: 重跑成功
        
    else 資料不完整
        Engine-->>Web: 錯誤:資料不完整
        Web-->>Admin: 需要修復資料
    end
```

### M05-M10 模組流程概要

#### M05 - 薪資審核與核准模組
**核心流程：** 部門審核 → HR複核 → 財務核准 → 銀行匯款

#### M06 - 報表與匯出模組  
**核心流程：** 薪資單PDF → 銀行匯款檔 → 稅務申報 → ERP分錄

#### M07 - 檔案交換中心模組
**核心流程：** SFTP傳輸 → PGP加密 → 完整性驗證 → 自動重傳

#### M08 - 安全與權限模組
**核心流程：** JWT認證 → 角色授權 → 欄位加密 → 稽核記錄

#### M09 - 稽核與日誌模組
**核心流程：** 操作記錄 → 差異追蹤 → 合規報表 → 異常告警

#### M10 - 系統設定與監控模組
**核心流程：** 參數管理 → 排程控制 → 健康監控 → 告警通知

---

## 🔗 AS400 DB2 整合架構

### 整合策略概要

```mermaid
graph TB
    subgraph ".NET 前端系統"
        A[Blazor Server App]
        B[Web API Gateway]
        C[Entity Framework Core]
        D[DB2 Data Provider]
    end
    
    subgraph "整合層"
        E[連線池管理]
        F[資料同步服務]
        G[RPG程式調用]
        H[檔案交換服務]
    end
    
    subgraph "AS400 IBM i"
        I[DB2/400 資料庫]
        J[RPG/CLP 批次程式]
        K[IFS 檔案系統]
        L[SFTP 服務]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    E --> I
    
    B --> F
    F --> I
    
    B --> G
    G --> J
    
    B --> H
    H --> K
    H --> L
```

### 關鍵整合點

| 整合類型 | 技術方案 | 資料流向 | 備註 |
|----------|----------|----------|------|
| **資料查詢** | Entity Framework + DB2 Provider | .NET ← DB2/400 | 員工、考勤、薪資資料讀取 |
| **批次計算** | XMLSERVICE / REST API | .NET → RPG → DB2 | 薪資計算引擎調用 |
| **檔案交換** | SFTP / IFS 直接存取 | .NET ↔ IFS | DAT匯入、PDF匯出 |
| **即時同步** | 定時排程 + 異動觸發 | .NET ↔ DB2/400 | 雙向資料同步 |

---

## 📊 系統資料流程

### 月度薪資處理完整資料流

```mermaid
flowchart LR
    subgraph "資料輸入"
        A1[員工異動CSV]
        A2[考勤DAT檔]
        A3[加班申請單]
        A4[請假申請單]
        A5[薪資參數XML]
    end
    
    subgraph "資料驗證"
        B1[格式檢查]
        B2[業務邏輯驗證]
        B3[重複性檢查]
        B4[完整性驗證]
    end
    
    subgraph "資料處理"
        C1[員工主檔更新]
        C2[考勤異常處理]
        C3[加班時數統計]
        C4[參數版本控制]
    end
    
    subgraph "計算引擎"
        D1[應發薪資計算]
        D2[各項扣款計算]
        D3[實發薪資計算]
        D4[稅額計算]
    end
    
    subgraph "審核流程"
        E1[部門主管審核]
        E2[HR總務審核]
        E3[財務主管審核]
    end
    
    subgraph "輸出產生"
        F1[個人薪資單PDF]
        F2[銀行匯款檔案]
        F3[政府申報檔案]
        F4[ERP分錄檔案]
        F5[管理報表]
    end
    
    A1 --> B1
    A2 --> B1
    A3 --> B2
    A4 --> B2
    A5 --> B3
    
    B1 --> C1
    B2 --> C2
    B3 --> C3
    B4 --> C4
    
    C1 --> D1
    C2 --> D1
    C3 --> D2
    C4 --> D3
    
    D1 --> E1
    D2 --> E1
    D3 --> E2
    D4 --> E3
    
    E1 --> F1
    E2 --> F2
    E3 --> F3
    E3 --> F4
    E3 --> F5
```

---

## 👤 用戶操作流程

### 主要角色操作流程圖

```mermaid
graph TD
    subgraph "HR人事專員操作流程"
        HR1[登入系統] --> HR2[員工資料維護]
        HR2 --> HR3[考勤資料匯入]
        HR3 --> HR4[異常考勤處理]
        HR4 --> HR5[薪資參數設定]
        HR5 --> HR6[啟動薪資批次]
    end
    
    subgraph "部門主管操作流程"
        DEPT1[登入系統] --> DEPT2[查看部門薪資]
        DEPT2 --> DEPT3[審核薪資結果]
        DEPT3 --> DEPT4{審核決定}
        DEPT4 -->|通過| DEPT5[確認審核]
        DEPT4 -->|退回| DEPT6[填寫退回原因]
    end
    
    subgraph "財務人員操作流程"
        FIN1[登入系統] --> FIN2[最終薪資審核]
        FIN2 --> FIN3[產生銀行匯款檔]
        FIN3 --> FIN4[上傳銀行SFTP]
        FIN4 --> FIN5[接收銀行回饋]
        FIN5 --> FIN6[完成薪資發放]
    end
    
    subgraph "一般員工操作流程"
        EMP1[登入ESS系統] --> EMP2[查詢個人薪資]
        EMP2 --> EMP3[下載薪資單PDF]
        EMP3 --> EMP4[查看考勤記錄]
        EMP4 --> EMP5[申請考勤異常]
    end
    
    HR6 --> DEPT2
    DEPT5 --> FIN2
    FIN6 --> EMP2
```

---

## 📋 所需材料與資源清單

### 🏢 硬體設備需求

| 設備類型 | 規格要求 | 數量 | 用途 | 預估費用 |
|----------|----------|------|------|----------|
| **應用伺服器** | Windows Server 2022<br/>CPU: 8 Core<br/>RAM: 32GB<br/>Storage: 500GB SSD | 2台 | .NET應用程式運行 | NT$ 200,000 |
| **Web伺服器** | Windows Server 2022<br/>CPU: 4 Core<br/>RAM: 16GB<br/>Storage: 200GB SSD | 2台 | IIS + Reverse Proxy | NT$ 120,000 |
| **快取伺服器** | Linux Ubuntu 22.04<br/>CPU: 4 Core<br/>RAM: 16GB<br/>Storage: 100GB SSD | 1台 | Redis 快取服務 | NT$ 60,000 |
| **負載平衡器** | F5 或 HAProxy 硬體設備 | 1台 | 流量分散與高可用 | NT$ 150,000 |

### 💿 軟體授權需求

| 軟體名稱 | 版本 | 授權類型 | 數量 | 預估費用 |
|----------|------|----------|------|----------|
| **Windows Server** | 2022 Standard | 伺服器授權 | 4套 | NT$ 80,000 |
| **SQL Server** | 2022 Standard | 開發測試用 | 1套 | NT$ 50,000 |
| **Visual Studio** | 2022 Enterprise | 開發授權 | 5套 | NT$ 150,000 |
| **IBM DB2 Connect** | v11.5 | 連線授權 | 100 User | NT$ 200,000 |
| **Office 365** | E3 計畫 | 使用者授權 | 10 User | NT$ 30,000 |

### 🛠️ 開發工具與套件

| 工具類別 | 具體項目 | 版本需求 | 授權類型 |
|----------|----------|----------|----------|
| **IDE開發環境** | Visual Studio 2022 Enterprise | v17.8+ | 商業授權 |
| **版本控制** | Git + Azure DevOps Services | 最新版 | 免費/付費 |
| **API測試** | Postman Team | 最新版 | 團隊授權 |
| **資料庫工具** | IBM Data Studio | v4.1+ | 免費 |
| **文檔工具** | Confluence + Jira | 最新版 | Atlassian授權 |
| **監控工具** | Application Insights | 最新版 | Azure付費 |

### 📊 第三方服務需求

| 服務類型 | 服務提供商 | 規格需求 | 月費用 |
|----------|------------|----------|--------|
| **雲端主機** | Azure/AWS | 4 vCPU, 16GB RAM | NT$ 20,000 |
| **CDN服務** | Azure CDN | 500GB流量 | NT$ 5,000 |
| **郵件服務** | SendGrid | 10,000封/月 | NT$ 3,000 |
| **簡訊服務** | Twilio | 1,000則/月 | NT$ 2,000 |
| **SSL憑證** | DigiCert | Wildcard SSL | NT$ 15,000/年 |

### 🔐 安全與憑證需求

| 安全項目 | 規格要求 | 說明 | 費用 |
|----------|----------|------|------|
| **SSL憑證** | EV SSL Certificate | 年興紡織網域專用 | NT$ 25,000/年 |
| **代碼簽章憑證** | Code Signing Certificate | .NET執行檔簽章 | NT$ 15,000/年 |
| **PGP金鑰對** | 2048-bit RSA | 銀行檔案加密用 | 自行產生 |
| **HSM設備** | SafeNet Luna | 金鑰管理硬體 | NT$ 300,000 |

### 📚 AS400 相關需求

| 項目類型 | 具體需求 | 數量/規格 | 說明 |
|----------|----------|-----------|------|
| **DB2連線授權** | IBM DB2 Connect | 100 User License | 連接AS400資料庫 |
| **XMLSERVICE** | 開源套件 | 最新版本 | RPG程式調用介面 |
| **5250模擬器** | IBM i Access Client | 5套授權 | 終端機連線工具 |
| **FTP/SFTP設定** | AS400內建服務 | 配置服務 | 檔案傳輸通道 |

### 📋 系統整合材料

| 整合項目 | 所需資料/文件 | 提供單位 | 重要程度 |
|----------|---------------|----------|----------|
| **資料庫Schema** | 完整的DB2資料表結構 | AS400管理員 | 🔴 必要 |
| **現有RPG程式** | 薪資計算RPG原始碼 | 系統廠商 | 🔴 必要 |
| **業務流程文件** | 現行薪資作業手冊 | HR部門 | 🔴 必要 |
| **權限矩陣** | 使用者角色權限清單 | IT部門 | 🟡 重要 |
| **稅務規則** | 最新稅法計算公式 | 財務部門 | 🔴 必要 |
| **銀行介面規格** | 匯款檔案格式說明 | 往來銀行 | 🔴 必要 |

### 📄 法規合規材料

| 法規類型 | 所需文件 | 取得方式 | 更新頻率 |
|----------|----------|----------|----------|
| **個資法規範** | 個資保護實施細則 | 法務部門 | 年度更新 |
| **勞基法規定** | 工時與薪資規範 | 勞動部網站 | 季度確認 |
| **稅務法規** | 所得稅扣繳規定 | 財政部網站 | 年度更新 |
| **金融法規** | 銀行資訊安全規範 | 金管會公告 | 年度更新 |

### 🧪 測試環境需求

| 環境類型 | 硬體需求 | 軟體需求 | 用途 |
|----------|----------|----------|------|
| **開發環境** | 個人電腦 | Visual Studio + DB2 | 程式開發 |
| **測試環境** | 虛擬機器 2台 | 完整系統複製 | 功能測試 |
| **預生產環境** | 與正式環境相同 | 完整系統複製 | UAT測試 |
| **Demo環境** | 雲端主機 | 精簡版系統 | 展示用途 |

### 📞 外部協作需求

| 協作對象 | 合作內容 | 聯絡窗口 | 時程要求 |
|----------|----------|----------|----------|
| **AS400廠商** | 系統整合支援 | 技術主管 | 整個專案期間 |
| **銀行IT部門** | 匯款介面測試 | 資訊部經理 | 測試階段 |
| **會計師事務所** | 合規性審查 | 查帳會計師 | 上線前審查 |
| **資安廠商** | 滲透測試服務 | 資安顧問 | 上線前測試 |

### 💰 總預算估算

| 費用類別 | 金額 (新台幣) | 說明 |
|----------|---------------|------|
| **硬體設備** | NT$ 530,000 | 伺服器、網路設備 |
| **軟體授權** | NT$ 510,000 | 作業系統、開發工具、DB2 |
| **雲端服務** | NT$ 360,000 | 12個月雲端服務費用 |
| **安全憑證** | NT$ 340,000 | SSL、HSM、簽章憑證 |
| **外部服務** | NT$ 200,000 | 顧問、測試、訓練 |
| **備援與維護** | NT$ 160,000 | 備份設備、維護合約 |
| **總計** | **NT$ 2,100,000** | 不含人力成本 |

---

## ⚠️ 風險控制

### 主要風險評估與對策

| 風險項目 | 風險等級 | 潛在影響 | 防範對策 | 應變措施 |
|----------|----------|----------|----------|----------|
| **AS400 DB2 整合複雜度** | 🔴 高 | 開發延遲、資料不一致 | 提前 POC 驗證、專家諮詢 | 備用整合方案、外部支援 |
| **現有 RPG 程式相依性** | 🟡 中 | 批次功能異常 | XMLSERVICE 封裝、充分測試 | 漸進式遷移、雙軌並行 |
| **大量資料處理效能** | 🟡 中 | 使用者體驗差 | 分批處理、非同步機制 | 硬體升級、演算法優化 |
| **法規異動影響** | 🟡 中 | 合規風險 | 定期法規檢查、彈性設計 | 快速修正機制、緊急發佈 |
| **使用者接受度** | 🟢 低 | 系統使用率低 | 早期參與、充分培訓 | 功能調整、操作簡化 |

### 品質保證措施

#### **程式碼品質**
- **程式碼審查** - 所有程式碼必須經過 Peer Review
- **單元測試** - 測試覆蓋率需達 80% 以上
- **靜態分析** - 使用 SonarQube 進行程式碼品質檢查
- **效能測試** - 定期進行負載測試與壓力測試

#### **系統整合品質**
- **API 測試** - 完整的 API 自動化測試套件
- **端到端測試** - 模擬真實業務流程的整合測試
- **資料一致性** - 跨系統資料同步驗證機制
- **回歸測試** - 每次發佈前的完整功能驗證

---

## 🎯 成功標準

### 技術指標

| 指標項目 | 目標值 | 測量方法 | 驗收標準 |
|----------|--------|----------|----------|
| **系統回應時間** | ≤ 2 秒 | 前端頁面載入時間 | 90% 頁面符合標準 |
| **API 效能** | ≤ 300ms | Web API 回應時間 | 95% API 符合標準 |
| **批次處理效能** | 10k員工 ≤ 30分鐘 | 薪資計算批次時間 | 生產環境實測通過 |
| **系統可用性** | ≥ 99.5% | 月度監控統計 | 符合 SLA 要求 |
| **資料準確性** | 100% | 與 AS400 資料比對 | 零資料遺失或錯誤 |

### 業務指標

| 指標項目 | 目標值 | 測量方法 | 驗收標準 |
|----------|--------|----------|----------|
| **使用者滿意度** | ≥ 85% | UAT 問卷調查 | 使用者反饋良好 |
| **操作效率提升** | ≥ 30% | 作業時間比較 | 較現有系統更快 |
| **錯誤率降低** | ≤ 5% | 人工錯誤統計 | 顯著減少人工錯誤 |
| **培訓成效** | ≤ 2 小時 | 新使用者學習時間 | 快速上手使用 |

### 合規指標

| 指標項目 | 目標值 | 驗證方法 | 合規標準 |
|----------|--------|----------|----------|
| **資安滲透測試** | 無高風險漏洞 | 第三方安全評估 | 通過資安檢測 |
| **個資法合規** | 100% 合規 | 法務部門審核 | 符合個資保護法 |
| **稽核軌跡完整性** | 100% 記錄 | 稽核日誌檢查 | 所有操作可追蹤 |
| **備份恢復** | RPO ≤ 15分鐘 | 災難演練測試 | 符合 DR 要求 |

---

## 📞 結論與建議

### 專案總結

本計劃書提供了年興紡織人事薪資系統 **.NET 前端現代化** 的完整藍圖，涵蓋：

✅ **系統架構設計** - 完整的三層式架構與 AS400 整合策略  
✅ **業務流程分析** - 十大功能模組的詳細流程設計  
✅ **技術實作方案** - .NET 8 + Blazor Server 現代化技術棧  
✅ **資源需求規劃** - 硬體、軟體、人力的完整清單  
✅ **風險控制機制** - 技術與業務風險的全面評估  

### 關鍵成功因素

1. **AS400 整合穩定性** - DB2 連線與 RPG 程式調用的可靠性
2. **使用者體驗優化** - 現代化介面與直觀操作流程
3. **資料安全保護** - 符合法規要求的加密與稽核機制
4. **效能最佳化** - 大量資料處理的高效能解決方案
5. **團隊技能整合** - .NET 與 AS400 專業知識的結合

### 立即行動建議

#### **第一階段準備工作 (1個月內)**
1. **組建專案團隊** - 確保 .NET 與 AS400 專家到位
2. **環境準備** - 開發、測試環境建置
3. **需求確認** - 與業務單位詳細確認功能需求
4. **技術 POC** - 驗證 DB2 整合可行性

#### **風險緩解措施**
1. **技術風險** - 提前進行關鍵技術驗證
2. **時程風險** - 分階段交付，降低整體風險  
3. **品質風險** - 建立完善的測試與品保機制
4. **變更風險** - 建立靈活的需求變更管理流程

### 預期效益

#### **短期效益 (6個月內)**
- 現代化使用者介面提升操作體驗
- 自動化流程減少人工錯誤
- 即時監控與告警機制

#### **中期效益 (1年內)**  
- 完整取代舊有 LANSA 前端系統
- 提升薪資處理效率 30% 以上
- 強化資訊安全與合規性

#### **長期效益 (2年以上)**
- 為未來微服務化奠定基礎
- 支援更多業務擴展需求
- 降低系統維護成本

---

**📅 最後更新：** 2025-01-26  
**📧 聯絡資訊：** 如需進一步討論技術細節或專案執行，請聯繫開發團隊進行詳細評估。

**🎯 下一步行動：** 建議立即啟動技術可行性驗證 (POC) 階段，確認 AS400 DB2 整合方案可行性，為正式開發奠定穩固基礎。