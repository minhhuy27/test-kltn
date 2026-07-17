# 📚 TÀI LIỆU ÔN TẬP — CHUẨN BỊ GẶP GIẢNG VIÊN PHẢN BIỆN

> **Đề tài:** Thử nghiệm Google Chronicle — Ứng dụng AI trong giám sát và phản ứng an ninh trên nền tảng Google Cloud
> **Buổi gặp:** Thứ 7, tuần này

---

## PHẦN 1: TỔNG QUAN ĐỀ TÀI

### 1.1. Bài toán giải quyết
- SOC truyền thống phụ thuộc con người → chậm, dễ bỏ sót, thiếu nhân lực 24/7
- Cần một pipeline **tự động, event-driven** kết hợp AI để giảm MTTD/MTTR
- Tận dụng hạ tầng **Serverless** (không cần quản lý server) để giảm chi phí vận hành

### 1.2. Mục tiêu
1. Xây dựng pipeline phát hiện - phân tích - phản ứng **hoàn toàn serverless** trên GCP
2. Tích hợp **AI (Gemini + OpenAI fallback)** để tự động đánh giá mức độ nghiêm trọng
3. Áp dụng mô hình **Human-in-the-Loop** — AI đề xuất, con người phê duyệt
4. Triển khai toàn bộ hạ tầng bằng **Infrastructure as Code (Terraform)**

### 1.3. Phạm vi
- **Kịch bản tấn công:** Data Exfiltration từ Cloud Storage (insider threat)
- **Nền tảng:** Google Cloud Platform (GCP)
- **Không bao gồm:** Triển khai Chronicle thực tế (do giới hạn license) → thay bằng BigQuery để kiểm chứng logic YARA-L

---

## PHẦN 2: CƠ SỞ LÝ THUYẾT (Chương 2 trong báo cáo)

### 2.1. SOC — Trung tâm Vận hành An ninh
- **Định nghĩa:** Bộ phận/đội ngũ chuyên trách giám sát, phát hiện, phân tích và ứng phó mối đe dọa an ninh mạng 24/7 [1]
- **Ba trụ cột (PPT):** Con người (People) + Quy trình (Process) + Công nghệ (Technology) [2]
- **4 chức năng chính:** Giám sát → Phát hiện → Điều tra → Phản ứng
- **4 thách thức SOC truyền thống:**
  1. **Khối lượng log khổng lồ** — SIEM cũ tính phí theo EPS/GB → buộc "bỏ bớt log" → tạo điểm mù
  2. **Alert fatigue** — 90-95% cảnh báo là false positive → analyst "trơ lì", bỏ sót true positive
  3. **Thiếu nhân sự** — 59% tổ chức thiếu hụt kỹ năng an ninh mạng nghiêm trọng (ISC² 2025) [4]
  4. **Thời gian phản ứng dài** — quy trình thủ công, công cụ rời rạc, copy-paste IP giữa các tool

### 2.2. SIEM — Security Information and Event Management
- **Vai trò:** Nền tảng thu thập, chuẩn hóa và tương quan log từ đa nguồn
- **Hạn chế SIEM truyền thống:**
  - Lưu trữ và tính toán gắn chặt (coupled) → giới hạn bởi phần cứng vật lý
  - Dữ liệu lưu trữ "nóng" chỉ 30-90 ngày, sau đó phải "lưu trữ lạnh"
  - Truy vấn dữ liệu cũ (re-hydration) mất hàng giờ/ngày
  - Thiếu khả năng tương quan đa sự kiện theo thời gian

### 2.3. SOAR — Security Orchestration, Automation and Response
- **Vai trò:** Tự động hóa phản ứng sự cố, giảm tải cho con người
- **Mối quan hệ SIEM – SOAR – SOC:**
  - SIEM = "mắt" (thu thập, phát hiện)
  - SOAR = "tay" (phản ứng, xử lý tự động)
  - SOC = tổng thể (người + quy trình + công nghệ)

