# Báo cáo cá nhân — K4-L3B Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Evidence thực hiện theo [`docs/SCREENSHOT_GUIDE.md`](../docs/SCREENSHOT_GUIDE.md); dùng đường dẫn tương đối.

## 1. Thông tin học viên

- **Họ và tên:** Trần Hoàng Duy Anh
- **MSSV:** 2A202602558
- **Lớp:** K4-L3B
- **Repository URL:** https://github.com/Josee0801/K4-L3-DAY13-TranHoangDuyAnh-2A202602558-Monitoring-LLMOps
- **Commit SHA cuối:** `f34dd5969073c069517f42a46251eff1e2f40fbb` (cập nhật lại sau commit cuối)
- **Challenge ID:** Chưa được Lab Coach cung cấp
- **Tên project Langfuse cá nhân:** `day13-k4-l3b-2A202602558`

## 2. Evidence index

Giữ các evidence theo [`docs/SCREENSHOT_GUIDE.md`](../docs/SCREENSHOT_GUIDE.md). Có thể dùng output text cho ba validator và ảnh cho các mục runtime.

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.png` hoặc `evidence/01-pytest.txt` |
| Log validator | `evidence/02-log-validator.png` hoặc `evidence/02-log-validator.txt` |
| Dashboard validator | `evidence/03-dashboard-validator.png` hoặc `evidence/03-dashboard-validator.txt` |
| Structured log | `evidence/04-structured-log.png` |
| PII redaction | `evidence/05-pii-redaction.png` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard overview | `evidence/11-dashboard-overview.png` |
| Incident metric | `evidence/12-incident-metric.png` |
| Incident log | `evidence/13-incident-log.png` |
| Incident trace | `evidence/14-incident-trace.png` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | 80/100 | 100/100 | Baseline thiếu enrichment; kết quả cuối đủ context và không có PII leak. |
| `validate_dashboard.py` | 6/6 panel | 6/6 panel | Dashboard contract hợp lệ. |
| `pytest` | 22 passed | 22 passed | Baseline test pass. |
| Số traces hợp lệ | 0 | 10 root traces + child spans | Project Langfuse cá nhân đã nhận 10 `lab-agent-run`, 10 `retrieval` và 10 `llm-generation`. |
| Số PII leak | 0 | 0 | Validator không phát hiện PII trong log runtime. |
| Latency P95 / TTFT P95 | Chưa ghi nhận | 211ms / 64ms | Tính từ 70 event `response_sent` trong `data/logs.jsonl`. |
| Retrieval success rate | Chưa ghi nhận | 100% | Tính từ 70 event có `tool_success=true`. |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:** Middleware xóa context cũ, nhận `x-request-id` hoặc sinh ID dạng `req-<8-hex>`, bind vào structlog contextvars và trả lại qua response headers.
- **Các metadata được ghi vào structured log:** `user_id_hash`, `session_id`, `feature`, `model`, `env` được bind trước event `request_received` và kế thừa cho các log cùng request.
- **Cách bảo đảm PII được scrub trước khi ghi:** `scrub_event` chạy trước file renderer, scrub đệ quy mọi string trong event/payload; các pattern email, điện thoại Việt Nam, CCCD và thẻ thanh toán được thay bằng marker redact.
- **Cách kiểm chứng kết quả:** `validate_logs.py` đạt `100/100`; log runtime có đủ context và email mẫu trong `message_preview` hiển thị dạng `[REDACTED_EMAIL]`.

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Langfuse project `day13-k4-l3b-2A202602558` hiển thị 10 root traces do workload của repo tạo.
- **Cấu trúc root/retrieval/generation observations:** Root `lab-agent-run` chứa hai child observations `retrieval` (`retriever`) và `llm-generation` (`generation`). Generation ghi model, input/output tokens và cost; input/output chỉ là preview đã scrub.
- **Cách nối trace với log:** `correlation_id` được truyền trong metadata của root và hai child observations, đồng thời xuất hiện trong structured log.
- **Prompt name:** `day13-chat`.
- **Version/label baseline:** Version 1 của `day13-chat`, labels `baseline` và `production`.
- **Version/label candidate:** Version 2 của `day13-chat`, label `candidate` (đối chiếu lại ảnh `04-prompt-versioning.png`).
- **Trace ID của mỗi version:** Version 1: `21022d4b6bbfd46a95d62ad8fe1c6cc1`; Version 2: `9db78d1053719a592d8e6d95e1bb0d8b`.
- **Cách promote và rollback `production`:** Promote label `production` sang version 2, chạy workload xác nhận trace, sau đó chuyển label về version 1; cần bổ sung trace IDs và ảnh trước/sau vào evidence.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** `latency`, `traffic`, `errors`, `cost`, `tokens`, `quality`; validator đạt `6/6`.
- **SLO và lý do chọn:** SLO `99.5%` request thành công với latency không quá `3000ms` trong cửa sổ 28 ngày, phù hợp với trải nghiệm phản hồi của API.
- **Cách tính error budget:** `100% - 99.5% = 0.5%`; với 10,000 request, tối đa 50 request được phép không đạt SLO.
- **Ba alert và runbook tương ứng:** `HighLatencyP95` (`p95 > 3000ms` trong 5 phút), `HighErrorRate` (`error_rate > 2%` trong 5 phút) và `LowRetrievalSuccess` (`retrieval success < 90%` trong 10 phút), được định nghĩa trong [`config/alert_rules.yaml`](../config/alert_rules.yaml) và [`docs/alerts.md`](../docs/alerts.md).

> Ví dụ cách viết error budget: "SLO 99.5% trong 28 ngày nghĩa là error budget 0.5%. Nếu workload có 10,000 request thì tối đa 50 request được phép lỗi hoặc chậm hơn ngưỡng SLO."

## 7. Điều tra challenge

- **Challenge ID:** `day13-k4-l3b-monitoring-llmops-v1`.
- **Khoảng thời gian điều tra:** `2026-09-30T05:13:06Z`–`2026-09-30T05:13:17Z` UTC.
- **Triệu chứng từ metrics:** Incident `rag_slow` làm latency P95 tăng lên `2653ms`, P99 `2654ms`; request bình thường có P50 khoảng `152ms`.
- **Log line và correlation ID liên quan:** Các request bị ảnh hưởng gồm `req-20b088c1`, `req-6c452f14`, `req-a358411f`, `req-0dc903fa` và `req-8a057b7e`; log `response_sent` có latency `2652`–`2654ms`, `tool_name=retrieval`, `tool_success=true`.
- **Trace ID và span gây ảnh hưởng:** `51b7ea2406a2a39f55abe03c5fb3217b`; trace tương ứng có span `retrieval` bị chậm bất thường.
- **Root cause:** Practice/challenge incident `rag_slow` làm retrieval chậm khoảng 2.5 giây, khiến toàn request vượt mức latency thông thường.
- **Fix action:** Đã tắt incident bằng `scripts/inject_incident.py --base-url http://127.0.0.1:8001 --disable`; cần kiểm tra lại latency sau khi phục hồi retrieval.
- **Preventive measure:** Giữ alert `HighLatencyP95`, theo dõi retrieval span bằng trace, đặt timeout/fallback cho vector store và kiểm tra correlation ID trong runbook.

