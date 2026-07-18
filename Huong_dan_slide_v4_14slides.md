# HƯỚNG DẪN NỘI DUNG TỪNG SLIDE (V4 — 14 SLIDES + BACKUP)

**Đề tài:** THỬ NGHIỆM GOOGLE CHRONICLE: ỨNG DỤNG AI TRONG GIÁM SÁT VÀ PHẢN ỨNG AN NINH TRÊN NỀN TẢNG GOOGLE CLOUD

**14 slides chính + 7 backup | ~11 phút trình bày + 3 phút demo = ~14 phút (dư 1 phút buffer)**

## ⚠️ THAY ĐỔI SO VỚI V3 (theo yêu cầu GVPB)

| Yêu cầu GVPB | Cách xử lý |
|---|---|
| Làm rõ đóng góp | Restructure Slide Kết luận → 4 đóng góp chính |
| Ưu tiên số liệu hiệu năng | Thêm Bảng 4.5 (bottleneck) + Bảng 4.6 (enrichment impact) |
| Nêu hạn chế rõ hơn | Bổ sung chi tiết + lý giải tại sao |
| Làm rõ hệ thống | Thêm annotation vào sơ đồ 3 vùng |
| Giảm ~15 slide | 18 → 14 slide |

**Slide bị cắt:** Agenda (nói miệng) | AI trong SOC (→ backup) | YARA-L+Terraform (→ backup)
**Slide bị gộp:** Ma trận + Đánh giá → 1 slide

## Phân chia người trình bày

- **NGƯỜI A** — Slides 1–7 (~4.5 phút): Mở đầu + Lý thuyết + Kiến trúc + Detection
- **NGƯỜI B** — Slides 8–14 + Demo (~10.5 phút): AI + Remediation + Demo + Kết quả + Kết luận

---

## Slide 1 — Trang bìa
👤 NGƯỜI A

- Tên đề tài tiếng Việt (font chính)
- Tên tiếng Anh (font nhỏ hơn, italic, ngay dưới tên TV):
  *"Experimenting with Google Chronicle: Applying AI in Security Monitoring and Incident Response on Google Cloud Platform"*
- Họ tên + MSSV 2 sinh viên
- GVHD: ThS. Đỗ Hoàng Cường
- Trường ĐH KHTN – ĐHQG TP.HCM | Khoa CNTT | 2026

🎤 "Kính chào Hội đồng. Em là [Tên A] cùng [Tên B], xin trình bày khóa luận..."

📌 **Không có Slide Agenda** — nói miệng:
🎤 "Bài trình bày gồm 4 phần: Bối cảnh & Mục tiêu, Thiết kế hệ thống, Kết quả đánh giá, và Kết luận."

---

## Slide 2 — Bối cảnh & Vấn đề (= Slide 3 cũ)
👤 NGƯỜI A

3 ô thống kê (giữ nguyên):
- 4.8 triệu người — Thiếu hụt chuyên gia an ninh (ISC2 2025)
- 181 ngày — Thời gian phát hiện TB (IBM 2025)
- $4.88M — Chi phí TB mỗi vụ rò rỉ (IBM Cost of Data Breach)

3 bullet:
- Alert fatigue → SOC analyst quá tải
- MTTR cao → hàng giờ - ngày
- Chi phí SIEM truyền thống đắt

→ "Liệu kiến trúc Cloud-Native + AI có thể giải quyết?"

🎤 "Theo ISC2 2025, thiếu 4.8 triệu chuyên gia. IBM cho thấy trung bình mất 181 ngày phát hiện sự cố, thiệt hại gần 5 triệu đô. SOC truyền thống đang quá tải."

---

## Slide 3 — Mục tiêu & Phạm vi (= Slide 4 cũ)
👤 NGƯỜI A

Giữ nguyên nội dung. Đảm bảo có:
- Cột trái: Mục tiêu LT + TT
- Cột phải: Phạm vi ✅ Đã làm / ✗ Không làm

🎤 Giữ nguyên lời thoại.

---

## Slide 4 — Nền tảng lý thuyết (= Slide 5 cũ + 1 ý từ Slide 6)
👤 NGƯỜI A

**Giữ nguyên bố cục 2 cột:**
- Cột trái (xanh): SOC & SOAR
- Cột phải (vàng): Google SecOps (Chronicle)

**THÊM MỚI** — 1 dòng cuối slide (dưới "→ Nhóm tự xây pipeline"):

> **AI trong SOC:** Gemini phân tích log + tóm tắt sự cố | Kiểm soát: Fixed JSON schema + Human-in-the-loop