### 2.4. Google Chronicle (Google Security Operations)
> ⚠️ **ĐÂY LÀ PHẦN CỐT LÕI — giảng viên phản biện rất có thể hỏi về Chronicle**

**Kiến trúc Cloud-Native (Decoupled Architecture) [8]:**
- **Lớp lưu trữ:** Colossus (hệ thống tệp phân tán thế hệ mới của Google) → lưu Petabyte dữ liệu
- **Lớp tính toán:** BigQuery + Dremel → quét hàng tỷ dòng log trong vài giây
- **Ưu điểm tách biệt:** Khi dữ liệu tăng đột biến (DDoS), chỉ cần mở rộng lớp lưu trữ, không ảnh hưởng tính toán

**Khả năng lưu trữ nóng:**
- Toàn bộ dữ liệu mặc định lưu "nóng" **12 tháng** (SIEM cũ chỉ 30-90 ngày)
- Truy vấn dữ liệu 1 năm trước = tốc độ như dữ liệu 5 phút trước (mili-giây)
- Xóa bỏ khái niệm "re-hydration"

**Cơ chế làm giàu dữ liệu:**
- **Aliasing tự động:** Giải quyết vấn đề IP động (DHCP) — liên kết IP → hostname → user
- **Threat Intelligence tích hợp:** Mandiant, VirusTotal, các đối tác bên thứ 3

**3 chức năng cốt lõi:** Detection → Incident Investigation → Threat Hunting

### 2.5. UDM — Unified Data Model
- **Vấn đề:** Log thô từ nhiều nguồn (firewall, endpoint, cloud) có format khác nhau → khó tương quan
- **Giải pháp UDM:** Chuẩn hóa mọi log về 1 schema chung, tự động bởi Parser
- **Ví dụ ánh xạ:**

| Trường UDM | Cloud Audit Log tương ứng |
|---|---|
| `principal.user.userid` | `authenticationInfo.principalEmail` |
| `principal.ip` | `requestMetadata.callerIp` |
| `network.http.user_agent` | `requestMetadata.callerSuppliedUserAgent` |
| `target.resource.name` | `resourceName` |

- **Ý nghĩa:** Analyst chỉ cần viết `principal.ip` thay vì regex phức tạp trên log thô

### 2.6. YARA-L — Ngôn ngữ phát hiện
- **Nguồn gốc:** Google (đội ngũ Chronicle Security), kế thừa từ YARA (VirusTotal)
- **Tên gọi:** "L" = **Logs**. YARA dành cho File, YARA-L dành cho Log
- **5 thành phần:** `meta`, `events`, `match`, `outcome`, `condition` [10]
- **Ưu điểm vượt trội so với SQL/SPL:**
  - **Detection-as-Code:** Viết luật như viết code, quản lý bằng Git
  - **Truy vấn trực tiếp trên UDM:** Không cần regex
  - **Tương quan theo thời gian (hop window):** "Nếu A xảy ra, rồi B xảy ra trong 5 phút, nhưng C không xảy ra"
  - **Hiệu năng Petabyte:** Quét nhiều năm dữ liệu trong vài giây
  - **Curated Detections:** Bộ luật sẵn từ Mandiant/Google, cập nhật liên tục

| So sánh | YARA (truyền thống) | YARA-L |
|---|---|---|
| Đối tượng | File (mã độc) | Log (sự kiện an ninh) |
| Dữ liệu | Tĩnh (1 file) | Chuỗi sự kiện theo thời gian |
| Ví dụ | "Túi xách có con dao?" | "Mua dao → đến ngân hàng → đeo mặt nạ trong 10 phút?" |

### 2.7. AI trong SOC
**Vai trò:**
- Giảm MTTD/MTTR từ giờ/ngày → phút/giây
- Giảm false positive bằng cách học mẫu hành vi bình thường
- Tự động hóa tác vụ lặp lại (thu thập IP, kiểm tra virus)

