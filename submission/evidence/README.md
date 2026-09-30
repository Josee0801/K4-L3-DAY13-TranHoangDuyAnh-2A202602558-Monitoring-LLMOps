# Evidence cá nhân

Đặt ảnh hoặc output text dùng để chấm vào thư mục này. Danh sách đầy đủ xem tại [docs/SUBMISSION.md](../../docs/SUBMISSION.md).

Ba output text hoặc ảnh validator:

```text
pytest.txt
log-validator.txt
dashboard-validator.txt
```

Các ảnh runtime theo `docs/SCREENSHOT_GUIDE.md`:

```text
04-structured-log.png
05-pii-redaction.png
06-trace-list.png
07-trace-waterfall.png
08-trace-metadata.png
09-prompt-versions.png
10-prompt-rollback.png
11-dashboard-overview.png
12-incident-metric.png
13-incident-log.png
14-incident-trace.png
```

Ảnh log lấy từ terminal/`data/logs.jsonl`; ảnh trace/prompt lấy từ project Langfuse cá nhân; ảnh dashboard lấy từ dashboard runtime. Không mở/chụp trang API Keys và không để lộ secret/PII.

Từ `submission/REPORT.md`, dẫn ảnh bằng đường dẫn tương đối:

```markdown
![Incident trace](evidence/03-incident-trace.png)
```

Không commit secret, API key, PII thô hoặc evidence của học viên/lớp khác.
