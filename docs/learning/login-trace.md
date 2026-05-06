# go-admin 登入鏈路學習導讀

> 一條 `POST /api/v1/login` 從 client 進到 JWT 出來的完整追蹤
>
> 本文件由 SpecKit feature `001-learning-trace-login` 產出。
> Spec 與相關設計檔在 `specs/001-learning-trace-login/`。
>
> **語言**：zh-TW 散文 + 原始程式碼／註解保留原文。

## 目錄

1. [學習目標 & 前置條件](#1-學習目標--前置條件)
2. [用 Docker Compose 把後端跑起來](#2-用-docker-compose-把後端跑起來)
3. [用 curl 登入並拿到 JWT](#3-用-curl-登入並拿到-jwt)
4. [登入鏈路序列圖](#4-登入鏈路序列圖)
5. [逐層追蹤每一段 code](#5-逐層追蹤每一段-code)
6. [Runtime 驗證 recipe](#6-runtime-驗證-recipe)
7. [SDK 邊界](#7-sdk-邊界)
8. [下一步](#8-下一步)

---

## 1. 學習目標 & 前置條件

### 1.1 你讀完這份文件之後可以做到

- 用**一條** Docker 指令把 go-admin 後端跑起來，不需要本機裝 Go、不需要 MySQL、也不需要 `go-admin-ui` 前端。
- 用 `curl` 拿到 `admin/123456` 對應的 JWT，並理解 dev-mode captcha bypass 為什麼能通過。
- 講得出 `POST /api/v1/login` 從 router 進來、到 JWT 出去之間每一層的角色，並指得出對應的 `file:line`。
- 用 runtime 證據（GORM 的 SQL 日誌、JWT claims、`sys_login_log` 資料列）反向驗證自己讀程式碼讀出來的預測。

### 1.2 前置條件（核心只有一項：Docker）

- **Docker**：Docker Desktop（macOS）、Docker Desktop + WSL2（Windows）或 Docker Engine + `compose` plugin（Linux / WSL2）。本文件實測過 Linux 與 WSL2，未驗證純 Windows 路徑。
- 終端機（多數 OS 內建 `curl`，或自行安裝）。
- 數百 MB 磁碟空間放 `golang:1.24-alpine` builder image、`alpine` runtime image、以及 builder 的 module cache。

```text
required: docker (with compose plugin), curl
```

### 1.3 不需要的東西（特別澄清）

專案 README「Ready to work」一節列了 Go、MySQL、Node、`go-admin-ui` 等項目，看起來像必備。**對這份學習文件而言通通不需要**：

- 不需要在本機安裝 **Go toolchain**（多階段 Dockerfile 在 builder stage 內編譯）。
- 不需要 **MySQL**（compose 檔走 SQLite，DB 檔已內嵌進 image）。
- 不需要 **Node / `go-admin-ui` 前端**（直接用 `curl` 打 API）。

### 1.4 時間預算

- 第一次 `docker compose ... up --build`：5–10 分鐘，主要花在抓 base image 與 Go module（網速決定）。
- 之後每次重啟：~30 秒。第一次成功之後，從敲指令到拿到 JWT ≤ 5 分鐘（SC-005）。
- 整份 walkthrough（讀完 + 跑完 + 驗證）：≤ 30 分鐘（SC-001）。

### 1.5 不在本文範圍

Casbin / data-scope 過濾、寫入路徑的 operation log、code-gen 等主題都**不在這份文件**——它們是後續獨立 spec 的題目，第 8 節會列出指引。

---

## 2. 用 Docker Compose 把後端跑起來

> 本節將在 T007 由 `[US1]` 任務填寫。

---

## 3. 用 curl 登入並拿到 JWT

> 本節將在 T008 由 `[US1]` 任務填寫。

---

## 4. 登入鏈路序列圖

> 本節將在 T011 由 `[US2]` 任務填寫。

---

## 5. 逐層追蹤每一段 code

> 本節將在 T012 由 `[US2]` 任務填寫。

---

## 6. Runtime 驗證 recipe

> 本節將在 T016 由 `[US3]` 任務填寫。

---

## 7. SDK 邊界

> 本節將在 T013 由 `[US2]` 任務填寫。

---

## 8. 下一步

> 本節將在 T014 由 `[US2]` 任務填寫。
