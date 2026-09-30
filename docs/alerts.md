# Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert mẫu để tham khảo

Ví dụ dưới đây minh họa mức độ cụ thể cần có. Học viên không cần copy nguyên, nhưng ba alert trong bài nộp nên rõ ràng tương tự: điều kiện là gì, kéo dài bao lâu, ảnh hưởng tới user ra sao và người trực cần kiểm tra gì trước.

- Tên: `HighLatencyP95`
- Severity: `warning`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: latency P95 của `response_sent.latency_ms`
- Điều kiện và thời gian duy trì: `p95(latency_ms) > 3000ms` trong 5 phút
- Ảnh hưởng tới người dùng: người dùng phải chờ lâu hơn trước khi nhận câu trả lời
- Ba bước kiểm tra đầu tiên:
  1. Mở dashboard latency để xác nhận P95/P99 và khoảng thời gian tăng.
  2. Lọc `data/logs.jsonl` trong khoảng đó, lấy một `correlation_id` có `latency_ms` cao.
  3. Mở trace cùng `correlation_id` trên Langfuse, so sánh các span chính để xác định bước nào bất thường.
- Mitigation tạm thời: dựa trên evidence thực tế để rollback prompt, khôi phục cấu hình liên quan, tắt practice scenario hoặc giảm tải khi demo.
- Owner: `student-<MSSV>`

## Alert 1: High Latency P95

- Severity: `warning`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: `response_sent.latency_ms`, SLO latency <= 3000ms
- Điều kiện: `p95(latency_ms) > 3000ms` trong 5 phút
- Ảnh hưởng: người dùng phải chờ lâu hơn trước khi nhận câu trả lời.
- Kiểm tra: xác nhận panel latency; lọc log lấy `correlation_id` có latency cao; mở trace cùng ID và so sánh retrieval với generation.
- Mitigation: rollback prompt về version production ổn định, tắt scenario gây tải hoặc giảm concurrency.
- Owner: `student-oncall`

## Alert 2: High Error Rate

- Severity: `critical`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: tỷ lệ request lỗi và SLO successful requests.
- Điều kiện: `error_rate_pct > 2%` trong 5 phút
- Ảnh hưởng: người dùng nhận lỗi hoặc không nhận được câu trả lời.
- Kiểm tra: xác nhận panel errors; lọc `request_failed` theo `error_type` và `correlation_id`; mở trace để xem span lỗi.
- Mitigation: tắt incident practice đang gây lỗi, khôi phục dependency/configuration ổn định và theo dõi error rate.
- Owner: `student-oncall`

## Alert 3: Low Retrieval Success

- Severity: `warning`
- Duration: `10m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: `retrieval_success_rate_pct`, guardrail tối thiểu 90%.
- Điều kiện: `retrieval_success_rate_pct < 90%` trong 10 phút
- Ảnh hưởng: câu trả lời có thể thiếu context hoặc giảm chất lượng.
- Kiểm tra: xem panel errors/retrieval; lọc log có `tool_success=false`; mở trace và kiểm tra retriever span.
- Mitigation: khôi phục vector store/configuration, chuyển tạm sang fallback an toàn và đánh giá lại quality score.
- Owner: `student-oncall`
