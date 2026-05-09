# 01 - Kiến trúc tham chiếu cho AI OS

## 1. Mục tiêu của tài liệu này

Tài liệu này không mô tả AI OS theo kiểu khái niệm chung chung, mà chốt một câu hỏi thực tế:

**nếu bắt đầu xây một AI OS hôm nay, cần những khối nào, nối với nhau ra sao, và nên cắt biên hệ thống như thế nào để còn đưa vào production.**

---

## 2. Định nghĩa phạm vi sản phẩm

Trong repo này, AI OS được hiểu là một nền tảng có 5 khả năng cốt lõi:

1. nhận yêu cầu từ người dùng hoặc hệ thống
2. biến yêu cầu thành plan có thể chạy
3. dùng được context, memory và tri thức nội bộ
4. hành động qua tool/API/workflow
5. bị kiểm soát bởi policy, audit và human approval

Như vậy, phạm vi **không** phải là viết kernel hay desktop OS mới. Phạm vi đúng là xây một **agent runtime platform** có các thành phần đủ chặt để:

- gắn vào sản phẩm hiện hữu
- mở rộng được thêm agent/use case
- quan sát được khi chạy thật
- rollback được khi agent làm sai

---

## 3. Kiến trúc mức cao

```text
User / App / Event
        |
        v
 [Intent Gateway]
        |
        v
 [Orchestrator / Planner] <----------------------+
        |                                        |
        |                                [Policy Engine]
        v                                        |
 [Context + Memory Layer]                        |
        |                                        |
        v                                        |
 [Tool Router / Workflow Executor] --------------+
        |
        v
 Internal APIs / SaaS / Browser / DB / Queue / Human Approval

 Side layers:
 - Observability
 - Identity & Access Control
 - Prompt / Output Safety
 - Cost & Rate Control
```

---

## 4. Các thành phần bắt buộc

### 4.1. Intent Gateway

Đây là đầu vào thống nhất cho cả người dùng và hệ thống.

Nó nên nhận ít nhất 4 loại request:

- chat request từ UI
- action request từ application
- event từ webhook/queue
- scheduled job từ workflow

Đầu ra của lớp này không nên là prompt tự do, mà là một payload chuẩn hóa, ví dụ:

- `actor`
- `workspace`
- `goal`
- `constraints`
- `attachments`
- `risk_level`
- `requested_tools`

**Quy tắc thiết kế:** mọi luồng đều phải đi qua một schema đầu vào chung, nếu không sau này rất khó policy hóa.

### 4.2. Orchestrator / Planner

Đây là lõi của AI OS.

Nó làm 4 việc:

1. phân loại nhiệm vụ
2. quyết định single-agent hay multi-agent
3. chọn tool hoặc workflow cần gọi
4. tạo execution state để theo dõi tiến trình

Ở giai đoạn MVP, không nên làm planner quá tự do. Nên dùng mô hình:

- một catalog use case rõ ràng
- mỗi use case có allowed tools
- planner chỉ chọn trong phạm vi đó

Làm như vậy chậm hơn về mặt “ảo tưởng thông minh”, nhưng nhanh hơn về mặt production.

### 4.3. Context + Memory Layer

Lớp này nên tách làm 3 vùng:

- **session state**: trạng thái chạy hiện tại
- **retrieval context**: tài liệu, SOP, ticket, spec, knowledge base
- **long-term memory**: hồ sơ người dùng, lịch sử tác vụ, preference, kết quả đã xác nhận

Không nên gom tất cả vào một vector store rồi gọi chung là memory. Về vận hành, 3 vùng này có vòng đời dữ liệu khác nhau, policy khác nhau và chi phí khác nhau.

### 4.4. Tool Router / Workflow Executor

Thành phần này quyết định AI có “làm được việc” hay chỉ “trả lời hay”.

Nó cần làm được:

- map intent sang tool cụ thể
- kiểm tra quyền trước khi gọi
- ép tham số theo schema
- timeout, retry, circuit breaker
- lưu trace input/output của từng tool call
- gửi sang human approval nếu vượt ngưỡng rủi ro

Nếu không có lớp này, mọi tool call sẽ bị nhúng thẳng vào agent code và hệ thống sẽ nhanh chóng không kiểm soát nổi.

### 4.5. Policy Engine

Đây là lớp bắt buộc trước production.

Nó nên trả lời được các câu hỏi như:

- user này có được đọc nguồn dữ liệu này không?
- agent này có được gọi công cụ tạo lệnh thanh toán không?
- action này có cần human approval không?
- output này có chứa dữ liệu nhạy cảm hoặc thông tin vượt phạm vi không?

Một policy tối thiểu nên xét theo:

- actor
- workspace/tenant
- tool
- data class
- risk level
- execution mode

### 4.6. Observability

Muốn vận hành AI OS thật thì không được chỉ log mỗi prompt và response.

Cần ít nhất:

- trace theo từng request
- step log theo từng plan/tool call
- token/model cost
- latency theo agent và theo tool
- approval rate
- failure rate
- fallback-to-human rate

Nếu thiếu lớp này, đội vận hành sẽ không biết hệ thống hỏng ở reasoning, retrieval hay integration.

---

## 5. Boundary cần chốt ngay từ đầu

### 5.1. Agent không được tự có toàn quyền

Không agent nào được:

- tự mở rộng danh sách tool
- tự đổi policy
- tự lấy secret gốc
- tự ghi trực tiếp vào hệ thống trọng yếu mà không qua guardrail

### 5.2. Memory không được dùng như data lake vô kiểm soát

Cần phân loại rõ:

- dữ liệu chỉ cho truy xuất tạm thời
- dữ liệu được phép lưu dài hạn
- dữ liệu phải redaction trước khi lưu
- dữ liệu bị cấm đưa vào prompt

### 5.3. Tool phải có hợp đồng rõ ràng

Mỗi tool tích hợp nên có:

- mục đích
- input schema
- output schema
- quyền yêu cầu
- mức rủi ro
- điều kiện approval
- owner chịu trách nhiệm

Nếu một tool không mô tả được 7 điểm này thì chưa nên cho AI gọi.

---

## 6. Kiến trúc triển khai theo giai đoạn

## Giai đoạn 1: MVP

Mục tiêu:

- 1 orchestrator
- 3-5 tool nội bộ
- 1 memory retrieval đơn giản
- 1 hàng đợi xử lý tác vụ
- 1 dashboard quan sát cơ bản

Đủ để chứng minh:

- hệ thống hiểu việc
- gọi tool đúng
- không vượt quyền
- có log để debug

## Giai đoạn 2: Production pilot

Thêm vào:

- approval flow
- role-based access control
- tool risk classification
- retry/circuit breaker
- evaluation pipeline
- ngân sách cost theo team/use case

## Giai đoạn 3: Scale

Thêm vào:

- multi-agent theo domain
- memory dài hạn có lifecycle rõ
- cache/context optimization
- model routing
- multi-tenant isolation
- policy-as-code

---

## 7. Một kiến trúc repo thực dụng để bắt đầu

```text
/apps
  /control-plane        # dashboard, policy admin, audit viewer
  /user-portal          # giao diện người dùng / chat / task inbox

/services
  /intent-gateway
  /orchestrator
  /memory-service
  /tool-router
  /policy-service
  /evaluation-service

/connectors
  /crm
  /erp
  /helpdesk
  /browser
  /database

/agents
  /support-agent
  /ops-agent
  /research-agent

/schemas
  /intent
  /tool
  /policy
  /event

/infra
  /observability
  /deployment
  /secrets
```

Điểm quan trọng là **tách connector, policy và orchestrator**. Đừng nhét tất cả vào một service “agent-api” duy nhất.

---

## 8. Quyết định công nghệ ở mức thực tế

Không có stack duy nhất đúng cho mọi đội, nhưng có vài nguyên tắc:

- gateway và orchestrator nên dùng stack backend mà team hiện đang vận hành tốt nhất
- tool connector nên ưu tiên service nhỏ, schema rõ, dễ test
- memory nên bắt đầu từ retrieval đơn giản trước, chưa cần knowledge graph nếu chưa có use case rõ
- queue/event bus nên có nếu tác vụ dài hơn vài giây
- observability nên dùng hệ thống team đã quen để giảm friction

Thứ tự ưu tiên đúng là:

1. quan sát được
2. kiểm soát được
3. tích hợp được
4. rồi mới tối ưu “độ thông minh”

---

## 9. Checklist chốt kiến trúc trước khi code

- đã định nghĩa 3 use case đầu tiên chưa
- đã chốt input schema chung chưa
- đã phân loại tool theo mức rủi ro chưa
- đã có approval rule cho action nhạy cảm chưa
- đã tách session state với long-term memory chưa
- đã định nghĩa audit log cần lưu gì chưa
- đã chốt ai là owner cho từng connector chưa
- đã chốt KPI pilot chưa

Nếu còn thiếu từ 3 mục trở lên thì chưa nên lao vào build.

---

## 10. Tài liệu tiếp theo cần đọc

Sau khi chốt kiến trúc, đọc tiếp [`02-lo-trinh-xay-dung.md`](./02-lo-trinh-xay-dung.md) để chuyển từ sơ đồ sang backlog, mốc bàn giao và cách dựng MVP.