⚠️ Chỉ thêm 1 dòng. KHÔNG cần slide AI riêng.

🎤 "SOC gồm 3 trụ cột nhưng đang đối mặt alert fatigue. SOAR giải quyết bằng tự động hóa. Google tích hợp trong Chronicle. Do không có license, nhóm tự xây pipeline. Về AI, nhóm dùng Gemini phân tích log với JSON schema cố định và human-in-the-loop để kiểm soát."

---

## Slide 5 — Mô hình kiến trúc 3 vùng ⭐ (= Slide 7 cũ)
👤 NGƯỜI A

⚠️ **CẢI THIỆN theo yêu cầu "Làm rõ hệ thống":**

Thêm **3 label rõ ràng** trên sơ đồ:
1. 🔴 **VÙNG 1 — Attacker Zone:** gcloud CLI, gsutil, Python SDK
2. 🔵 **VÙNG 2 — Victim Zone:** Honeypot bucket (55 file), SA victim-employee
3. 🟢 **VÙNG 3 — SOC Zone:** Pipeline tự thiết kế — 4 lớp (Detection → Enrichment → AI → Response)

Thêm **mũi tên** chỉ hướng luồng dữ liệu: Attack → Audit Log → Detection → Response

Thêm **1 dòng dưới sơ đồ:**
> Toàn bộ hạ tầng: **Terraform IaC — 7 modules** | Tái tạo hoàn toàn từ mã nguồn

(Thay thế cho Slide 12 cũ bị cắt — nhắc Terraform ở đây)

🎤 "Kiến trúc chia 3 vùng. Vùng đỏ giả lập tin tặc. Vùng xanh chứa Honeypot bucket 55 file. Vùng xanh lá là SOC Zone do nhóm tự thiết kế gồm 4 lớp. Toàn bộ hạ tầng quản lý bằng 7 modules Terraform."

💡 Khi hội đồng hỏi kiến trúc, luôn quay lại slide này.

---

## Slide 6 — Luồng phòng thủ Event-Driven 5 bước (= Slide 8 cũ)
👤 NGƯỜI A

Giữ nguyên: Flow 5 bước + bảng GCP Service/Chức năng.

🎤 "Luồng 5 bước event-driven: Monitoring phát hiện bất thường, Cloud Function enrichment 4 lớp, Gemini phân tích severity, Telegram thông báo admin, và tự động disable SA khi approved."

---

## Slide 7 — Detection & Context Enrichment (= Slide 9 cũ)
👤 NGƯỜI A (Slide cuối)

Giữ nguyên nội dung.

📌 **CÂU CHUYỂN GIAO** (Người A → Người B):
🎤 "Vừa rồi em trình bày kiến trúc tổng thể và cơ chế phát hiện. Bây giờ [Tên B] sẽ trình bày chi tiết AI Pipeline, demo hệ thống và kết quả đánh giá."

---

## NGƯỜI B — Kỹ thuật chi tiết, Demo & Kết quả (Slides 8–14)

## Slide 8 — AI Triage Pipeline (= Slide 10 cũ)
👤 NGƯỜI B (Slide đầu tiên)

Giữ nguyên. Sửa nhỏ: "3%" → "3.4%"

🎤 "Em là [Tên B]. AI Triage dùng Gemini 2.5 Flash, fallback OpenAI. Prompt deterministic với JSON schema cố định. AI chỉ chiếm 3.4% tổng thời gian phản ứng."

---

## Slide 9 — Human-in-the-Loop & Remediation (= Slide 11 cũ)
👤 NGƯỜI B

Giữ nguyên.

🎤 "Admin phê duyệt qua Telegram. Link ký HMAC-SHA256, hết hạn 1 giờ, 1 lần dùng. Approve → disable SA + tạo SCC Finding với MITRE mapping."

---

## 🎬 DEMO (~3 phút) — Đặt sau Slide 9

👤 NGƯỜI B demo, NGƯỜI A điều khiển slide

Kịch bản:
1. Chạy attack_simulation.py → 55 file tải
2. Chờ trigger (hoặc video quay sẵn phần chờ)
3. Xem Telegram → Approve
4. SA bị disable + SCC Finding

🎤 "Bây giờ em demo trực tiếp. Em giả lập tấn công và cho Hội đồng thấy toàn bộ pipeline."

---

## Slide 10 — Sơ đồ E2E + Phân tích Bottleneck ⭐ CẢI THIỆN (= Slide 13 cũ)
👤 NGƯỜI B

**Giữ nguyên** phần trên: Sơ đồ 7 ô + T_response ≈ 265s