**Rủi ro [12]:**
- **Ảo giác (Hallucination):** AI có thể "bịa" thông tin thuyết phục nhưng sai
- **Hộp đen:** Khó giải thích tại sao AI đưa ra kết luận → khó kiểm toán
- **Tấn công ngược:** Data Poisoning — hacker làm nhiễm dữ liệu huấn luyện

**Triết lý cốt lõi: AI hỗ trợ, KHÔNG thay thế con người [12]**
- Mô hình "Phi công và Lái tự động": AI xử lý 90% tình huống ổn định, con người cầm lái khi khẩn cấp
- **Human-in-the-Loop:** AI tóm tắt + đề xuất → con người "ký duyệt"
- Con người chịu trách nhiệm pháp lý cuối cùng

**Context Enrichment + Prompt Engineering:**
- Dữ liệu thô (metric) không đủ cho AI → cần bổ sung ngữ cảnh (IP, UA, thời gian)
- Prompt Engineering: Gán vai trò SOC triage → cung cấp bối cảnh tổ chức (VN) → yêu cầu JSON output cố định
- Dual-AI Fallback: Primary (Gemini) + Backup (OpenAI GPT) → đảm bảo availability

### 2.8. ⭐ LÝ DO CHUYỂN HƯỚNG TỪ CHRONICLE → KIẾN TRÚC EVENT-DRIVEN HIỆN TẠI

> **Đây là câu hỏi phản biện gần như chắc chắn sẽ được hỏi — phải trả lời mạch lạc và tự tin.**

**Vấn đề cốt lõi:**
- Google Chronicle (Google SecOps) yêu cầu **license Enterprise** với chi phí rất cao
- Tài khoản cá nhân/sinh viên **không thể truy cập** giao diện Native của Chronicle
- Không thể triển khai YARA-L detection rules trên Chronicle thực tế
- Không thể sử dụng Gemini-in-Chronicle (tính năng AI tích hợp sẵn)

**Giải pháp chuyển hướng (được nêu rõ tại Mục 1.3 trong báo cáo):**

Đề tài **ứng dụng kiến thức lý thuyết đã nghiên cứu** để thiết kế một kiến trúc SOAR phi máy chủ (Serverless) thay thế, ánh xạ từng chức năng Chronicle sang dịch vụ GCP tương đương:

| Chức năng Chronicle | Thành phần thay thế trên GCP | Giải thích |
|---|---|---|
| **Log Ingestion** | Cloud Audit Logs + Log Sink + Pub/Sub | Thu thập và định tuyến log |
| **Detection Engine (YARA-L)** | Log-based Metric + Alert Policy | Phát hiện hành vi bất thường theo ngưỡng |
| **Data Enrichment** | Cloud Logging API + ip-api.com + Python | 4 lớp làm giàu ngữ cảnh thủ công |
| **Gemini in Chronicle** | Gemini API (trực tiếp) + OpenAI fallback | AI phân tích với kiến trúc kép |
| **SOAR Response** | Webhook Cloud Function + Telegram | Human-in-the-loop + tự động vô hiệu hóa SA |
| **Security Findings** | Security Command Center V2 | Ghi nhận finding với MITRE ATT&CK |
| **YARA-L Validation** | BigQuery SQL | Kiểm chứng logic phát hiện trên dữ liệu thật |

**Cách trả lời khi bị hỏi:**
> "Đề tài vẫn giữ tên Chronicle vì phần lý thuyết (Chương 2) nghiên cứu chuyên sâu về kiến trúc, UDM, và YARA-L của Chronicle. Tuy nhiên, do giới hạn license Enterprise, phần thực nghiệm (Chương 3) chuyển sang thiết kế kiến trúc event-driven serverless trên GCP — ánh xạ từng chức năng của Chronicle sang các dịch vụ GCP tương đương. Logic phát hiện YARA-L được kiểm chứng qua BigQuery SQL trên dữ liệu Cloud Audit Log thực tế. Đây là cách tiếp cận thực tế nhất trong giới hạn tài nguyên sinh viên."

---

## PHẦN 3: KIẾN TRÚC HỆ THỐNG (3 VÙNG)