> Gợi ý cách viết ngắn, không thay cho evidence thực tế: "Metric cho thấy `[latency/error/cost/quality]` bất thường trong `[khoảng thời gian]`. Log line `[event]` có `correlation_id=[...]` đại diện cho request bị ảnh hưởng. Trace cùng `correlation_id` cho thấy span `[retrieval/generation/prompt/tool]` có dấu hiệu `[chậm/lỗi/token tăng]`. Root cause là `[nguyên nhân suy ra từ evidence]`. Fix action là `[hành động khôi phục]`; preventive measure là `[alert/runbook/test/guardrail để ngăn tái diễn]`."

## 8. Giải thích và tự đánh giá

- **Một quyết định kỹ thuật quan trọng và lý do:** Dùng child observations `retrieval` và `llm-generation` của Langfuse SDK v4 để tách thời gian, trạng thái, token và cost của từng bước.
- **Một lỗi/blocker đã gặp:** Port `8000` đã được service khác sử dụng; bổ sung `--base-url` cho `load_test.py` và chạy lab trên port `8001`.
- **Cách tìm nguyên nhân và xử lý:** Kiểm tra `/health`, xác định process giữ port, khởi động đúng một API instance và chạy workload vào `http://127.0.0.1:8001`.
- **Cách hiểu luồng Metrics → Logs → Traces:** Metrics khoanh vùng triệu chứng; log chọn request qua `correlation_id`; trace cho biết retrieval hay generation là span gây chậm/lỗi; sau đó mới kết luận root cause.
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** Prompt version giúp truy nguyên thay đổi; token/cost đo tác động chi phí; SLO định lượng mức phục vụ; rollback đưa production về version ổn định khi candidate gây regression.
- **Điều quan trọng nhất đã học:** Structured logs, traces và dashboard chỉ hữu ích khi nối được cùng request bằng correlation ID.
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** CP3 đã điều tra xong; còn cần bổ sung trace IDs của prompt v1/v2, ảnh prompt rollback, ảnh metadata và dashboard runtime trước khi nộp.

## 9. Checklist trước khi nộp

- [ ] Kết quả và evidence thuộc commit SHA cuối.
- [ ] Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- [ ] Có đủ evidence theo `docs/SCREENSHOT_GUIDE.md`.
- [ ] Incident evidence nối đúng metric → log → trace.
- [ ] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- [ ] Repository chạy lại được theo README.
- [ ] Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- [ ] URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
