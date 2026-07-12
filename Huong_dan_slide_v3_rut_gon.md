# HƯỚNG DẪN NỘI DUNG TỪNG SLIDE (V3 — RÚT GỌN 18 SLIDES)

**Đề tài:** THỬ NGHIỆM GOOGLE CHRONICLE: ỨNG DỤNG AI TRONG GIÁM SÁT VÀ PHẢN ỨNG AN NINH TRÊN NỀN TẢNG GOOGLE CLOUD

**18 slides chính + backup | ~12 phút trình bày + 3 phút demo = 15 phút**

## ⚠️ THAY ĐỔI SO VỚI V2

- GỘP 3 cặp: Slide 5+6 → Slide 5 mới | Slide 10+11 → Slide 9 mới | Slide 14+15 → Slide 12 mới | Slide 22+23 → Slide 17 mới
- CẮT 2 slide → backup: Slide 19 (Chi phí), Slide 20 (Mapping)
- Tổng: 24 → 18 slides

## Phân chia người trình bày

- **NGƯỜI A** — Slides 1–9 (~5 phút): Mở đầu + Lý thuyết + Kiến trúc + Detection
- **NGƯỜI B** — Slides 10–18 + Demo (~10 phút): AI + Remediation + Kết quả + Kết luận + Demo

⚠️ Người A kết thúc bằng câu chuyển giao → Người B tiếp nối. Cả 2 đều đứng trước hội đồng.

---

## Phần 1: Mở đầu (Slides 1–4, ~2 phút)

### Slide 1 — Trang bìa
👤 NGƯỜI A

Nội dung: Tên đề tài, Họ tên 2 SV + MSSV, GVHD: ThS. Đỗ Hoàng Cường, Trường ĐH KHTN – ĐHQG TP.HCM | Khoa CNTT | 2026

🎤 "Kính chào Hội đồng. Em là [Tên A] cùng [Tên B], xin trình bày khóa luận với đề tài THỬ NGHIỆM GOOGLE CHRONICLE: ỨNG DỤNG AI TRONG GIÁM SÁT VÀ PHẢN ỨNG AN NINH TRÊN NỀN TẢNG GOOGLE CLOUD."

### Slide 2 — Nội dung trình bày (Agenda)
👤 NGƯỜI A

- 01 — Bối cảnh & Mục tiêu (Slide 3–4)
- 02 — Cơ sở lý thuyết (Slide 5–6)
- **03 — Thiết kế & Triển khai (Slide 7–13)** ← highlight phần chính
- 04 — Kết quả & Đánh giá (Slide 14–15)
- 05 — Kết luận & Hướng phát triển (Slide 16–17)

🎤 "Bài trình bày gồm 5 phần. [Tên A] sẽ trình bày phần tổng quan và kiến trúc, [Tên B] sẽ trình bày phần kỹ thuật chi tiết, demo và kết quả."

### Slide 3 — Bối cảnh & Vấn đề
👤 NGƯỜI A

3 ô thống kê: Alert fatigue | MTTR cao (IBM 2025) | Chi phí SIEM truyền thống

🎤 "Theo ISC2 năm 2025, toàn cầu thiếu 4.8 triệu chuyên gia an ninh. IBM cho thấy mỗi vụ rò rỉ gây thiệt hại gần 5 triệu đô. Đó là lý do em thực hiện đề tài này."

### Slide 4 — Mục tiêu & Phạm vi
👤 NGƯỜI A

- Cột trái — Mục tiêu (Lý thuyết + Thực tiễn)
- Cột phải — Phạm vi (Đã làm ✅ / Không làm ✗)

🎤 "Về lý thuyết, em nghiên cứu hạn chế của SIEM/SOC truyền thống, tìm hiểu SOAR và kiến trúc Cloud-Native của Google Chronicle. Về thực tiễn, em xây dựng Lab trên GCP, giả lập credential theft, tích hợp AI kép và thử nghiệm YARA-L trên BigQuery. Phạm vi giới hạn ở 18 kịch bản đã kiểm chứng."

---

## Phần 2: Cơ sở lý thuyết (Slides 5–6, ~1.5 phút)

### Slide 5 — Nền tảng lý thuyết ⭐ GỘP (Slide 5+6 cũ)
👤 NGƯỜI A

**Cột trái (xanh) — SOC & SOAR:**
- SOC 3 trụ cột: Con người · Quy trình · Công nghệ
- Thách thức: Alert fatigue, thiếu nhân sự
- → SOAR giải quyết: Orchestration + Automation + Response