### 3.1. Sơ đồ luồng tổng thể
```
Honeypot Bucket → Cloud Audit Logs → Log Sink → Pub/Sub
                                    → Log-based Metric → Alert Policy → Pub/Sub
                                                                         ↓
                                                            Orchestrator Bot (Cloud Function)
                                                                         ↓
                                                            4 lớp Enrichment → AI Triage → Telegram Alert
                                                                                              ↓ (Admin nhấn Approve)
                                                            Webhook Remediation (Cloud Function)
                                                                         ↓
                                                            Disable SA + SCC Finding + Audit Log
```

### 3.2. Ba vùng kiến trúc

| Vùng | Thành phần | Chức năng |
|---|---|---|
| **Vùng 1: Phát hiện** | Honeypot Bucket, Cloud Audit Logs, Log Sink, Log-based Metric, Alert Policy | Phát hiện hành vi bất thường |
| **Vùng 2: Phân tích** | Orchestrator Bot, Context Enrichment (4 lớp), Dual-AI Triage | Làm giàu ngữ cảnh + AI đánh giá |
| **Vùng 3: Phản ứng** | Telegram Bot, Webhook Remediation, IAM API, SCC V2 | Human-in-the-loop + tự động xử lý |

---

## PHẦN 4: CHI TIẾT KỸ THUẬT TỪNG THÀNH PHẦN

### 4.1. Honeypot Bucket
- **Vai trò:** Bucket bẫy chứa 55 file giả lập dữ liệu nhạy cảm
- **Cơ chế:** Mọi truy cập `storage.objects.get` đều được ghi vào Cloud Audit Logs (Data Access)
- **Ý nghĩa:** Bất kỳ ai tải file = hành vi đáng ngờ (vì không ai có lý do hợp lệ truy cập)

### 4.2. Log Sink + Pub/Sub
- **Log Sink:** Bộ lọc chuyển tiếp audit log từ Cloud Logging sang Pub/Sub topic
- **Pub/Sub:** Message queue trung gian — đảm bảo delivery ít nhất 1 lần (at-least-once)
- **Tại sao cần Pub/Sub?** Giải ghép (decouple) giữa nguồn log và hàm xử lý

### 4.3. Log-based Metric + Alert Policy
- **Log-based Metric:** Đếm số lượt `storage.objects.get` theo `principalEmail`
- **Alert Policy:** Kích hoạt khi vượt ngưỡng **≥25 downloads trong 60 giây**
- **Alignment Period = 60s:** Cửa sổ tumbling — gom event trong 60 giây rồi đếm
- **⚠️ Đây là bottleneck chính** — chiếm ~96% thời gian phản hồi (~253 giây)

> **CÂU HỎI HAY GẶP:** Tại sao T_response lại cao (~265 giây)?
> **TRẢ LỜI:** 96% thời gian nằm ở hạ tầng Cloud Monitoring (alignment period + cooldown 180s + notification delay). Pipeline ứng dụng (enrichment + AI + Telegram) chỉ mất ~12 giây (~4%).

### 4.4. Context Enrichment (4 lớp)

| Lớp | Nguồn | Dữ liệu thu được | Mục đích |
|---|---|---|---|
| **1. Cloud Logging Query** | Cloud Logging API (`entries:list`) | `callerIp`, `userAgent` | Lấy thông tin mà Alert notification không chứa |
| **2. IP Geolocation** | ip-api.com (free, 45 req/min) | Country, City, ISP | Phát hiện IP nước ngoài bất thường |
| **3. User Agent Analysis** | Phân tích chuỗi UA | gsutil vs Python SDK vs browser | Phân biệt công cụ tấn công tự động |
| **4. Time-of-Day** | Python datetime (UTC+7) | Giờ, ngày, business hours? | Phát hiện truy cập ngoài giờ/cuối tuần |

> **Tại sao cần 4 lớp?** Alert notification từ Cloud Monitoring chỉ chứa metric aggregated data, KHÔNG có callerIp hay userAgent. Phải truy vấn ngược Cloud Logging API để lấy.

