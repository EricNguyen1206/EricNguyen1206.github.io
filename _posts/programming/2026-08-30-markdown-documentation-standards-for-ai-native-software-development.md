---
layout: post
title: "Nghiên cứu các chuẩn tổ chức tài liệu Markdown (.md) trong workflow phát triển phần mềm AI-Native"
date: 2026-08-30 09:00:00 +0700
categories: programming
---

Trong kỷ nguyên của các **Coding Agents** và **AI-Assisted IDEs** (như Cursor, Claude Code, Windsurf, Roo Code, Antigravity, Aider, Devin...), có một sự dịch chuyển âm thầm nhưng mang tính cách mạng: **Tài liệu Markdown (`.md`) không còn chỉ dành cho con người đọc sau khi code xong, mà trở thành lớp giao diện (Interface), bộ nhớ (Memory), và giao thức điều khiển (Control Protocol) chính giữa Kỹ sư và AI Agent.**

Nếu bạn ném cho AI một mớ tài liệu lộn xộn, dài dòng hoặc thiếu cấu trúc, agent sẽ nhanh chóng rơi vào cái bẫy **"Lost in the Middle"** (mất tập trung trong context window lớn), sinh ra ảo giác (hallucination), vi phạm convention hoặc tốn hàng triệu token vô ích.

Bài viết này là tổng hợp nghiên cứu chi tiết về **các chuẩn và mô hình tổ chức tài liệu `.md` tốt nhất hiện nay** để tối ưu hóa context, tăng độ chính xác của AI và duy trì codebase luôn sạch sẽ.

---

## 1. Bản đồ 5 trụ cột tài liệu Markdown trong AI-Native Workflow

Một hệ thống tài liệu AI-Native chuẩn chỉnh được chia thành 5 nhóm trụ cột chính, mỗi nhóm phục vụ một mục đích cụ thể trong vòng đời suy luận và thực thi của Agent:

```
                  ┌────────────────────────────────────────┐
                  │      AI-Native Markdown Ecosystem      │
                  └───────────────────┬────────────────────┘
                                      │
     ┌────────────────┬───────────────┼───────────────┬────────────────┐
     ▼                ▼               ▼               ▼                ▼
┌──────────┐    ┌───────────┐   ┌───────────┐   ┌───────────┐    ┌───────────┐
│ Steering │    │   Spec    │   │ Execution │   │ Dynamic   │    │ External  │
│    &     │    │     &     │   │     &     │   │   Agent   │    │ Knowledge │
│Governance│    │    ADR    │   │  Memory   │   │  Skills   │    │  Exports  │
└────┬─────┘    └─────┬─────┘   └─────┬─────┘   └─────┬─────┘    └─────┬─────┘
     │                │               │               │                │
     ▼                ▼               ▼               ▼                ▼
AGENTS.md        PRD / Specs    Implementation   SKILL.md         llms.txt
.cursorrules     ADR / RFCs         Plans        (Progressive     llms-full.txt
.cursor/rules/                  Checklists /      Disclosure)
                                Walkthroughs
```

| Nhóm trụ cột | File / Format điển hình | Đối tượng đọc | Mục đích cốt lõi |
| :--- | :--- | :--- | :--- |
| **1. Steering & Governance** | `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/*.mdc` | Toàn bộ Agent sessions | Định hình tính cách, quy tắc code, cấm kỵ, hướng dẫn build/test. |
| **2. Spec & Architecture** | `docs/specs/*.md`, `docs/adr/*.md` | Human & Agent khi lập kế hoạch | "Single Source of Truth" về yêu cầu nghiệp vụ và kiến trúc kỹ thuật. |
| **3. Execution & Memory** | `implementation_plan.md`, `walkthrough.md`, `knowledge/` | Agent trong phiên làm việc | Theo dõi trạng thái tác vụ (State Tracking) và bài học kinh nghiệm (Lessons Learned). |
| **4. Dynamic Skills** | `skills/<skill_name>/SKILL.md` | Agent khi được kích hoạt | Nạp tri thức theo ngữ cảnh (On-demand/Lazy-loading) để tiết kiệm context. |
| **5. External Knowledge** | `llms.txt`, `llms-full.txt` | LLM bên ngoài / Web crawler | Chuẩn hóa tài liệu thư viện/API để LLM bên ngoài dễ dàng tra cứu. |

