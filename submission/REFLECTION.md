# Day 23 Lab Reflection

> Fill in each section. Grader reads the "What I'd change" paragraph closest.

**Student:** Nguyễn Ngọc Cường
**Submission date:** 2026-05-11
**Lab repo URL:** https://github.com/feedingpet/Day23-Track2-Observability-Lab

---

## 1. Hardware + setup output

Paste output of `python3 00-setup/verify-docker.py`:

```
Docker:        OK  (29.4.2)
Compose v2:    OK  (5.1.3)
RAM available: 3.67 GB (NEED >= 4.0 GB)
Ports free:    BOUND: [8000, 9090, 3000, 3100, 16686, 4317, 4318, 8888]
Report written: D:\VinUni\assignments\Day23-Track2-Observability-Lab\00-setup\setup-report.json
```

---

## 2. Track 02 — Dashboards & Alerts

### 6 essential panels (screenshot)

Drop `submission/screenshots/dashboard-overview.png`.

### Burn-rate panel

Drop `submission/screenshots/slo-burn-rate.png`.

### Alert fire + resolve

| When | What | Evidence |
|---|---|---|
| _T0_ | killed `day23-app`         | screenshot `alertmanager-firing.png` |
| _T0+90s_ | `ServiceDown` fired   | screenshot `slack-firing.png` |
| _T1_ | restored app              | — |
| _T1+60s_ | alert resolved        | screenshot `slack-resolved.png` |

### One thing surprised me about Prometheus / Grafana

Tôi rất ngạc nhiên trước khả năng kết nối và gửi cảnh báo tự động từ hệ thống tới Slack một cách liền mạch qua Webhook. Việc nhận được thông báo ngay lập tức trên điện thoại khi dịch vụ gặp sự cố giúp quy trình giám sát trở nên chủ động và hiệu quả hơn rất nhiều so với việc phải kiểm tra thủ công

---

## 3. Track 03 — Tracing & Logs

### One trace screenshot from Jaeger

Drop `submission/screenshots/jaeger-trace.png` showing `embed-text → vector-search → generate-tokens` spans.

### Log line correlated to trace

Paste the log line and the trace_id it links to:

```
{"model": "llama3-mock", "input_tokens": 4, "output_tokens": 63, "quality": 0.892, "duration_seconds": 0.2675, "event": "prediction served", "level": "info", "timestamp": "2026-05-11T10:58:15.853491Z", "trace_id": "0b58824d98c775771bebe970f2fd1d73", "span_id": "7c3a456b02bf5457"}
```

### Tail-sampling math

Dựa trên cấu hình của bộ xử lý tail_sampling trong OTel Collector, logic giữ lại Trace như sau:

Trace lỗi (Error): Giữ lại 100% (policy: keep-errors).
Trace chậm (> 2 giây): Giữ lại 100% (policy: keep-slow).
Trace bình thường: Giữ lại 1% (policy: probabilistic-1pct).
Phép tính giả định: Giả sử dịch vụ tạo ra $N$ traces/giây, trong đó có 1% là lỗi và 1% là request chậm. Tỉ lệ dữ liệu được giữ lại ($Fraction$) sẽ là: $Fraction = (1.0 \times 0.01) + (1.0 \times 0.01) + (0.01 \times 0.98)$ $Fraction = 0.01 + 0.01 + 0.0098 = 0.0298$

Kết luận: Hệ thống giữ lại khoảng ~3% tổng lượng Trace, giúp giảm 97% chi phí lưu trữ và băng thông mà vẫn đảm bảo giữ được 100% các dữ liệu quan trọng (lỗi và hiệu năng kém).

---

## 4. Track 04 — Drift Detection

### PSI scores

Paste `04-drift-detection/reports/drift-summary.json`:

{
  "prompt_length": {
    "psi": 3.461,
    "kl": 1.7982,
    "ks_stat": 0.702,
    "ks_pvalue": 0.0,
    "drift": "yes"
  },
  "embedding_norm": {
    "psi": 0.0187,
    "kl": 0.0324,
    "ks_stat": 0.052,
    "ks_pvalue": 0.133853,
    "drift": "no"
  },
  "response_length": {
    "psi": 0.0162,
    "kl": 0.0178,
    "ks_stat": 0.056,
    "ks_pvalue": 0.086899,
    "drift": "no"
  },
  "response_quality": {
    "psi": 8.8486,
    "kl": 13.5011,
    "ks_stat": 0.941,
    "ks_pvalue": 0.0,
    "drift": "yes"
  }
}

