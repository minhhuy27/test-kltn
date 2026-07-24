# LỜI THOẠI BẢO VỆ KHÓA LUẬN — V5 (26 SLIDES)

**Đề tài:** THỬ NGHIỆM GOOGLE CHRONICLE: ỨNG DỤNG AI TRONG GIÁM SÁT VÀ PHẢN ỨNG AN NINH TRÊN NỀN TẢNG GOOGLE CLOUD

**Tổng:** 16 slide chính + 1 DEMO + 9 backup | ~12 phút trình bày + ~3 phút demo = ~15 phút

## Phân chia người trình bày

| Phần | Slides | Người | Thời gian |
|---|---|---|---|
| Mở đầu + Lý thuyết + Kiến trúc | 1–7 | **NGƯỜI A** | ~4.5 phút |
| *Chuyển giao* | | A → B | ~15 giây |
| AI + Remediation | 8–9 | **NGƯỜI B** | ~2 phút |
| 🎬 DEMO | — | **B demo, A điều khiển slide** | ~3 phút |
| Kết quả + Kết luận | 10–16 | **NGƯỜI B** | ~4.5 phút |

---

## PHẦN CHÍNH (16 slides)

---

### Slide 1 — Trang bìa
👤 NGƯỜI A

🎤 *"Kính chào quý Thầy Cô trong Hội đồng. Em là [Tên A] cùng [Tên B], xin trình bày khóa luận tốt nghiệp: Thử nghiệm Google Chronicle — Ứng dụng AI trong giám sát và phản ứng an ninh trên nền tảng Google Cloud. Bài trình bày gồm 5 phần: Bối cảnh & Vấn đề, Kiến trúc hệ thống, Hiện thực hóa, Kết quả đánh giá, và Kết luận."*

⏱️ ~30 giây

---

### Slide 2 — Bối cảnh & Vấn đề
👤 NGƯỜI A

🎤 *"Theo ISC2 năm 2025, toàn cầu thiếu 4.8 triệu chuyên gia an ninh. IBM cho thấy trung bình mất 181 ngày mới phát hiện sự cố, và mỗi vụ rò rỉ thiệt hại gần 5 triệu đô. SOC truyền thống đang đối mặt 3 thách thức lớn: alert fatigue khiến analyst bỏ sót mối đe dọa thật, thời gian phản ứng kéo dài hàng giờ đến hàng ngày, và chi phí license SIEM truyền thống rất đắt đỏ. Câu hỏi đặt ra: liệu kiến trúc Cloud-Native kết hợp AI có thể giải quyết?"*

⏱️ ~40 giây

---

### Slide 3 — Mục tiêu & Phạm vi
👤 NGƯỜI A

🎤 *"Về mục tiêu lý thuyết, em nghiên cứu hạn chế của SIEM/SOC truyền thống, tìm hiểu SOAR và kiến trúc Cloud-Native của Google Chronicle bao gồm UDM và YARA-L. Về thực tiễn, em xây dựng Lab trên GCP, giả lập tấn công credential theft, thiết kế detection event-driven, tích hợp Context Enrichment 4 lớp và AI kép Gemini/OpenAI, đồng thời thử nghiệm chuyển đổi YARA-L sang SQL trên BigQuery. Phạm vi đã thực hiện: 18 kịch bản kiểm chứng trên GCP, tự xây pipeline thay thế Chronicle native. Ngoài phạm vi: chưa triển khai trên mạng doanh nghiệp thực tế, chưa bao quát toàn bộ MITRE ATT&CK, và không sử dụng giao diện Chronicle do giới hạn license."*

⏱️ ~50 giây

---

### Slide 4 — Nền tảng lý thuyết
👤 NGƯỜI A