---

## 2. Chi tiết từng chuẩn tài liệu

### 2.1. Trụ cột 1: Steering & Governance (Chỉ thị & Quy chuẩn hệ thống)

Đây là file được nạp tự động vào system context của AI mỗi khi bắt đầu phiên làm việc.

#### A. Chuẩn Universal: `AGENTS.md` / `CLAUDE.md`
Được chuẩn hóa bởi cộng đồng open-source và các hệ thống agent hiện đại, file này nằm tại root của repository để hướng dẫn tổng quát:

```markdown
# Repository Instructions

## Project Overview
Next.js 15 (App Router) + TypeScript + Prisma + TailwindCSS.

## Commands
- Dev: `pnpm dev`
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`

## Architecture & Conventions
- Data fetching: Use React Server Components by default.
- State management: Zustand for client-side state.
- Validation: Zod schemas colocated with API route handlers.

## Non-Negotiable Rules
- NEVER install new npm packages without explicit user approval.
- ALWAYS write unit tests for utility functions in `src/utils/`.
- NO `any` types allowed in TypeScript code.
```

#### B. Chuẩn Scoped Rules: `.cursor/rules/*.mdc` hoặc `.agents/rules/*.md`
Thay vì dồn một file `AGENTS.md` dài 2000 dòng khiến context bị loãng, các IDE/Agent tiên tiến sử dụng **Scoped Rules** kèm metadata kích hoạt (Globs matching):

```markdown
---
description: Quy chuẩn viết React Server Component và Client Component
globs: ["src/app/**/*.tsx", "src/components/**/*.tsx"]
alwaysApply: false
---

# React Component Rules

1. Thêm directive `'use client';` ở đầu file NẾU VÀ CHỈ NẾU component có hooks (`useState`, `useEffect`) hoặc event listeners (`onClick`).
2. Mọi icon phải import từ `@tabler/icons-react`.
3. Tách biệt rõ ràng Data Fetching Layer (Server) và UI Interaction Layer (Client).
```

> **Nguyên tắc vàng:** *Chỉ nạp luật khi agent thao tác trên các file liên quan (Pattern Matching).*

---

### 2.2. Trụ cột 2: Spec-Driven Development & ADR (Đặc tả & Quyết định kiến trúc)

Trong quy trình **Spec-Driven Development (SDD)**, AI không bao giờ được phép code ngay lập tức. Spec và ADR là la bàn định hướng:

#### A. Architecture Decision Record (ADR)
Mỗi quyết định kiến trúc quan trọng cần được ghi lại dưới dạng bất biến (immutable / append-only) trong `docs/adr/000X-name.md`:

```markdown
# ADR 0004: Sử dụng Event Sourcing cho module Thanh toán

- **Status**: Accepted
- **Date**: 2026-08-30
- **Deciders**: Eric Nguyen, Backend Team

## Context & Problem Statement
Module thanh toán cần đảm bảo khả năng truy vết (Audit Trail) tuyệt đối và đối soát giao dịch tài chính theo thời gian thực.

## Considered Options
1. Bảng CRUD truyền thống + Audit Log riêng lẻ.
2. Event Sourcing toàn phần với PostgreSQL Event Store.

## Decision Outcome
Chọn **Option 2 (Event Sourcing)** vì:
- Dữ liệu không bao giờ bị ghi đè hay xoá mất dấu.
- Có thể replay sự kiện để tái hiện lỗi (reproduce state) chính xác 100%.