**THÊM MỚI** — Bảng phân tích bottleneck (từ Bảng 4.5 báo cáo):

| Giai đoạn | Thời gian | Tỷ lệ |
|---|---|---|
| Cloud Monitoring (metric window) | ~255s | **96.2%** |
| Enrichment + AI Triage | ~9s | 3.4% |
| Human Approval + Response | ~1s | 0.4% |

**Callout nổi bật (khung đỏ):**
> ⚠️ **Nút thắt:** Cloud Monitoring chiếm 96.2% — Giải pháp: Log Sink real-time → MTTD < 30 giây

🎤 "Tổng thời gian 265 giây. Phân tích cho thấy 96% là do Cloud Monitoring dùng metric window 60 giây. Enrichment + AI chỉ tốn 9 giây. Nếu chuyển sang Log Sink real-time, MTTD có thể giảm xuống dưới 30 giây."

---

## Slide 11 — Kết quả & Đánh giá ⭐ GỘP MỚI (Slide 14+15 cũ)
👤 NGƯỜI B

**Bố cục: Trên/Dưới (stacked)**

**Phần trên (55%) — Ma trận kiểm thử:**
- **100% phát hiện** (18/18) — font lớn, nổi bật
- Bảng 4 tổ hợp tiêu biểu (giữ nguyên)
- IP nước ngoài: CRITICAL **83%** (5/6)

**Phần dưới (45%) — So sánh hiệu năng:**

| Chỉ số | SOC truyền thống* | Đề tài này | Phạm vi |
|---|---|---|---|
| MTTD | 24-48 giờ | **~4.4 phút** | 1 kịch bản |
| MTTR | Hàng giờ-ngày | **~10 giây** | 1 hành động |

*\* Benchmark SANS 2024*

🎤 "18 kịch bản, 100% phát hiện. IP nước ngoài nhận CRITICAL 83%. MTTD 4.4 phút so với benchmark 24-48 giờ, MTTR 10 giây. Lưu ý phạm vi chỉ 1 kịch bản credential theft."

---

## Slide 12 — Kết luận ⭐ RESTRUCTURE ĐÓNG GÓP
👤 NGƯỜI B

**Chia 2 phần rõ ràng:**

**🎯 Đóng góp chính** (khung nổi bật, nền vàng nhạt):

| # | Đóng góp | Chi tiết |
|---|---|---|
| 1 | **Pipeline SOAR tự xây** | End-to-end trên GCP, không cần Chronicle license |
| 2 | **Context Enrichment 4 lớp** | Nâng chất lượng AI: MEDIUM → HIGH/CRITICAL |
| 3 | **Kiến trúc AI kép** | Gemini + OpenAI fallback → đảm bảo availability |
| 4 | **IaC 100%** | 7 Terraform modules, tái tạo hoàn toàn từ mã nguồn |

**📊 Kết quả đo lường** (khung xanh):
- MTTD: **4.4 phút** (vs 24-48 giờ benchmark)
- MTTR: **10 giây** sau phê duyệt
- Detection rate: **100%** (18/18)

🎤 "Đề tài có 4 đóng góp chính: Thứ nhất, xây dựng thành công pipeline SOAR end-to-end mà không cần Chronicle license. Thứ hai, enrichment 4 lớp giúp AI phân biệt severity chính xác — không có enrichment AI chỉ trả MEDIUM. Thứ ba, kiến trúc AI kép đảm bảo hệ thống luôn hoạt động. Thứ tư, toàn bộ hạ tầng IaC, tái tạo từ mã nguồn."

---

## Slide 13 — Hạn chế & Hướng phát triển (= Slide 17 cũ, bổ sung)
👤 NGƯỜI B

**Cột trái (đỏ nhạt) — ⚠️ Hạn chế:**
1. Không có Chronicle license → tự xây thay thế
2. **Chỉ 1 kịch bản** (credential theft) — chưa đa kịch bản
3. **Metric window 60s** → MTTD tối thiểu 4 phút (chiếm 96%)
4. **LLM non-deterministic** — cùng input, severity có thể thay đổi
5. Không có module UDM riêng
6. YARA-L → SQL: tumbling window → bỏ sót sự kiện biên

**Cột phải (xanh lá) — 🚀 Hướng phát triển:**
1. **Real-time detection** (Log Sink → MTTD < 30s)
2. Mở rộng: privilege escalation, lateral movement
3. YARA-L trên Chronicle khi có license
4. Auto-approve: CRITICAL + confidence ≥ 0.95
5. Tích hợp VirusTotal, AbuseIPDB