**Cột phải (vàng) — Google SecOps (Chronicle):**
- 3 thành phần cốt lõi: SIEM+SOAR tích hợp, UDM, YARA-L 2.0
- Kiến trúc Cloud-Native: Mở rộng vô hạn, lưu trữ nóng 12 tháng

**Dòng dưới:** → Nhóm tự xây pipeline mô phỏng trên GCP (do không có license Chronicle)

🎤 "SOC gồm 3 trụ cột nhưng đang đối mặt alert fatigue và thiếu nhân sự. SOAR giải quyết bằng tự động hóa. Google tích hợp SIEM+SOAR trong Chronicle với UDM chuẩn hóa và YARA-L. Do không có license, nhóm tự xây pipeline tương đương trên GCP."

### Slide 6 — AI trong SOC (= Slide 7 cũ)
👤 NGƯỜI A

- Vai trò: Phân tích log, tương quan ngữ cảnh, tóm tắt sự cố
- Gemini trong Chronicle: Natural language search, tóm tắt cảnh báo
- Rủi ro & Kiểm soát: Hallucination → Fixed JSON schema + Validation + Human-in-the-loop

🎤 "AI phân tích log tự động, tương quan ngữ cảnh và tóm tắt sự cố. Tuy nhiên có rủi ro hallucination — nhóm kiểm soát bằng JSON schema cố định và human-in-the-loop."

⚠️ Hội đồng thường hỏi sâu phần này.

---

## Phần 3a: Kiến trúc & Detection (Slides 7–9, ~3 phút)

### Slide 7 — Mô hình 3 vùng ⭐ (= Slide 8 cũ, Slide quan trọng nhất)
👤 NGƯỜI A

⚠️ Dùng sơ đồ kiến trúc. Hội đồng hỏi nhiều nhất ở đây.

- VÙNG 1 — Attacker Zone (đỏ): gcloud CLI + gsutil + Python SDK
- VÙNG 2 — Victim Zone (xanh dương): Honeypot 55 file, SA victim-employee
- VÙNG 3 — SOC Zone (xanh lá): Pipeline tự thiết kế, 4 lớp chức năng
- Hạ tầng: Terraform IaC — 7 modules

🎤 "Kiến trúc chia 3 vùng. Vùng đỏ Attacker giả lập tin tặc. Vùng xanh Victim chứa Honeypot. Vùng xanh lá là SOC Zone — nhóm tự thiết kế nhằm thay thế Chronicle native, gồm 4 lớp. Toàn bộ hạ tầng quản lý bằng Terraform."

💡 Khi hội đồng hỏi kiến trúc, luôn quay lại slide này.

### Slide 8 — Luồng phòng thủ Event-Driven 5 bước (= Slide 9 cũ)
👤 NGƯỜI A

Flow: Attack → ① Detection → ② Enrichment → ③ AI Triage → ④ Approval → ⑤ Response

🎤 "Luồng 5 bước event-driven: Monitoring phát hiện, Orchestrator làm giàu, AI phân tích, Telegram thông báo admin, và tự động disable SA khi được phê duyệt."

### Slide 9 — Detection & Context Enrichment ⭐ GỘP (Slide 10+11 cũ)
👤 NGƯỜI A (Slide cuối của Người A)

**Phần trên — Detection Event-Driven:**
- Log-based Metric → Alert Policy → Pub/Sub → Cloud Function
- Ngưỡng: >25 objects.get / 60s theo principalEmail

**Phần dưới — Context Enrichment 4 lớp:**

| Lớp | Thu được | Mục đích |
|---|---|---|
| 1 | callerIp + userAgent | Nguồn gốc truy cập |
| 2 | Country, ISP | Phát hiện IP nước ngoài |
| 3 | gsutil/Python SDK | Tự động vs thủ công |
| 4 | Giờ HC/ngoài giờ | Truy cập bất thường |

🔴 CRITICAL: IP nước ngoài + gsutil + ngoài giờ | 🟡 HIGH: IP VN + giờ HC

🎤 "Detection dùng event-driven: Metric đếm >25 lần tải/60s thì trigger Cloud Function. Function enrichment 4 lớp: IP, địa lý, công cụ, thời gian. IP nước ngoài + gsutil + ngoài giờ → CRITICAL."

📌 CÂU CHUYỂN GIAO (Người A → Người B):
🎤 "Vừa rồi em đã trình bày kiến trúc tổng thể và cơ chế phát hiện. Bây giờ [Tên B] sẽ trình bày chi tiết về AI Triage Pipeline, cơ chế phản ứng tự động và kết quả đánh giá."