🎤 *"SOC truyền thống dựa trên 3 trụ cột: Con người, Quy trình, Công nghệ — nhưng đang đối mặt alert fatigue, thiếu nhân sự và chi phí đắt đỏ. Điều này thúc đẩy sự ra đời của SOAR — giải pháp tự động hóa điều phối và phản ứng. Google tích hợp SIEM và SOAR trong Chronicle với UDM chuẩn hóa log và YARA-L Detection Rules. Tuy nhiên, do không có license Chronicle Enterprise, nhóm tự xây pipeline mô phỏng trên GCP. Về AI, nhóm sử dụng Gemini phân tích log với Fixed JSON schema và Human-in-the-loop để kiểm soát rủi ro hallucination."*

⏱️ ~45 giây

---

### Slide 5 — Mô hình kiến trúc 3 vùng ⭐
👤 NGƯỜI A

🎤 *"Kiến trúc chia 3 vùng. Vùng đỏ — Attacker Zone — giả lập tin tặc sử dụng gcloud CLI, gsutil và Python SDK với Service Account Key bị đánh cắp. Vùng xanh dương — Victim Zone — chứa Honeypot bucket với 55 file giả lập dữ liệu nhạy cảm. Vùng xanh lá — SOC Zone — là phần nhóm tự thiết kế, gồm toàn bộ pipeline event-driven từ phát hiện, làm giàu ngữ cảnh, AI phân tích đến phản ứng tự động. Toàn bộ hạ tầng được quản lý bằng 7 modules Terraform, có thể tái tạo hoàn toàn từ mã nguồn."*

⏱️ ~45 giây

> 💡 Khi hội đồng hỏi kiến trúc → luôn quay lại slide này.

---

### Slide 6 — Luồng phòng thủ Event-Driven (5 bước)
👤 NGƯỜI A

🎤 *"Luồng phòng thủ gồm 5 bước event-driven. Bước 1: Cloud Monitoring phát hiện bất thường khi số lượng tải file vượt ngưỡng 25 lần trong 60 giây. Bước 2: Cloud Function thực hiện enrichment 4 lớp bổ sung ngữ cảnh. Bước 3: Gemini AI phân tích severity và tạo báo cáo. Bước 4: Gửi cảnh báo qua Telegram cho SOC Admin phê duyệt. Bước 5: Khi được approve, hệ thống tự động disable Service Account và tạo SCC Finding."*

⏱️ ~40 giây

---

### Slide 7 — Detection & Context Enrichment
👤 NGƯỜI A (slide cuối)

🎤 *"Detection dùng event-driven: Log-based Metric đếm trên 25 lần objects.get trong 60 giây thì trigger Cloud Function. Function thực hiện enrichment 4 lớp: IP geolocation, thời gian truy cập, công cụ sử dụng, và thông tin tài sản. Mỗi lớp giúp bổ sung ngữ cảnh để AI đánh giá chính xác hơn. Ví dụ: cùng hành vi tải file, nếu IP nước ngoài kết hợp gsutil và ngoài giờ hành chính → CRITICAL; ngược lại IP Việt Nam trong giờ → chỉ HIGH."*

📌 **CÂU CHUYỂN GIAO:**
🎤 *"Vừa rồi em đã trình bày kiến trúc tổng thể và cơ chế phát hiện. Bây giờ [Tên B] sẽ trình bày chi tiết về AI Triage Pipeline, cơ chế phản ứng tự động và kết quả đánh giá."*

⏱️ ~50 giây (bao gồm chuyển giao)

---

### Slide 8 — AI Triage Pipeline
👤 NGƯỜI B (slide đầu tiên)

🎤 *"Em là [Tên B]. Về AI Triage Pipeline: hệ thống sử dụng Gemini 2.5 Flash làm engine chính, fallback sang GPT-5.4 Mini khi Gemini gặp lỗi. Prompt được thiết kế deterministic với Fixed JSON schema gồm 7 trường bắt buộc: severity, confidence, summary, reason, recommended_action, escalate, và mitre_technique. Cách tiếp cận này kiểm soát được output của AI, tránh hallucination. AI chỉ chiếm khoảng 9 giây xử lý — tức 3.4% tổng thời gian phản ứng."*