🎤 "Về hạn chế, thẳng thắn: không có license Chronicle, chỉ 1 kịch bản, và nút thắt 4 phút do metric window. Về hướng phát triển, chuyển sang Log Sink real-time có thể giảm MTTD xuống dưới 30 giây."

💡 **Thẳng thắn** — hội đồng đánh giá cao sự trung thực.

---

## Slide 14 — Cảm ơn Hội đồng (= Slide 18 cũ)
👤 CẢ 2 NGƯỜI

🎤 "Chúng em xin cảm ơn Hội đồng đã lắng nghe. Kính mời quý Thầy Cô đặt câu hỏi."

---

## Backup Slides (7 slides — chỉ dùng khi Q&A)

| # | Nội dung | Dùng khi hỏi |
|---|---|---|
| B1 | **YARA-L Rules + Terraform IaC** | "YARA-L chạy thế nào? Terraform?" |
| B2 | **AI trong SOC** (Gemini/Chronicle chi tiết) | "AI hoạt động cụ thể ra sao?" |
| B3 | **Context Enrichment Impact** (Bảng 4.6) | "Enrichment có thực sự cần thiết?" |
| B4 | **Dual-AI Fallback** log (Hình 4.1-4.2) | "Tại sao 2 AI engine?" |
| B5 | Chi phí vận hành | "Chi phí bao nhiêu?" |
| B6 | Mapping Lý thuyết → Thực tiễn | "Lý thuyết liên hệ thực tiễn?" |
| B7 | So sánh Event-driven vs SQL Polling | "Tại sao event-driven?" |

---

## Phân chia Q&A

### Người A trả lời (Kiến trúc & Lý thuyết)
- "Tại sao serverless?" → Pay-per-use, auto-scale
- "Chronicle vs Splunk?" → Cloud-native, UDM chuẩn hóa
- "Pub/Sub nghẽn?" → Retry + dead-letter queue
- "SOAR khác SIEM?" → SIEM thu thập, SOAR tự động hóa
- "Attacker biết honeypot?" → Defense-in-Depth

### Người B trả lời (AI & Bảo mật & Đánh giá)
- "Gemini hallucinate?" → Fixed JSON + validate 7 trường
- "Tại sao 2 engine?" → Lật backup B4
- "AI bị adversarial?" → Pipeline deterministic
- "HMAC đủ an toàn?" → Chuẩn công nghiệp + TTL 1h
- "Enrichment cần thiết?" → Lật backup B3 (không enrichment → chỉ MEDIUM)
- "So với SOC thương mại?" → PoC chứng minh feasibility

---

## Bảng ánh xạ cuối cùng: V3 (18 slides) → V4 (14 slides)

| Slide V3 | Slide V4 | Hành động |
|---|---|---|
| 1 (Bìa) | 1 | Thêm tên EN |
| 2 (Agenda) | — | ❌ CẮT |
| 3 (Bối cảnh) | 2 | Giữ |
| 4 (Mục tiêu) | 3 | Giữ |
| 5 (Nền tảng LT) | 4 | Thêm 1 dòng AI |
| 6 (AI trong SOC) | — | ❌ CẮT → Backup B2 |
| 7 (3 Vùng) | 5 | Thêm annotation + Terraform |
| 8 (Flow 5 bước) | 6 | Giữ |
| 9 (Detection+Enrichment) | 7 | Giữ |
| 10 (AI Triage) | 8 | Sửa 3% → 3.4% |
| 11 (HitL+Remediation) | 9 | Giữ |
| 12 (YARA-L+TF) | — | ❌ CẮT → Backup B1 |
| 13 (E2E) | 10 | ⭐ Thêm bảng bottleneck |
| 14 (Ma trận) + 15 (Đánh giá) | 11 | ⭐ GỘP |
| 16 (Kết luận) | 12 | ⭐ Restructure đóng góp |
| 17 (Hạn chế+HP) | 13 | Bổ sung chi tiết |
| 18 (Cảm ơn) | 14 | Giữ |

---

## Tổng thời gian

| Phần | Slides | Thời gian |
|---|---|---|
| Mở đầu | 1–3 | ~1.5 phút |
| Lý thuyết | 4 | ~1 phút |
| Kiến trúc & Kỹ thuật | 5–9 | ~3.5 phút |
| 🎬 DEMO | — | ~3 phút |
| Kết quả | 10–11 | ~2 phút |
| Kết luận | 12–14 | ~2 phút |
| **TỔNG** | **14 slide + demo** | **~13 phút** |

> Còn ~2 phút buffer cho chuyển giao + trường hợp nói chậm hơn dự kiến.