⚠️ Chuyển giao mượt mà, không im lặng đổi chỗ. Người B bước lên ngay.

---

## NGƯỜI B — Kỹ thuật chi tiết & Kết quả (Slides 10–18 + Demo)

## Phần 3b: AI, Remediation & IaC (Slides 10–12, ~2 phút)

### Slide 10 — AI Triage Pipeline (= Slide 12 cũ)
👤 NGƯỜI B (Slide đầu tiên của Người B)

- Flow: Alert + Enriched Context → Gemini 2.5 Flash → Structured JSON → Fallback: OpenAI
- Ví dụ JSON output: {"severity":"HIGH", "confidence":0.9, "reason":"IP Vietnam + gcloud-python + off-hours"}
- Deterministic prompt: Fixed JSON schema → kiểm soát output
- AI xử lý ~10 giây (3.8% tổng T_response)

🎤 "Em là [Tên B], tiếp nối phần kỹ thuật chi tiết. AI Triage dùng Gemini 2.5 Flash, fallback OpenAI. Prompt deterministic với JSON schema cố định. Ví dụ: IP Việt Nam + gcloud-python + ngoài giờ → HIGH, confidence 0.9."

⚠️ Hội đồng sẽ hỏi: Tại sao 2 engine? Fixed JSON có hạn chế gì? AI hallucinate?

### Slide 11 — Human-in-the-Loop & Remediation (= Slide 13 cũ)
👤 NGƯỜI B

- Phê duyệt Telegram: HMAC-SHA256 signature, link hết hạn 1h, one-time-use guard
- Khi Approve: Disable SA (IAM API) → SCC Finding (MITRE mapping) → Audit trail
- Pipeline phản hồi ~10-15 giây

🎤 "Hệ thống yêu cầu admin phê duyệt qua Telegram. Link ký HMAC-SHA256, hết hạn 1 giờ, chỉ dùng 1 lần. Khi approve: disable SA ngay lập tức, tạo SCC Finding với MITRE ATT&CK mapping."

### Slide 12 — YARA-L Rules & Infrastructure as Code ⭐ GỘP (Slide 14+15 cũ)
👤 NGƯỜI B

**Cột trái (xanh) — YARA-L trên BigQuery:**

| Rule | Phát hiện | Ngưỡng |
|---|---|---|
| Mass Download | Tải nhiều file | >25/60s |
| Off-Hours Access | Ngoài giờ HC | UTC+7 ngoài 8h-17h |
| Suspicious Tool | Công cụ scripted | gsutil/Python SDK |

- YARA-L → SQL vì không có Chronicle license
- ✅ 55 lượt tải khớp chính xác

**Cột phải (vàng) — Terraform IaC:**
- 7 modules: iam, network, storage, logging_detection, functions, scc, monitoring
- ⚠️ Honeypot: null_resource + gsutil (tránh false positive)

🎤 "3 detection rules YARA-L chuyển sang SQL chạy trên BigQuery, khớp chính xác 55 lượt. Toàn bộ hạ tầng bằng Terraform 7 modules. Honeypot dùng null_resource tránh false positive trong Terraform state."

---

## 🎬 DEMO (~3 phút) — Đặt sau Slide 12

👤 NGƯỜI B demo, NGƯỜI A hỗ trợ điều khiển slide

Kịch bản:
1. Chạy attack_simulation.py → 55 file được tải
2. Chờ Cloud Monitoring trigger (hoặc dùng video quay sẵn phần chờ)
3. Xem Telegram notification → nhấn Approve
4. Xem kết quả: SA bị disable + SCC Finding
5. (Tùy chọn) Xem BigQuery YARA-L query

🎤 "Bây giờ em xin demo trực tiếp hệ thống. Em sẽ giả lập tấn công credential theft và cho Hội đồng thấy toàn bộ pipeline từ phát hiện đến phản ứng."

---

## Phần 4: Kết quả & Đánh giá (Slides 13–15, ~2 phút)

### Slide 13 — Sơ đồ luồng End-to-End (= Slide 16 cũ)
👤 NGƯỜI B

- 7 ô: Attack → Detect → Enrich → Triage → Notify → Approve → Respond
- T_response ≈ 265 giây (~4.4 phút) — benchmark ngành: 24-48 giờ (SANS 2024)
- Bảng thống kê: Mean 265s | Std 17.8s | Min 236s | Max 292s