### 4.5. AI Triage — Kiến trúc Dual-AI

**Primary:** Gemini 2.5 Flash
- `temperature = 0.1` (giảm tính ngẫu nhiên, tăng tính nhất quán)
- `responseMimeType = "application/json"` (output JSON có cấu trúc)
- Timeout: 60 giây

**Fallback:** OpenAI GPT
- Tự động kích hoạt khi Gemini lỗi (timeout, API error, rate limit)
- Dùng cùng prompt template (`_build_triage_prompt`)
- `response_format: { type: "json_object" }`

**Output schema (7 trường bắt buộc):**
```json
{
  "severity": "LOW|MEDIUM|HIGH|CRITICAL",
  "confidence": 0.0-1.0,
  "should_escalate": true/false,
  "summary": "...",
  "reason": "...",
  "recommended_remediation": "...",
  "service_account_email": "..."
}
```

> **CÂU HỎI HAY GẶP:** AI có bao giờ đánh giá sai không?
> **TRẢ LỜI:** AI là non-deterministic — cùng input có thể cho severity khác nhau giữa các lần chạy. Tuy nhiên, `temperature=0.1` giúp giảm thiểu sự dao động. Hệ thống thiết kế theo nguyên tắc "AI đề xuất, con người quyết định" nên severity sai không gây hậu quả nghiêm trọng.

### 4.6. Telegram Alert + Human-in-the-Loop
- Bot gửi tin nhắn chứa kết quả phân tích AI + nút **"Approve Remediation"**
- Nút này là URL đã ký HMAC-SHA256 (xem mục 4 bên dưới)
- SOC Admin đọc, đánh giá, rồi nhấn nút → kích hoạt webhook

### 4.7. Webhook Remediation (Cloud Function #2)
Khi Admin nhấn "Approve", webhook thực hiện **3 bước tuần tự:**

| Bước | Hành động | API/Service |
|---|---|---|
| 1 | Vô hiệu hóa Service Account bị xâm nhập | IAM REST API (`:disable`) |
| 2 | Tạo Finding trên Security Command Center | SCC V2 API (`CreateFinding`) |
| 3 | Ghi nhật ký kiểm toán (audit trail) | Cloud Logging (structured log) |

**SCC Finding bao gồm:**
- MITRE ATT&CK mapping: Exfiltration (T1020), Credential Access (T1528), Valid Accounts (T1078)
- Exfiltration details: Bucket name, resource type
- Access info: Principal email, method name
- Next steps: 5 bước khắc phục chi tiết

### 4.8. Infrastructure as Code (Terraform)
- **7 modules:** `iam`, `network`, `storage`, `logging_data`, `monitoring`, `scc`, `serverless`
- Toàn bộ hạ tầng tạo bằng 1 lệnh: `terraform apply`
- **Tại sao dùng Terraform?** Reproducible, version-controlled, có thể review trước khi apply

---

## PHẦN 5: CƠ CHẾ BẢO MẬT — DEFENSE-IN-DEPTH

### 5.1. HMAC-SHA256 Signed URL
**Vấn đề:** URL phê duyệt gửi qua Telegram — nếu bị lộ, kẻ tấn công có thể tự approve
**Giải pháp:**
```
sign_payload = f"{incident_id}|{service_account_email}|{severity}|{issued_at}"
signature = HMAC-SHA256(signing_secret, sign_payload)
```
- Webhook kiểm tra chữ ký trước khi thực thi
- Dùng `hmac.compare_digest()` → chống timing attack

### 5.2. Link Expiry
- `issued_at` được nhúng vào URL khi tạo
- Webhook kiểm tra: `now - issued_at > 3600s` → từ chối (link hết hạn sau 1 giờ)
- Cũng kiểm tra `issued_at > now + 300s` → từ chối (chống gian lận thời gian)