### Which test fits which feature?

- Prompt Length (Độ dài prompt): **PSI (Population Stability Index)**

  **Lý do:** Đây là chỉ số lý tưởng để phát hiện sự thay đổi về phân phối độ dài (ví dụ: hệ thống đột nhiên nhận nhiều request dài hơn). PSI nhạy cảm với sự dịch chuyển của phân phối và dễ diễn giải: PSI > 0.25 là "drift đáng kể".
- Embedding Norm (Chuẩn hóa Embedding): **KS Test (Kolmogorov-Smirnov)**

  **Lý do:** Embedding là các vector liên tục. KS test là lựa chọn tiêu chuẩn để so sánh sự khác biệt giữa hai phân phối liên tục. Nó kiểm tra xem liệu hai mẫu (batch hiện tại so với baseline) có được rút ra từ cùng một phân phối hay không.
- Response Length (Độ dài phản hồi): **MMD (Maximum Mean Discrepancy)**

  **Lý do:** Tương tự Response Quality, Response Length cũng là một đặc trưng liên tục. MMD hiệu quả trong việc phát hiện sự thay đổi trong phân phối của các đặc trưng kỹ thuật như độ dài. Nếu độ dài response tăng đột biến (ví dụ do thay đổi mô hình hoặc prompt), MMD sẽ phát hiện ra điều này.
- Response Quality (Chất lượng phản hồi): **KL Divergence (Kullback-Leibler Divergence)**

  **Lý do:** Mặc dù KL Divergence tính toán dựa trên sự khác biệt về phân phối, nó đặc biệt phù hợp ở đây vì "chất lượng" thường được mô hình hóa dưới dạng phân phối xác suất (ví dụ: xác suất các token tiếp theo). KL Divergence đo lường lượng thông tin bị mất khi sử dụng phân phối mới (hiện tại) để xấp xỉ phân phối cũ (baseline), giúp phát hiện sự thay đổi chất lượng một cách định lượng.

---

## 5. Track 05 — Cross-Day Integration

Chỉ số đo lường từ Qdrant (Ngày 19) là khó tích hợp nhất. Lý do là vì Qdrant sử dụng định dạng dữ liệu telemetry riêng biệt, đòi hỏi phải cấu hình chính xác các scrape target trong Prometheus và đảm bảo thông luồng mạng giữa các container để bộ công cụ quan sát có thể truy cập được dữ liệu mà không bị lỗi timeout.

---

## 6. The single change that mattered most

> **Grader reads this closest.** What one thing about your stack design — a metric you added, a label you dropped, a panel you reorganized, an alert threshold you tuned — made the biggest difference between "works" and "useful"? Write 1-2 paragraphs. Connect it to a concept from the deck.

Thay đổi quan trọng nhất với mình là **chuẩn hoá môi trường chạy (venv) + đồng bộ version dependency để drift detection “chạy được” thành “dùng được”**. Ở Track 04, mình gặp tình huống “đã cài Evidently trong venv nhưng script lại báo `evidently not installed`”. Khi kiểm tra kỹ, có 2 vấn đề: (1) đôi lúc terminal dùng nhầm Python hệ thống thay vì `.\venv\Scripts\python.exe`, nên package trong venv không được nhìn thấy; (2) quan trọng hơn, bản Evidently cũ (0.4.x) **không tương thích với `pydantic==2.x`** (ImportError bên trong Evidently), làm cho phần tạo HTML report thất bại và bị hiểu nhầm là “chưa cài”.

Mình xử lý theo hướng “reproducible pipeline”: luôn chạy script bằng đúng interpreter của venv (`.\venv\Scripts\python.exe 04-drift-detection\scripts\drift_detect.py`), sau đó **nâng Evidently lên 0.7.21** (tương thích với pydantic v2) và cập nhật code theo API mới: dùng `from evidently import Report`, `from evidently.presets import DataDriftPreset`, và `snapshot.save_html(...)` thay vì `report.save_html(...)`. Cuối cùng, mình cập nhật `requirements.txt` để đóng băng version, giúp người khác cài lại và chạy ra được cả `drift-summary.json` lẫn `drift-report.html` ngay lần đầu. Liên hệ với nội dung trong deck: observability/monitoring chỉ “hữu ích” khi **đáng tin và tái lập**; nếu toolchain phụ thuộc môi trường mơ hồ hoặc lệch version thì kết quả giám sát (ở đây là drift report) sẽ không ổn định và khó dùng trong vận hành.