## Consequences
- *Positive*: Auditability tối đa, dễ dàng scale read-model bằng CQRS.
- *Negative*: Tăng độ phức tạp khi migration schema sự kiện.
```

#### B. Functional Specification (`docs/specs/*.spec.md`)
Bản đặc tả chi tiết có cấu trúc rõ ràng giúp AI sinh code và unit test bám sát 100% yêu cầu:

```markdown
# Spec: Luồng Đổi Mật Khẩu (Password Reset Flow)

## 1. User Story
Là một người dùng đã đăng ký, tôi muốn đặt lại mật khẩu qua email khi tôi quên.

## 2. API Contract
- `POST /api/v1/auth/reset-password/request`
  - Input: `{ email: string }`
  - Output: `{ message: string }` (Status 200 luôn trả về kể cả khi email không tồn tại để tránh User Enumeration attack).
- `POST /api/v1/auth/reset-password/confirm`
  - Input: `{ token: string, newPassword: string }`

## 3. Acceptance Criteria & Edge Cases
- [ ] Token hết hạn sau đúng 15 phút.
- [ ] Token chỉ sử dụng được 1 lần duy nhất (single-use).
- [ ] Mật khẩu mới không được trùng với 3 mật khẩu gần nhất.
```

---

### 2.3. Trụ cột 3: Execution Artifacts & Memory (Tài liệu thực thi & Trạng thái tác vụ)

Khi thực thi các task phức tạp kéo dài nhiều bước, agent cần duy trì bộ nhớ làm việc (Working Memory) để không bị quên ngữ cảnh giữa chừng:

```
  [User Request] 
         │
         ▼
┌─────────────────────────┐
│  implementation_plan.md │ ◄── Thiết kế giải pháp & kế hoạch xác minh (Review trước khi code)
└────────┬────────────────┘
         │ (User Approved)
         ▼
┌─────────────────────────┐
│     task_list.md        │ ◄── Cập nhật trạng thái từng bước (Todo -> In Progress -> Done)
└────────┬────────────────┘
         │ (Code & Test)
         ▼
┌─────────────────────────┐
│     walkthrough.md      │ ◄── Tóm tắt kết quả, bằng chứng test & diff để kiểm toán
└─────────────────────────┘
```

#### Chuẩn cấu trúc của `implementation_plan.md`:
1. **User Review Required**: Điểm mấu chốt cần người dùng phê duyệt trước (phá vỡ tương thích, schema change).
2. **Proposed Changes**: Danh sách file thêm mới (`[NEW]`), sửa (`[MODIFY]`), xóa (`[DELETE]`).
3. **Verification Plan**: Kịch bản test tự động (`npm test`) và manual test rõ ràng.

#### Chuẩn cấu trúc của `walkthrough.md`:
Bản tóm tắt nghiệm thu sau khi agent hoàn thành, chứa log chạy test thành công, diff quan trọng và hướng dẫn cho người kiểm duyệt.

---

### 2.4. Trụ cột 4: Dynamic Agent Skills & Progressive Disclosure

Đây là kỹ thuật đột phá nhất trong thiết kế Agent hiện đại. Thay vì nhét toàn bộ hướng dẫn deploy, database migration, debug tool... vào một prompt khổng lồ, ta dùng **Progressive Disclosure** (Nạp thông tin theo từng lớp).

```
Level 1: System Prompt chỉ nạp Index (Tên skill + Mô tả 1 dòng ngắn)
                    │
                    ▼ (Khi cần dùng skill cụ thể)
Level 2: Agent tự động đọc SKILL.md chi tiết
                    │
                    ▼ (Khi cần chạy script phụ trợ)
Level 3: Agent đọc và thực thi script trong scripts/ hoặc resources/
```

#### Chuẩn định dạng `SKILL.md` (chuẩn YAML Frontmatter):

```markdown
---
name: db-migration-helper
description: Hướng dẫn an toàn khi tạo và chạy Prisma migration cho PostgreSQL trong môi trường production.
trigger_conditions:
  - Khi người dùng yêu cầu sửa đổi `schema.prisma`
  - Khi thêm bảng mới hoặc sửa kiểu dữ liệu cột
---

# Prisma Database Migration Workflow

## 1. Quy tắc an toàn
- Tuyệt đối KHÔNG xóa cột trực tiếp (Zero-downtime migration: Thêm cột mới -> Deprecate cột cũ -> Xóa sau).
- Luôn kiểm tra lock timeout đối với bảng trên 1 triệu rows.

## 2. Các bước thực hiện chuẩn
1. Sửa `prisma/schema.prisma`.
2. Tạo migration dạng preview:
   ```bash
   npx prisma migrate dev --create-only --name <ten_migration>
   ```
3. Kiểm tra file SQL sinh ra trong `prisma/migrations/`.
4. Chạy kiểm tra dữ liệu mẫu (Seeding & Verification).
```

---

### 2.5. Trụ cột 5: Chuẩn `llms.txt` (External Documentation Standard)

Được khởi xướng bởi Jeremy Howard (Answer.AI) và được các tổ chức lớn như FastHTML, Anthropic, Vercel ủng hộ, `llms.txt` là file Markdown chuẩn nằm ở root website (`https://your-domain.com/llms.txt`) cung cấp thông tin ngắn gọn, súc tích dành riêng cho LLM:

```markdown
# MyProject Documentation for LLMs

> Thư viện xử lý streaming dữ liệu thời gian thực hiệu năng cao cho TypeScript.

## Core APIs
- [EventStream](https://example.com/docs/event-stream.md): Xử lý luồng SSE.
- [WebSocketClient](https://example.com/docs/ws-client.md): Client socket có auto-reconnect.

## Optional & Deep Dive
- [Full LLM Dump](https://example.com/llms-full.txt): Toàn bộ tài liệu gộp trong 1 file text phẳng.
```

---

## 3. Cấu trúc thư mục mẫu chuẩn cho một dự án AI-Native

Dưới đây là cây thư mục đề xuất (Reference Architecture) gom toàn bộ các chuẩn trên vào một codebase thực tế:

```text
my-awesome-project/
├── .agents/                        # Cấu hình AI Agent cấp repository
│   ├── rules/                      # Luật theo ngữ cảnh (Scoped Rules)
│   │   ├── code-style.md
│   │   ├── security-guidelines.md
│   │   └── api-design.md
│   └── skills/                     # Các skill nạp động (Progressive Disclosure)
│       ├── database-migration/
│       │   └── SKILL.md
│       └── e2e-testing/
│           ├── SKILL.md
│           └── scripts/run-smoke.sh
├── .cursor/                        # Cấu hình riêng cho Cursor IDE (nếu dùng)
│   └── rules/
│       ├── nextjs-patterns.mdc
│       └── tailwind-conventions.mdc
├── docs/                           # Tài liệu thiết kế & kiến trúc (Con người & AI cùng đọc)
│   ├── adr/                        # Architecture Decision Records
│   │   ├── 0001-monorepo-structure.md
│   │   └── 0002-authentication-jwt-vs-session.md
│   ├── specs/                      # Đặc tả chức năng (Spec-Driven Development)
│   │   ├── user-onboarding.spec.md
│   │   └── payment-webhook.spec.md
│   └── architecture-overview.md
├── knowledge/                      # Bộ nhớ tích lũy (Knowledge Base / Known Pitfalls)
│   ├── metadata.json               # Index tóm tắt để search nhanh
│   ├── third-party-gotchas.md      # Các bẫy thư viện bên thứ 3 hay gặp
│   └── incident-postmortems.md     # Bài học từ các sự cố quá khứ
├── AGENTS.md                       # Chỉ thị toàn cục cho Agent
├── CLAUDE.md                       # Chỉ thị cho Claude Code / Anthropic ecosystem
├── llms.txt                        # Tra cứu nhanh cho LLM bên ngoài
└── README.md                       # Dành cho con người đọc
```

---

## 4. Các nguyên tắc viết Markdown tối ưu hóa cho LLM Token

Khi viết tài liệu cho AI đọc, phong cách hành văn cần tuân thủ 5 nguyên tắc kỹ thuật sau:

### 1. High Signal-to-Noise Ratio (Tỷ lệ thông tin / Token tối đa)
* **Kém:** *"Trong phần này, chúng ta sẽ cùng nhau thảo luận về cách tốt nhất để cấu hình Prisma. Việc cấu hình đúng là rất quan trọng vì nó giúp hệ thống chạy mượt hơn..."* (Nhiều từ thừa thãi, tốn token).
* **Chuẩn:** 
  ```markdown
  ### Prisma Configuration Requirements
  - Connection pooling: Max 10 connections per serverless instance.
  - Timeout: Set `statement_timeout = 5000ms`.
  ```

### 2. Ưu tiên GFM Tables và Code Snippets
LLM xử lý cấu trúc dạng bảng (Table) và cặp khóa-giá trị (Key-Value) nhanh và chính xác hơn nhiều so với việc phân tích đoạn văn dài.

### 3. File Anchoring & Line Range Links
Khi trích dẫn code trong markdown, hãy cung cấp đường dẫn chính xác kèm line range:
```markdown
Tham khảo cách xử lý lỗi tại [error-handler.ts:L45-L60](file:///src/utils/error-handler.ts#L45-L60).
```

### 4. Deterministic Frontmatter (Dùng YAML có schema)
Mọi tài liệu skill hoặc rule nên có frontmatter với các trường chuẩn: `title`, `description`, `globs`, `tags`, `dependencies`.

---

## 5. Bảng so sánh các chuẩn phổ biến hiện nay

| Chuẩn / Format | Công cụ hỗ trợ chính | Điểm mạnh nhất | Điểm cần lưu ý |
| :--- | :--- | :--- | :--- |
| **`AGENTS.md`** | Đa nền tảng (Universal) | Đơn giản, dùng chung cho mọi tool (Aider, Claude, Roo Code, Antigravity...) | Dễ bị quá tải nếu dồn quá nhiều quy tắc vào 1 file. |
| **`.cursor/rules/*.mdc`** | Cursor IDE | Hỗ trợ globs matching tự động nạp theo đuôi file | Phụ thuộc vào hệ sinh thái Cursor. |
| **`SKILL.md` (Progressive)** | Antigravity / Custom Agentic SDKs | Tiết kiệm token tối đa qua 3 tầng nạp động | Cần agent runtime có hỗ trợ đọc file theo nhu cầu. |
| **`docs/adr/*.md`** | Mọi dự án phần mềm | Lưu trữ lịch sử quyết định kiến trúc bất biến | Cần kỹ sư duy trì tính cập nhật. |
| **`llms.txt`** | Web crawlers, AI Research tools | Định dạng chuẩn cho public API & library docs | Phù hợp nhất cho thư viện hoặc open-source packages. |

---

## 6. Checklist áp dụng ngay hôm nay

Nếu bạn đang bắt đầu một dự án mới hoặc muốn "AI-native hóa" repo hiện tại, hãy làm theo các bước:

- [ ] **Bước 1: Tạo `AGENTS.md` ở root**: Ghi rõ tech-stack, lệnh test/build và 3-5 điều cấm kỵ cốt lõi.
- [ ] **Bước 2: Phân tách Scoped Rules**: Tách các quy tắc chi tiết theo framework (React, DB, Test) vào thư mục `.agents/rules/` hoặc `.cursor/rules/`.
- [ ] **Bước 3: Thiết lập `docs/adr/` và `docs/specs/`**: Viết spec trước khi yêu cầu AI code tính năng mới.
- [ ] **Bước 4: Áp dụng luồng Plan-First**: Yêu cầu AI sinh `implementation_plan.md` -> Review & Phê duyệt -> Mới cho phép sinh code.
- [ ] **Bước 5: Lưu lại bài học vào `knowledge/`**: Mỗi khi sửa được một bug hóc búa, lưu lại giải pháp vào file markdown để AI không lặp lại sai lầm trong tương lai.

---

## Lời kết

Trong phát triển phần mềm AI-Native, **Code là sản phẩm đầu ra, nhưng Documentation chính là bộ điều khiển và bộ nhớ của hệ thống**. Một kho tài liệu `.md` được cấu trúc bài bản, phân tầng rõ ràng sẽ biến AI Agent từ một "thực tập sinh hay quên" thành một "senior engineer" thấu hiểu sâu sắc toàn bộ hệ thống của bạn.
