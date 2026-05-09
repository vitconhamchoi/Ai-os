# 02 - Lộ trình xây AI OS từ MVP tới production

## 1. Mục tiêu của tài liệu này

Tài liệu này trả lời câu hỏi: **nên xây AI OS theo thứ tự nào để ra được hệ thống usable, không sa đà vào demo đẹp nhưng không deploy được.**

---

## 2. Nguyên tắc triển khai

Trước khi chia phase, cần chốt 4 nguyên tắc:

1. bắt đầu từ use case, không bắt đầu từ model
2. mọi năng lực mới đều phải gắn với metric đo được
3. ưu tiên control plane trước khi mở rộng autonomy
4. rollout theo domain hẹp rồi mới nhân rộng

Nếu làm ngược, dự án rất dễ rơi vào tình trạng:

- agent trả lời hay nhưng không giải quyết tác vụ
- tool nhiều nhưng không ai dám cho chạy thật
- chi phí inference tăng nhanh nhưng giá trị tạo ra thấp

---

## 3. Backlog tối thiểu để có MVP

Một MVP đúng nghĩa cần đủ 6 cụm việc sau:

### 3.1. Use case definition

Chọn 1-2 use case thật cụ thể, ví dụ:

- xử lý ticket nội bộ theo SOP
- tổng hợp báo cáo vận hành hằng ngày
- tra cứu tri thức và đề xuất hành động tiếp theo

Deliverable:

- mục tiêu đầu ra
- nguồn dữ liệu cần dùng
- tool cần tích hợp
- điều kiện thành công/thất bại
- bước nào bắt buộc human approval

### 3.2. Core platform skeleton

Phải dựng được:

- intent gateway
- orchestrator
- tool router
- memory retrieval cơ bản
- audit log

Deliverable:

- một request đi qua được toàn bộ pipeline
- trace được từng bước
- replay được lỗi

### 3.3. Connector layer

Chỉ tích hợp những tool phục vụ trực tiếp use case MVP.

Không nên mở quá rộng. Mỗi connector cần:

- auth riêng
- schema vào/ra
- error mapping
- timeout/retry rule

### 3.4. Guardrail cơ bản

Tối thiểu phải có:

- authn/authz
- redaction dữ liệu nhạy cảm
- allowlist tool
- rate limit
- approval cho hành động rủi ro

### 3.5. User surface

Tạo giao diện đủ dùng, không cần đẹp:

- nơi gửi yêu cầu
- nơi xem trạng thái task
- nơi phê duyệt hoặc từ chối action
- nơi xem lịch sử đã chạy

### 3.6. Evaluation + monitoring

Ngay từ MVP đã phải có:

- task success rate
- average completion time
- tool failure rate
- handoff rate
- cost per task

---

## 4. Kế hoạch 90 ngày thực dụng

## Giai đoạn 0 - Chuẩn bị (tuần 1-2)

Mục tiêu:

- chọn use case
- chốt owner
- chốt boundary dữ liệu và quyền
- vẽ sơ đồ kiến trúc đầu tiên

Kết quả cần có:

- PRD ngắn cho từng use case
- danh sách tool sẽ tích hợp
- risk register bản đầu
- KPI pilot

## Giai đoạn 1 - Dựng khung chạy được (tuần 3-5)

Mục tiêu:

- có đường đi end-to-end cho một task
- agent gọi được ít nhất 1-2 tool
- log được toàn bộ execution

Kết quả cần có:

- service skeleton
- schema cho intent/tool/action
- tracing ID xuyên suốt request
- task inbox hoặc UI tối thiểu

## Giai đoạn 2 - Làm hệ thống usable (tuần 6-8)

Mục tiêu:

- thêm retrieval context
- thêm policy
- thêm approval flow
- xử lý lỗi và retry tử tế

Kết quả cần có:

- task chạy được với dữ liệu nội bộ thật
- action rủi ro bị chặn đúng rule
- người dùng nhìn thấy trạng thái rõ ràng

## Giai đoạn 3 - Pilot production (tuần 9-12)

Mục tiêu:

- chạy với user thật hoặc team thật
- đo được chất lượng
- khóa chi phí và rủi ro

Kết quả cần có:

- dashboard KPI
- quota/cost control
- runbook on-call
- đánh giá nên mở rộng, giữ nguyên hay dừng

---

## 5. Thứ tự ưu tiên module

Nếu nguồn lực ít, nên build theo thứ tự này:

1. schema + intent gateway
2. orchestrator
3. tool router
4. logging/audit
5. approval flow
6. retrieval/memory
7. UI cho user
8. evaluation automation
9. multi-agent

Lưu ý: nhiều đội làm ngược, cố làm multi-agent trước. Đây thường là sai lầm vì hệ thống chưa có control plane đủ mạnh.

---

## 6. Một backlog mẫu cho team sản phẩm

### Track A - Platform

- định nghĩa schema chuẩn cho request, plan step, tool call, decision
- dựng orchestration state machine
- thêm retry/timeout/circuit breaker
- thêm cost tracking theo request

### Track B - Integration

- kết nối 3 hệ thống nguồn đầu tiên
- chuẩn hóa auth và permission cho connector
- ghi log input/output rút gọn cho connector

### Track C - Trust & Safety

- phân cấp tool theo mức rủi ro
- approval workflow cho action create/update/delete
- redaction dữ liệu trước khi lưu hoặc prompt
- audit trail cho mọi hành động quan trọng

### Track D - Product surface

- inbox cho task đang chạy
- panel xem plan step
- panel approval
- lịch sử task + kết quả cuối

### Track E - Evaluation

- bộ sample task để regression
- score quality theo từng use case
- theo dõi false positive / false action
- báo cáo cost và latency theo tuần

---

## 7. Cách chọn stack mà không tự làm khó mình

### Backend

Chọn công nghệ backend mà team đang vận hành ổn nhất. AI OS thất bại thường không phải vì ngôn ngữ, mà vì:

- service khó maintain
- connector khó debug
- không có tracing xuyên suốt

### Data

Tách rõ:

- transactional store cho task state
- object/document store cho audit và payload lớn
- retrieval index cho knowledge

Không ép một loại database làm tất cả.

### Model layer

Nên có abstraction từ đầu để thay được:

- model reasoning
- model extraction/classification
- embedding model

Không hardcode logic sản phẩm vào đặc tính của duy nhất một model provider.

---

## 8. Definition of done cho từng mốc

### MVP done khi:

- một task thật chạy end-to-end
- có log và trace đầy đủ
- user hiểu được hệ thống đang làm gì
- action nhạy cảm không tự chạy vô kiểm soát

### Pilot done khi:

- KPI đo được tối thiểu 2-4 tuần
- chi phí trên mỗi task nằm trong ngưỡng chấp nhận
- failure pattern đã được phân loại
- có runbook cho tình huống lỗi phổ biến

### Production-ready khi:

- có tenant/workspace isolation
- có RBAC rõ
- có review và approval rule
- có budget/cap/rate control
- có evaluation lặp lại định kỳ

---

## 9. Những thứ chưa nên làm quá sớm

- chưa cần tự huấn luyện model riêng nếu use case chưa chứng minh đủ giá trị
- chưa cần knowledge graph phức tạp nếu retrieval cơ bản còn chưa ổn
- chưa cần mở hàng chục connector nếu 3 connector đầu còn chưa bền
- chưa cần agent tự do dài chuỗi hành động nếu chưa có approval và rollback

AI OS bền là hệ được **giới hạn hợp lý**, không phải hệ có autonomy tối đa ngay từ đầu.

---

## 10. Đầu ra mong muốn sau tài liệu này

Sau khi đọc xong file này, đội triển khai cần chốt được:

- use case đầu tiên
- backlog 90 ngày
- cấu trúc repo/service
- tiêu chí MVP và pilot
- các điểm chặn bắt buộc về bảo mật, quyền và approval

Khi đã có 5 điểm trên, chuyển sang [`03-van-hanh-va-su-dung.md`](./03-van-hanh-va-su-dung.md) để thiết kế cách người dùng và đội vận hành sử dụng hệ thống mỗi ngày.
