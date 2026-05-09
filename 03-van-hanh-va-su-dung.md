# 03 - Vận hành và sử dụng AI OS trong thực tế

## 1. Mục tiêu của tài liệu này

Nếu `01-kien-truc-tham-chieu.md` trả lời “phải có những khối nào” và `02-lo-trinh-xay-dung.md` trả lời “phải xây theo thứ tự nào”, thì file này trả lời:

**khi hệ thống đã chạy, người dùng, admin và đội vận hành sẽ dùng nó ra sao để không biến AI OS thành một hộp đen khó kiểm soát.**

---

## 2. Ba nhóm người dùng chính

### 2.1. End user

Người tạo yêu cầu, ví dụ:

- hỏi hệ thống để lấy thông tin
- giao task cho agent xử lý
- nhận kết quả hoặc yêu cầu bổ sung dữ liệu

Điều end user cần không phải “AI nói hay”, mà là:

- gửi yêu cầu nhanh
- biết task đang ở bước nào
- biết khi nào cần họ xác nhận
- nhận kết quả có thể kiểm chứng

### 2.2. Operator / team vận hành

Đây là nhóm theo dõi hệ thống hàng ngày:

- thấy task lỗi
- thấy tool nào hay timeout
- thấy approval nào bị kẹt
- thấy cost nào đang tăng bất thường

### 2.3. Admin / owner nghiệp vụ

Nhóm này quản lý:

- policy
- quyền theo role
- tool catalog
- ngưỡng rủi ro
- scope rollout theo team hoặc domain

---

## 3. Luồng sử dụng chuẩn

Một luồng sử dụng đúng trong AI OS nên như sau:

1. người dùng gửi mục tiêu
2. hệ thống chuẩn hóa intent
3. orchestrator chia task thành step
4. hệ thống hiển thị plan hoặc trạng thái chính
5. agent gọi tool theo policy
6. nếu gặp step nhạy cảm thì gửi approval
7. khi xong thì trả kết quả, log, và bằng chứng liên quan

Điểm quan trọng là user phải nhìn thấy:

- hệ thống đang làm gì
- dựa trên dữ liệu nào
- bước nào đã chạy
- bước nào đang chờ người duyệt

Nếu không có 4 thứ đó, AI OS sẽ nhanh chóng mất niềm tin từ người dùng.

---

## 4. Giao diện tối thiểu nên có

Không cần đợi làm UI đẹp mới pilot. Nhưng phải có ít nhất 5 khu vực:

### 4.1. Task composer

Nơi user nhập:

- mục tiêu
- dữ liệu bổ sung
- deadline hoặc độ ưu tiên
- phạm vi hệ thống được phép dùng

### 4.2. Task timeline

Cho user và operator thấy:

- task đã vào hệ thống chưa
- đang reasoning hay đang chờ tool
- step nào lỗi
- step nào bị chặn bởi policy

### 4.3. Approval inbox

Đây là chỗ rất quan trọng trong hệ thống thật.

Mỗi approval nên hiển thị:

- agent muốn làm gì
- vì sao muốn làm
- tác động lên hệ thống nào
- dữ liệu nào sẽ bị thay đổi
- nút approve / reject / request changes

### 4.4. Result panel

Không chỉ trả một đoạn text. Kết quả nên có:

- summary cuối
- nguồn tham chiếu
- action đã chạy
- action nào không chạy được
- khuyến nghị bước tiếp theo

### 4.5. Ops dashboard

Cho operator xem:

- queue depth
- task success rate
- approval backlog
- tool error rate
- latency theo service
- cost theo use case

---

## 5. Quy trình vận hành hàng ngày

## Buổi sáng

Operator nên kiểm tra:

- task fail từ đêm trước
- approval còn treo
- connector đang lỗi hoặc chậm
- ngân sách đã dùng

## Trong ngày

Theo dõi:

- request tăng đột biến
- một tool bị lỗi hàng loạt
- agent lặp step bất thường
- output bị người dùng reject quá nhiều

## Cuối ngày / cuối tuần

Review:

- top task lỗi
- top approval bị từ chối
- top use case tạo giá trị
- top điểm nghẽn khiến phải handoff cho người

Không có review định kỳ thì AI OS sẽ bị “trôi chất lượng” rất nhanh.

---

## 6. Runbook cho sự cố phổ biến

### 6.1. Tool connector timeout

Biểu hiện:

- task treo lâu
- step retry nhiều lần
- latency tăng mạnh

Xử lý:

- ngắt connector lỗi khỏi allowlist tạm thời
- chuyển use case sang mode read-only hoặc human fallback
- kiểm tra auth, quota, rate limit và downstream health

### 6.2. Agent hành động vượt phạm vi

Biểu hiện:

- gọi sai tool
- đề xuất action không đúng role
- truy cập dữ liệu ngoài tenant/workspace

Xử lý:

- chặn policy ngay tầng gateway/tool router
- replay trace để xác định lỗi ở intent mapping hay rule
- thêm regression case cho tình huống đó

### 6.3. Cost tăng bất thường

Biểu hiện:

- token usage tăng
- task kéo dài hơn bình thường
- retrieval nạp quá nhiều context

Xử lý:

- giới hạn context window theo use case
- cache các bước classification/retrieval lặp lại
- ép planner dùng workflow ngắn hơn

### 6.4. Approval backlog tăng

Biểu hiện:

- request chờ duyệt quá lâu
- user đánh giá hệ thống “chậm”

Xử lý:

- gom approval theo batch nếu phù hợp
- chỉnh lại risk threshold
- tách action nào thực sự cần duyệt, action nào có thể auto-approve

---

## 7. KPI nên theo dõi

### KPI chất lượng

- task success rate
- factual correctness với use case cần độ chính xác cao
- tỷ lệ user chấp nhận kết quả
- số lần phải sửa bằng tay sau khi agent chạy

### KPI vận hành

- p50/p95 completion time
- tool failure rate
- retry rate
- approval turnaround time

### KPI kinh doanh

- thời gian tiết kiệm được cho mỗi task
- số ticket hoặc workflow xử lý tự động
- tỷ lệ adoption theo team
- cost trên một outcome hoàn thành

---

## 8. Cách rollout cho người dùng thật

Nên rollout theo 4 vòng:

1. internal dogfooding
2. pilot với một team nhỏ
3. mở theo workspace/domain
4. rollout rộng sau khi KPI ổn định

Ở mỗi vòng, cần chốt:

- ai được dùng
- họ dùng use case nào
- mức autonomy tới đâu
- ai chịu trách nhiệm khi có lỗi

---

## 9. Tiêu chí để nói hệ thống đang dùng được

AI OS được xem là dùng được khi:

- user hiểu hệ thống đang xử lý gì
- operator có cách phát hiện và khoanh vùng lỗi
- admin chỉnh được policy mà không phải sửa tay mọi service
- task tạo ra kết quả thật, không chỉ sinh văn bản đẹp

---

## 10. Bộ tài liệu này nên mở rộng tiếp như thế nào

Sau file này, các phần nên viết tiếp nếu muốn biến repo thành một bộ playbook hoàn chỉnh là:

- security baseline chi tiết theo mức rủi ro tool
- connector pattern cho từng nhóm hệ thống
- evaluation handbook cho agent quality
- deployment guide theo cloud / hybrid / on-prem

Ở điểm hiện tại, bộ 3 file đã đủ để chuyển repo từ mức “giải thích khái niệm” sang mức “định hướng thiết kế, xây dựng và vận hành thực tế”.