### 5.3. One-Time-Use Guard
**Vấn đề:** Admin có thể nhấn "Approve" nhiều lần → double-remediation
**Giải pháp:** Trước khi disable SA, webhook gọi IAM API kiểm tra trạng thái SA. Nếu đã bị disabled → trả HTML thông báo "Link Already Used", KHÔNG thực thi lại.

### 5.4. Tổng hợp các lớp bảo mật

| Lớp | Cơ chế | Chống lại |
|---|---|---|
| Xác thực | HMAC-SHA256 | URL giả mạo |
| Thời gian | Link Expiry (1 giờ) | URL bị lộ sau thời gian dài |
| Idempotent | One-Time-Use Guard | Double-remediation |
| Phát hiện | Honeypot + Threshold | Truy cập trái phép |
| Kiểm toán | Cloud Logging + SCC | Truy vết sau sự cố |

---

## PHẦN 6: KẾT QUẢ THỰC NGHIỆM

### 6.1. Ma trận kiểm thử (18 kịch bản, 12 đại diện)

3 biến đầu vào: **IP** (VN vs nước ngoài) × **User Agent** (gsutil vs Python SDK) × **Thời gian** (giờ HC vs ngoài giờ vs cuối tuần)

| IP | UA | Thời gian | Severity | Confidence |
|---|---|---|---|---|
| 🇻🇳 VN | gsutil | Giờ HC | HIGH | 0.90 |
| 🇻🇳 VN | Python | Ngoài giờ | HIGH | 0.90 |
| 🇳🇱 NL (VPN) | gsutil | Giờ HC | CRITICAL | 1.00 |
| 🇸🇬 SG (VPN) | Python | Giờ HC | CRITICAL | 0.90 |
| 🇳🇱 NL (VPN) | Python | Ngoài giờ | CRITICAL | 1.00 |
| 🇯🇵 JP (VPN) | Python | Cuối tuần | CRITICAL | 0.95 |

**Nhận xét:** IP nước ngoài → luôn CRITICAL. Python SDK > gsutil. Ngoài giờ/cuối tuần tăng confidence. **Tỷ lệ phát hiện: 18/18 = 100%.**

### 6.2. Hiệu năng

| Chỉ số | Giá trị |
|---|---|
| T_response trung bình | **~265 giây (~4.4 phút)** |
| Độ lệch chuẩn (σ) | 17.8 giây |
| Pipeline ứng dụng | ~12 giây (**~4%**) |
| Cloud Monitoring (bottleneck) | ~253 giây (**~96%**) |

### 6.3. YARA-L → BigQuery SQL

| Luật | MITRE | Mô tả |
|---|---|---|
| Mass Download | T1020 | >25 downloads/phút |
| Off-Hours Access | T1530 | Truy cập trước 8h hoặc sau 18h UTC+7 |
| Suspicious Tool | T1078 | Python SDK (không phải gsutil) |

**Hạn chế:** YARA-L dùng **hop window** (chồng lấn), SQL chỉ có **tumbling window** → SQL có thể bỏ sót edge case.

---

## PHẦN 7: CÂU HỎI PHẢN BIỆN DỰ KIẾN & GỢI Ý TRẢ LỜI

### Q1: Tại sao chọn Serverless thay vì server truyền thống?
**A:** Chi phí gần 0 (pay-per-invocation), tự động scale, không cần quản lý server/patch OS.

### Q2: AI severity có đáng tin không? Có thể bị đánh lừa không?
**A:** AI là non-deterministic, giảm thiểu bằng `temperature=0.1`. Hệ thống thiết kế theo Human-in-the-Loop — AI chỉ đề xuất, con người quyết định. Adversarial attacks (VD: user agent giả) là hạn chế đã nhận diện.

### Q3: Tại sao không dùng Chronicle mà chuyển sang BigQuery?
**A:** Chronicle yêu cầu license Enterprise rất đắt. BigQuery là bằng chứng thực nghiệm cho logic YARA-L. Thừa nhận SQL chỉ mô phỏng gần đúng.