🎤 "Tổng thời gian phát hiện là 265 giây cho kịch bản credential theft. Cửa sổ metric chiếm 96% thời gian. So với benchmark ngành 24-48 giờ, kết quả cho thấy tiềm năng của kiến trúc event-driven."

### Slide 14 — Ma trận kiểm thử (= Slide 17 cũ)
👤 NGƯỜI B

- 4 tổ hợp tiêu biểu (trong 18 kịch bản: 3 IP × 3 Tool × 2 Time)
- 100% phát hiện (18/18). IP nước ngoài nhận CRITICAL 83%

🎤 "18 kịch bản từ 3 biến: IP, công cụ, thời gian. 100% phát hiện. IP nước ngoài nhận CRITICAL 83% trong 12 kịch bản tiêu biểu."

### Slide 15 — Đánh giá hiệu năng (= Slide 18 cũ)
👤 NGƯỜI B

| Chỉ số | SOC truyền thống | SOC Serverless | Phạm vi đề tài |
|---|---|---|---|
| MTTD | 24-48 giờ* | ~4.4 phút | 1 kịch bản |
| MTTR | Hàng giờ-ngày | ~10 giây | 1 hành động |

* Benchmark SANS 2024

🎤 "Trong phạm vi credential theft, MTTD đạt 4.4 phút và MTTR 10 giây. So với benchmark 24-48 giờ, kết quả cho thấy tiềm năng lớn, nhưng cần mở rộng kịch bản để đánh giá toàn diện."

---

## Phần 5: Kết luận (Slides 16–18, ~1.5 phút)

### Slide 16 — Kết luận (= Slide 21 cũ)
👤 NGƯỜI B

**Về lý thuyết:** Nghiên cứu có hệ thống SOC, SIEM, SOAR, Chronicle, UDM, YARA-L

**Về thực tiễn (18 kịch bản):**
- MTTD ~4.4 phút, MTTR ~10 giây
- Context Enrichment 4 lớp → AI phân biệt rủi ro theo ngữ cảnh
- 3 luật YARA-L kiểm chứng BigQuery — khớp chính xác

**Kết luận cốt lõi:**
- Khẳng định tính khả thi Cloud-Native + AI trong SOC tự động
- Phù hợp tổ chức vừa & nhỏ — chi phí tối ưu nhờ serverless pay-per-use
- Toàn bộ hạ tầng IaC → tái tạo hoàn toàn từ mã nguồn

🎤 "Tóm lại, về lý thuyết đề tài nghiên cứu có hệ thống kiến trúc SOC hiện đại. Về thực tiễn, MTTD 4.4 phút và MTTR 10 giây trong phạm vi credential theft. Kết quả cho thấy tiềm năng của kiến trúc Cloud-Native + AI, phù hợp tổ chức vừa nhỏ với chi phí serverless."

### Slide 17 — Hạn chế & Hướng phát triển ⭐ GỘP (Slide 22+23 cũ)
👤 NGƯỜI B

**Cột trái (cam nhạt) — ⚠️ Hạn chế:**
- Không có Chronicle license
- Chỉ test credential theft
- Độ trễ 4 phút (metric window chiếm 96%)
- LLM non-deterministic
- Không có module UDM riêng biệt

**Cột phải (xanh lá nhạt) — 🚀 Hướng phát triển:**
- Real-time detection (Log Sink → MTTD <30s)
- Mở rộng: privilege escalation, lateral movement
- Triển khai YARA-L trên Chronicle khi có license
- Fine-tune AI model

🎤 "Về hạn chế: không có license Chronicle, phạm vi hẹp, độ trễ do metric window. Về hướng phát triển: chuyển sang real-time detection có thể giảm MTTD xuống dưới 30 giây, mở rộng kịch bản và triển khai trực tiếp trên Chronicle khi có license."

💡 Thẳng thắn — hội đồng đánh giá cao sự trung thực.

### Slide 18 — Cảm ơn Hội đồng (= Slide 24 cũ)
👤 CẢ 2 NGƯỜI (Cùng đứng lên cảm ơn)

🎤 "Chúng em xin cảm ơn Hội đồng đã lắng nghe. Kính mời quý Thầy Cô đặt câu hỏi."

---

## Backup Slides (chỉ dùng khi hội đồng hỏi)

💡 KHÔNG trình bày. Chỉ lật đến khi cần.

