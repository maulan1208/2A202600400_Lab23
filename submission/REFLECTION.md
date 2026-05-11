# Day 23 Lab Reflection

> Fill in each section. Grader reads the "What I'd change" paragraph closest.

**Student:** Nguyễn Mậu Lân
**Submission date:** 12/08/2003
**Lab repo URL:** https://github.com/maulan1208/2A202600400_Lab23

---

## 1. Hardware + setup output

Paste output of `python3 00-setup/verify-docker.py`:

```
Docker:        OK  (29.4.0)
Compose v2:    OK  (5.1.1)
RAM available: 7.41 GB (OK)
Ports free:    BOUND: [8000, 9090, 9093, 3000, 3100, 16686, 4317, 4318, 8888]
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

Về tầm quan trọng của các Labels. Một sai sót nhỏ trong việc đặt tên model có thể khiến toàn bộ Dashboard báo "No data" mặc dù ứng dụng vẫn đang chạy. Điều này cho thấy tính kỷ luật trong việc đặt tên metadata là yếu tố sống còn của một Data Architect.

---

## 3. Track 03 — Tracing & Logs

### One trace screenshot from Jaeger

Drop `submission/screenshots/jaeger-trace.png` showing `embed-text → vector-search → generate-tokens` spans.

### Log line correlated to trace

Paste the log line and the trace_id it links to:

```
2026-05-11 22:40:05.706 INFO [inference-api] [trace_id=b92fc440b7ef597e864c5688b665e69a span_id=eac46b689b8a537d] Prediction completed for model=llama3-mock status=ok duration_ms=243.53
```

### Tail-sampling math

If your service produced N traces/sec, what fraction did the policy keep? Show the calculation.

---

## 4. Track 04 — Drift Detection

### PSI scores

Paste `04-drift-detection/reports/drift-summary.json`:

```json
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
```

### Which test fits which feature?

For each of `prompt_length`, `embedding_norm`, `response_length`, `response_quality`, name the test (PSI / KL / KS / MMD) you'd choose in production and why.

PSI (Population Stability Index): Chọn cho prompt_length và response_quality vì nó đo lường sự thay đổi của toàn bộ quần thể dữ liệu theo thời gian, rất tốt cho việc phát hiện hành vi người dùng thay đổi.

KS Test: Chọn cho embedding_norm vì nó nhạy cảm với sự thay đổi về vị trí và hình dạng của phân phối, giúp phát hiện những sai lệch nhỏ trong không gian vector.

KL Divergence: Chọn cho response_quality để đo khoảng cách thông tin giữa hai phân phối, giúp xác định mức độ "xa lạ" của dữ liệu Production so với dữ liệu Train.

---

## 5. Track 05 — Cross-Day Integration

### Which prior-day metric was hardest to expose? Why?

_(2-3 sentences. If you didn't have prior days running, write about which one would be hardest based on the integration scripts.)_
Chỉ số từ Day 16 (AWS Infrastructure) là khó tích hợp nhất. Việc phải kết nối thông tin từ các EC2 instance/CloudWatch về một hệ thống Prometheus tập trung trong Docker yêu cầu cấu hình Network (Bastion Host/VPN) và xác thực IAM rất phức tạp để đảm bảo dữ liệu không bị lộ ra ngoài.

---

## 6. The single change that mattered most

> **Grader reads this closest.** What one thing about your stack design — a metric you added, a label you dropped, a panel you reorganized, an alert threshold you tuned — made the biggest difference between "works" and "useful"? Write 1-2 paragraphs. Connect it to a concept from the deck.

Thay đổi quan trọng nhất trong thiết kế stack của tôi là việc điều chỉnh ngưỡng cảnh báo (Alert Threshold) của PSI và cấu hình Route trong Alertmanager. Ban đầu, các cảnh báo bị gửi tràn lan (alert fatigue). Việc đặt ngưỡng PSI > 0.25 cho response_quality giúp đội ngũ vận hành chỉ nhận thông báo khi chất lượng model thực sự có vấn đề nghiêm trọng, thay vì các biến động nhiễu nhỏ.

Ngoài ra, việc tích hợp Data Drift trực tiếp vào vòng lặp Observability (thay vì chỉ xem Accuracy) giúp chúng ta biết tại sao model hỏng trước khi người dùng phàn nàn. Theo khái niệm "Shift-Left Testing", việc phát hiện Drift sớm chính là chìa khóa để bảo vệ uy tín của các hệ thống như XanhSM, nơi mà một câu trả lời sai về chính sách có thể gây thiệt hại tài chính ngay lập tức.