⏱️ ~40 giây

---

### Slide 9 — Human-in-the-Loop & Remediation
👤 NGƯỜI B

🎤 *"SOC Admin phê duyệt hành động qua Telegram. Link phê duyệt được ký bằng HMAC-SHA256, hết hạn sau 1 giờ và chỉ dùng được 1 lần — đảm bảo an toàn. Khi Admin nhấn Approve, hệ thống thực hiện 2 hành động: Bước 1 — disable Service Account bị xâm phạm qua IAM API; Bước 2 — tạo SCC Finding với MITRE ATT&CK mapping trên Security Command Center. Toàn bộ pipeline phản hồi trong khoảng 10 giây sau khi Admin phê duyệt."*

⏱️ ~40 giây

---

### 🎬 DEMO (~3 phút) — Slide 10
👤 NGƯỜI B demo, NGƯỜI A điều khiển slide

🎤 *"Bây giờ em sẽ demo trực tiếp toàn bộ pipeline."*

**Kịch bản:**
1. Chạy `attack_simulation.py` → 55 file được tải
2. Chờ Cloud Monitoring trigger (~4 phút) — có thể dùng video quay sẵn phần chờ
3. Xem cảnh báo Telegram → nhấn Approve Remediation
4. Kiểm tra: Service Account bị disabled + SCC Finding được tạo

🎤 *"Như Hội đồng thấy, từ lúc tấn công đến khi nhận cảnh báo mất khoảng 4 phút, và sau khi phê duyệt, Service Account bị khóa trong 10 giây."*

⏱️ ~3 phút

---

### Slide 11 — Sơ đồ luồng End-to-End ⭐
👤 NGƯỜI B

🎤 *"Sơ đồ End-to-End cho thấy toàn bộ thời gian phản ứng. MTTD — thời gian phát hiện — khoảng 265 giây, tức 4.4 phút. Phân tích bottleneck cho thấy Cloud Monitoring chiếm 253 giây, tương đương 96% tổng thời gian — đây là do metric window 60 giây mặc định. Phần Enrichment và AI Triage chỉ tốn 9 giây. MTTR — thời gian phản ứng sau khi Admin phê duyệt — khoảng 10 giây. Điều này cho thấy nút thắt không nằm ở pipeline của nhóm, mà nằm ở cơ chế gom metric của Cloud Monitoring."*

⏱️ ~45 giây

---

### Slide 12 — Ma trận thử nghiệm
👤 NGƯỜI B

🎤 *"Ma trận gồm 18 kịch bản kết hợp các yếu tố: 3 công cụ tấn công, IP trong nước và nước ngoài, trong giờ và ngoài giờ. Tỷ lệ phát hiện đạt 100% — tất cả 18 kịch bản đều được phát hiện và cảnh báo đúng. Đặc biệt, với IP nước ngoài, AI đánh giá CRITICAL 83% trường hợp — cho thấy Context Enrichment giúp AI phân biệt mức độ nguy hiểm hiệu quả."*

⏱️ ~35 giây

---

### Slide 13 — Đánh giá hiệu năng
👤 NGƯỜI B

🎤 *"Tổng hợp hiệu năng: MTTD 4.4 phút so với benchmark SANS 2024 là 24 đến 48 giờ — nhanh hơn đáng kể. MTTR 10 giây so với benchmark hàng giờ đến hàng ngày. Detection rate đạt 100%. Chi phí toàn bộ thử nghiệm nằm trong Free Tier của GCP, chi phí phát sinh gần bằng 0. Lưu ý phạm vi so sánh: hệ thống hiện tại chỉ cover 1 kịch bản credential theft, chưa phải SOC toàn diện."*

⏱️ ~35 giây

---

### Slide 14 — Kết luận
👤 NGƯỜI B