| Backup | Nội dung | Dùng khi hỏi |
|---|---|---|
| B1 | Chi phí vận hành (Slide 19 cũ) | "Chi phí so với SOC thương mại?" |
| B2 | Mapping Lý thuyết → Thực tiễn (Slide 20 cũ) | "Lý thuyết liên hệ thực tiễn?" |
| B3 | So sánh Event-driven vs SQL Polling | "Tại sao không dùng BigQuery polling?" |
| B4 | Screenshot BigQuery YARA-L results | "Kết quả YARA-L cụ thể?" |
| B5 | Code YARA-L ↔ SQL song song | "Chuyển đổi YARA-L thế nào?" |
| B6 | Sơ đồ overview draw.io chi tiết | "Kiến trúc chi tiết hơn?" |
| B7 | Dual-Pipeline (Pipeline B) | "Có thể giảm MTTD không?" |

---

## Phân chia trả lời câu hỏi Hội đồng

### Người A trả lời (Kiến trúc & Lý thuyết)
- "Tại sao serverless?" → Pay-per-use, auto-scale, không maintain
- "Chronicle vs Splunk/ELK?" → Cloud-native, UDM chuẩn hóa, YARA-L mạnh hơn regex
- "Pub/Sub bị nghẽn?" → Retry + dead-letter queue. Lab traffic thấp
- "Cloud Functions vs Cloud Run?" → CF native Pub/Sub trigger
- "SOAR khác SIEM?" → SIEM = thu thập+phát hiện, SOAR = điều phối+tự động+phản ứng
- "Attacker biết honeypot?" → Defense-in-Depth: Audit Logs vẫn detect

### Người B trả lời (AI & Bảo mật & Đánh giá)
- "Gemini hallucinate?" → Fixed JSON schema + validate 7 trường. Sai → reject → fallback
- "Tại sao dual-engine?" → Availability: Gemini lỗi → OpenAI tiếp quản
- "AI bị adversarial attack?" → Deterministic pipeline params
- "HMAC-SHA256 đủ an toàn?" → Chuẩn công nghiệp + TTL 1h + one-time-use
- "Test bao nhiêu mẫu?" → 18 kịch bản, 100% phát hiện, 0 false negative
- "So với SOC thương mại?" → Thương mại nhiều tính năng hơn, đề tài chứng minh feasibility

---

## Bảng ánh xạ Slide cũ → mới

| Slide cũ | Slide mới | Ghi chú |
|---|---|---|
| 1 | 1 | Giữ nguyên |
| 2 | 2 | Cập nhật agenda |
| 3 | 3 | Giữ nguyên |
| 4 | 4 | Giữ nguyên |
| 5+6 | **5** | **GỘP** |
| 7 | 6 | Giữ nguyên |
| 8 | 7 | Giữ nguyên |
| 9 | 8 | Giữ nguyên |
| 10+11 | **9** | **GỘP** |
| 12 | 10 | Giữ nguyên |
| 13 | 11 | Giữ nguyên |
| 14+15 | **12** | **GỘP** |
| 16 | 13 | Giữ nguyên |
| 17 | 14 | Giữ nguyên |
| 18 | 15 | Giữ nguyên |
| 19 | — | → Backup B1 |
| 20 | — | → Backup B2 |
| 21 | 16 | Giữ nguyên |
| 22+23 | **17** | **GỘP** |
| 24 | 18 | Giữ nguyên |

---

## Tổng thời gian

| Phần | Slides | Thời gian |
|---|---|---|
| Mở đầu | 1–4 | ~2 phút |
| Lý thuyết | 5–6 | ~1.5 phút |
| Kiến trúc & Kỹ thuật | 7–12 | ~4 phút |
| 🎬 DEMO | — | ~3 phút |
| Kết quả | 13–15 | ~2 phút |
| Kết luận | 16–18 | ~1.5 phút |
| **TỔNG** | **18 slides + demo** | **~14 phút** |

> Còn ~1 phút dự phòng cho chuyển giao.

## Lưu ý khi trình bày 2 người

- Chuyển giao mượt mà: Người A kết thúc bằng 1 câu giới thiệu Người B
- Đứng cạnh nhau: Cả 2 đều đứng, người không nói điều khiển slide
- Eye contact: Người nói nhìn hội đồng
- Phối hợp Q&A: Câu thuộc phần người kia → nhường
- Tập dượt: Ít nhất 3 lần, bấm giờ. Chuyển giao ≤ 15 giây
- Slide 7/8 là neo: Khi trả lời, luôn quay lại sơ đồ kiến trúc