### Q4: Nếu Gemini và OpenAI đều lỗi thì sao?
**A:** Cloud Function retry (tối đa 5 lần). Hướng cải tiến: thêm fallback rule-based.

### Q5: Tại sao chọn Telegram thay vì email/Slack?
**A:** Push notification tức thì, inline button (1 tap approve), miễn phí, đơn giản (chỉ cần token + chat ID).

### Q6: HMAC-SHA256 có đủ an toàn không?
**A:** Tiêu chuẩn công nghiệp (AWS Signature V4, Stripe, GitHub). Kết hợp link expiry + one-time-use + `hmac.compare_digest()`.

### Q7: Hệ thống xử lý false positive thế nào?
**A:** Human-in-the-Loop là lớp phòng thủ chính. AI cung cấp `reason` chi tiết. Chưa có cơ chế rollback tự động (hạn chế).

### Q8: So sánh với SOAR thương mại?
**A:** Hệ thống này là proof of concept, chi phí gần 0. Ưu: fully serverless, IaC, Dual-AI. Nhược: 1 kịch bản, chưa có case management.

### Q9: Log Sink hoạt động thế nào?
**A:** Bộ lọc trong Cloud Logging, lọc theo filter expression → push vào Pub/Sub → Cloud Function subscribe tự động.

### Q10: Alignment Period 60s nghĩa là gì?
**A:** Cửa sổ thời gian tối thiểu Cloud Monitoring gom metric. 60s là giá trị min GCP cho phép. Kèm cooldown 180s. Đây là giới hạn platform, không phải lỗi thiết kế.

---

## PHẦN 8: HẠN CHẾ & HƯỚNG PHÁT TRIỂN

### Hạn chế
1. Bottleneck Cloud Monitoring (96% T_response)
2. Non-deterministic AI severity
3. Một kịch bản duy nhất (Data Exfiltration)
4. Không có rollback tự động
5. Chronicle chưa triển khai thực tế

### Hướng phát triển
1. Real-time pipeline (bỏ qua Cloud Monitoring → T_response <30s)
2. Multi-scenario (IAM abuse, network anomaly)
3. Case management (Jira, ServiceNow)
4. Rollback mechanism
5. Chronicle integration khi có license

---

## PHẦN 9: BẢNG THUẬT NGỮ NHANH

| Thuật ngữ | Giải thích |
|---|---|
| **SOC** | Security Operations Center |
| **SIEM** | Security Information and Event Management |
| **SOAR** | Security Orchestration, Automation and Response |
| **MTTD/MTTR** | Mean Time To Detect / Respond |
| **UDM** | Unified Data Model (Chronicle) |
| **YARA-L** | Ngôn ngữ viết luật phát hiện của Chronicle |
| **IaC** | Infrastructure as Code |
| **Pub/Sub** | Publish/Subscribe messaging |
| **SCC** | Security Command Center |
| **MITRE ATT&CK** | Framework phân loại kỹ thuật tấn công |
| **HMAC** | Hash-based Message Authentication Code |
| **Honeypot** | Tài nguyên bẫy phát hiện xâm nhập |
| **Dual-AI Fallback** | Primary (Gemini) + Backup (OpenAI) |
| **Human-in-the-Loop** | Con người tham gia vòng lặp quyết định |
| **Defense-in-Depth** | Bảo mật nhiều lớp chồng nhau |
| **T1020** | Automated Exfiltration |
| **T1078** | Valid Accounts |
| **T1528** | Steal Application Access Token |
| **T1530** | Data from Cloud Storage |

---

> **💡 Mẹo khi gặp phản biện:**
> 1. **Thừa nhận hạn chế trước** khi bị hỏi → thể hiện sự trung thực
> 2. **Luôn kèm hướng khắc phục** khi nói về hạn chế
> 3. **Dùng số liệu cụ thể:** 265 giây, 96% bottleneck, 18/18 detection rate
> 4. **Phân biệt rõ** giới hạn platform vs giới hạn thiết kế
> 5. **Nhấn mạnh Human-in-the-Loop** — đây là điểm sáng nhất của đề tài