🎤 *"Đề tài có 4 đóng góp chính. Thứ nhất, xây dựng thành công pipeline SOAR end-to-end trên GCP mà không cần Chronicle license. Thứ hai, Context Enrichment 4 lớp giúp AI phân biệt severity chính xác — nếu không có enrichment, AI chỉ trả MEDIUM. Thứ ba, kiến trúc AI kép Gemini + GPT fallback đảm bảo hệ thống luôn hoạt động, có minh chứng thực tế Gemini bị lỗi và GPT xử lý thành công. Thứ tư, toàn bộ hạ tầng 100% IaC với 7 modules Terraform, có thể tái tạo hoàn toàn từ mã nguồn."*

⏱️ ~45 giây

---

### Slide 15 — Hạn chế & Hướng phát triển
👤 NGƯỜI B

🎤 *"Về hạn chế, thẳng thắn ghi nhận: không có Chronicle license nên phải tự xây pipeline thay thế; chỉ cover 1 kịch bản credential theft; nút thắt 4 phút do metric window 60 giây chiếm 96%; LLM có tính non-deterministic; và YARA-L chuyển sang SQL dùng tumbling window có thể bỏ sót event ở biên. Về hướng phát triển: chuyển sang Log Sink real-time để giảm MTTD xuống dưới 30 giây; mở rộng kịch bản privilege escalation, lateral movement; triển khai YARA-L trên Chronicle khi có license với hop window khắc phục bỏ sót biên; và tích hợp threat intelligence như VirusTotal, AbuseIPDB."*

⏱️ ~50 giây

> 💡 Thẳng thắn — hội đồng đánh giá cao sự trung thực về hạn chế.

---

### Slide 16 — Cảm ơn Hội đồng
👤 CẢ 2 NGƯỜI

🎤 *"Chúng em xin cảm ơn quý Thầy Cô trong Hội đồng đã lắng nghe. Kính mời quý Thầy Cô đặt câu hỏi."*

⏱️ ~10 giây

---

## BACKUP — Lời thoại khi bị hỏi

### B1 — YARA-L Rules trên BigQuery
🎤 *"Do không có Chronicle license, em chuyển logic YARA-L sang SQL trên BigQuery. 3 luật: Mass Download đếm trên 25 objects.get trong 60 giây, Off-Hours kiểm tra truy cập ngoài giờ hành chính, và Suspicious Tool phát hiện gsutil hoặc Python SDK. Kết quả query BigQuery cho thấy hệ thống phát hiện chính xác 55 file được tải."*

### B2 — Infrastructure as Code (Terraform)
🎤 *"Toàn bộ hạ tầng được quản lý bằng 7 modules Terraform: network, storage, IAM, serverless, monitoring, SCC, và attacker. Bất kỳ ai có source code đều có thể tái tạo hoàn toàn lab trong vài phút."*

### B3 — AI trong SOC
🎤 *"AI đóng 3 vai trò: phân tích log tự động giảm tải Analyst Tier 1, tương quan ngữ cảnh phát hiện mối đe dọa, và tóm tắt sự cố rút ngắn điều tra. Rủi ro hallucination được kiểm soát bằng Fixed JSON schema, validation đầu ra, và human-in-the-loop."*

### B4 — Ảnh hưởng Context Enrichment
🎤 *"Bảng này cho thấy cùng hành vi gsutil tải file, không có enrichment AI chỉ trả MEDIUM. Khi bổ sung IP nước ngoài + ngoài giờ, AI nhận diện CRITICAL với confidence 1.0. Enrichment tạo ra sự khác biệt quyết định trong đánh giá severity."*

### B5 — Dual-AI Fallback
🎤 *"Đây là log thực tế: Gemini trả lỗi HTTP 400, hệ thống tự động chuyển sang GPT-5.4 Mini và xử lý thành công trong cùng request. Tin nhắn Telegram bên phải là do GPT sinh ra — vẫn đầy đủ 7 trường JSON."*

