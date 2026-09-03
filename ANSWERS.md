# Lab 28 - Reflection và bàn giao

## Kiến trúc và quyền sở hữu

Hệ thống tách các boundary theo trách nhiệm: Envoy sở hữu routing/rate limit;
FastAPI sở hữu HTTP contract; Kafka sở hữu giao nhận at-least-once; Airflow sở
hữu orchestration; Spark/Delta sở hữu transaction và replay-safe merge; Feast
sở hữu online features; Qdrant sở hữu hybrid retrieval; MLflow sở hữu release
alias; vLLM sở hữu inference; Prometheus/Grafana và OpenTelemetry/Jaeger sở hữu
metrics và trace. Sơ đồ nộp kèm nằm tại
`docs/images/lab28-architecture-overview.png`.

## Các lựa chọn kỹ thuật chính

- Kafka producer dùng idempotency key và W3C `traceparent` dạng byte header.
  Header trace bị bỏ hoàn toàn khi không có context, thay vì gửi giá trị rỗng.
- Delta merge source được khử trùng theo `idempotency_key`; bản có
  `(occurred_at, event_id)` lớn nhất thắng. Kết quả được sắp theo key để replay
  cho kết quả xác định.
- Feast request dùng `FEATURE_REFS` từ contract trung tâm, tránh registry và API
  lệch danh sách feature.
- Readiness fail closed khi dependency bắt buộc lỗi, nhưng phân biệt
  `degraded` cho dependency có fallback. Envoy kiểm tra `/ready`, còn `/health`
  chỉ biểu diễn liveness.
- Kafka consumer chờ partition assignment trước khi tính số lần poll rỗng. Nếu
  không, lần chạy đầu có thể kết luận sai rằng topic rỗng trong lúc consumer
  vẫn đang join group.
- API key vLLM chỉ đi qua biến môi trường; không được ghi vào Compose, evidence
  hoặc repository.

## Failure/recovery đã kiểm chứng

- Kafka replay không làm tăng số hàng Delta hoặc số point Qdrant; J2 đạt 9 test.
- Feast/Qdrant outage tạo đúng `degraded` hoặc `not_ready`, sau đó phục hồi mà
  không mất dữ liệu; J4 core đạt 9 test và ba GPU degraded tests đạt.
- Envoy loại API khỏi rotation khi `/ready` trả 503. Cấu hình đặt
  `healthy_panic_threshold: 0` để cluster một upstream không quay lại panic
  routing. Test gateway eviction đạt sau khi sửa active health check.
- MLflow promotion và rollback thay đổi champion alias mà không sửa serving
  code; J3 core và ba GPU tests đều đạt.

## Production gaps

- Compose dùng Kafka một broker và nhiều backend local volume; production cần
  replication, backup, encryption, retention review và disaster recovery.
- Airflow standalone, SimpleAuthManager và Grafana `admin/admin` chỉ phù hợp
  môi trường lab. Production cần identity provider, secret manager và TLS.
- vLLM được chạy tạm trên Kaggle qua Cloudflare Quick Tunnel. Session, URL và
  GPU quota không bền vững; production cần endpoint riêng có authentication,
  private networking, autoscaling và capacity/SLO monitoring.
- LangSmith chưa được xác minh vì không có credential. Local Jaeger đã chứng
  minh required span names, nhưng trace cuối chỉ có ba service emitters; vLLM
  remote chưa export OTLP về collector local.
- Prometheus chưa scrape được vLLM qua endpoint tạm. Không hard-code tunnel URL
  vào desired state.
- Load profile P50/P95/P99 chưa được thu trong một phiên vLLM ổn định.

## Trạng thái xác minh

- Fast suite: 87 passed; Ruff, portability, integration matrix và Kubernetes /
  GitOps manifest validation đều đạt.
- Full integration không GPU/LangSmith: 56 passed.
- J1 từng đạt 15 passed với vLLM 0.26.0 thật và model
  `Qwen/Qwen3-1.7B` trên Kaggle T4.
- GPU subsets đã đạt: J3 3 test, J4 degraded 3 test, gateway eviction 1 test và
  J5 serving trace 1 test.
- Prometheus-vLLM, emitter thứ tư, LangSmith và load profile được báo
  `UNVERIFIED`; không dùng mock hoặc evidence giả để thay thế.

## Đóng góp

Bài được thực hiện cá nhân. Nguyễn Huy Hưng phụ trách cả bốn student tasks,
chạy và chẩn đoán integration journeys, cấu hình gateway/readiness, thu evidence
và chuẩn bị phần trình bày.
