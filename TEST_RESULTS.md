# Kết quả kiểm thử

Các kết quả dưới đây được ghi lại từ các lần chạy ngày 2026-09-03. Những gate
không hoàn thành được ghi rõ thay vì coi `skipped` là `passed`.

| Phạm vi | Kết quả |
|---|---:|
| `pytest starter-tests tests -q` | 87 passed |
| Ruff | All checks passed |
| Integration matrix | 245 checks passed |
| Portability | passed |
| Kubernetes/GitOps manifests | passed |
| Integration, không GPU/LangSmith | 56 passed, 16 deselected |
| J1 với vLLM thật | 15 passed |
| J2 idempotent replay | 9 passed |
| J3 core | 6 passed, 3 GPU skipped |
| J3 GPU subset | 3 passed |
| J4 core | 9 passed, 4 GPU skipped |
| J4 GPU degraded subset | 3 passed |
| J4 gateway eviction | 1 passed |
| J5 GPU serving trace | 1 passed |

## Chưa xác minh

- Prometheus scrape target cho vLLM chạy từ Kaggle.
- Tối thiểu bốn service emit span trên cùng trace (đã thấy ba service và đủ
  required span names).
- LangSmith export vì không có `LANGSMITH_API_KEY`.
- Load profile P50/P95/P99 trong một phiên vLLM ổn định.

Chi tiết live evidence nằm trong gói `evidence/`. File
`evidence/ip07-vllm-identity.json` là snapshot âm sau khi Kaggle session kết
thúc; bằng chứng dương tương ứng là lần chạy J1 15/15 ở trên. Không diễn giải
snapshot âm này thành trạng thái ready hiện tại.