### B6 — Chi phí vận hành
🎤 *"Toàn bộ pipeline dùng dịch vụ serverless nằm trong Free Tier của GCP. Chi phí duy nhất phát sinh là API calls cho LLM, và cả Gemini lẫn GPT đều có free tier giới hạn theo RPM/RPD. Ưu điểm serverless: chỉ tính phí khi có sự kiện — không phát sinh chi phí khi hệ thống idle."*

### B7 — Mapping Lý thuyết → Thực tiễn
🎤 *"Bảng mapping cho thấy mỗi khái niệm lý thuyết đều có thành phần tương ứng trong hệ thống. SOC 3 trụ cột → Lab 3 vùng. SIEM → Cloud Audit Log + BigQuery. SOAR → Cloud Functions event-driven. UDM → Context Enrichment 4 lớp. YARA-L → BigQuery SQL. Không có lý thuyết nào bị treo."*

### B8 — Tại sao Event-driven?
🎤 *"Event-driven phản ứng nhanh hơn polling và không tốn chi phí khi idle. Tuy nhiên, cả hai đều dùng tumbling window nên có thể bỏ sót event ở biên. YARA-L trên Chronicle dùng hop window chồng lấn khắc phục — đây là lý do hướng phát triển đề xuất triển khai YARA-L khi có license."*

### B9 — Dual-Pipeline (Bulk vs Real-Time)
🎤 *"Dạ có thể giảm MTTD xuống 10-15 giây. Em đã thử nghiệm Pipeline B dùng Log Sink trực tiếp thay vì Cloud Monitoring. Tuy nhiên, mỗi log event đều trigger 1 Cloud Function nên chi phí tăng tuyến tính. Em chỉ áp dụng cho tài sản quan trọng — Crown Jewel — còn bulk download vẫn dùng metric window để cân bằng chi phí."*

---

## Phân chia Q&A

### Người A trả lời (Kiến trúc & Lý thuyết)
| Câu hỏi | Trả lời tóm tắt |
|---|---|
| "Tại sao serverless?" | Pay-per-use, auto-scale, không cần server 24/7 |
| "Chronicle vs Splunk?" | Cloud-native, UDM chuẩn hóa, tích hợp SIEM+SOAR |
| "Pub/Sub nghẽn?" | Retry + dead-letter queue |
| "SOAR khác SIEM?" | SIEM thu thập & phân tích, SOAR tự động hóa phản ứng |
| "Attacker biết honeypot?" | Defense-in-depth, không chỉ dựa vào honeypot |
| "UDM vs Enrichment?" | UDM chuẩn hóa đa nguồn + ngữ cảnh; hệ thống 1 nguồn log nên tập trung enrichment |

### Người B trả lời (AI & Bảo mật & Đánh giá)
| Câu hỏi | Trả lời tóm tắt |
|---|---|
| "Gemini hallucinate?" | Fixed JSON schema + validate 7 trường + human-in-the-loop |
| "Tại sao 2 engine?" | → Lật backup B5 |
| "AI bị adversarial?" | Pipeline deterministic, AI chỉ phân tích không quyết định |
| "HMAC đủ an toàn?" | Chuẩn công nghiệp + TTL 1h + one-time-use guard |
| "Enrichment cần thiết?" | → Lật backup B4 |
| "So với SOC thương mại?" | PoC chứng minh feasibility, phạm vi 1 kịch bản |
| "Giảm MTTD được không?" | → Lật backup B9 |

---

## Tổng thời gian

| Phần | Slides | Thời gian |
|---|---|---|
| Mở đầu + Bối cảnh | 1–3 | ~2 phút |
| Lý thuyết + Kiến trúc | 4–7 | ~3 phút |
| AI + Remediation | 8–9 | ~1.5 phút |
| 🎬 DEMO | 10 | ~3 phút |
| Kết quả + Đánh giá | 11–13 | ~2 phút |
| Kết luận + Hạn chế | 14–16 | ~2 phút |
| **TỔNG** | **16 slide + demo** | **~13.5 phút** |

> Còn ~1.5 phút buffer cho chuyển giao + trường hợp nói chậm hơn dự kiến.
