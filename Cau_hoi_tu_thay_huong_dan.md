# CÂU HỎI TỪ THẦY HƯỚNG DẪN — CHUẨN BỊ BẢO VỆ

**Nguồn:** Buổi trao đổi với thầy Đỗ Hoàng Cương (02-03/08/2026)
**Lưu ý từ thầy:** *"Mấy em phải làm nổi bật giá trị học thuật, những ưu điểm và những chỗ thực nghiệm công phu. Hiểu biết ok hết rồi, cứ tự tin, từ tốn mà bảo vệ."*
**Tinh thần:** *"Sẵn sàng tự phản biện chính đề tài của mình để đưa ra hướng phát triển, các câu hỏi nghiên cứu mở — chứ không phải luôn luôn bảo vệ cái đúng."*

---

## Câu 1 ⭐ — Tumbling Window vs Hop Window

**Câu hỏi:** *"Giải thích rõ sự khác biệt giữa Tumbling Window (SQL) và Hop Window (YARA-L)."*

**Trả lời gợi ý:**

> *"Dạ, cả hai đều là cửa sổ thời gian dùng để gom sự kiện, nhưng khác nhau ở cách chia:*
>
> *Tumbling Window (cửa sổ cố định) — dùng trong SQL trên BigQuery — chia thời gian thành các khối liên tiếp không chồng lấn: 0-60s, 60-120s, 120-180s. Nếu kẻ tấn công tải 15 file ở giây 50-60 và 15 file ở giây 60-70, mỗi cửa sổ chỉ đếm 15 — không đạt ngưỡng 25 → bỏ sót.*
>
> *Hop Window (cửa sổ chồng lấn) — dùng trong YARA-L trên Chronicle — cửa sổ 60 giây nhưng trượt mỗi 30 giây: 0-60s, 30-90s, 60-120s. Cùng kịch bản trên, cửa sổ 30-90s sẽ đếm đủ 30 file → vượt ngưỡng → phát hiện.*
>
> *Đây là lý do em đề xuất trong hướng phát triển: triển khai YARA-L trên Chronicle khi có giấy phép để dùng hop window, khắc phục nhược điểm bỏ sót sự kiện ở biên."*

**Minh họa nhanh (vẽ trên bảng nếu cần):**
```
Tumbling:  |----W1----|----W2----|----W3----|
              0s        60s       120s

Hop:       |----W1---------|
                |----W2---------|
                    |----W3---------|
              0s   30s   60s   90s  120s
```

---

## Câu 2 — 18 lần thử nghiệm cùng 1 kịch bản

**Câu hỏi:** *"18 lần test nhưng hành vi cốt lõi vẫn chỉ là tải lậu dữ liệu từ Honeypot Bucket?"*

**Trả lời gợi ý:**

> *"Dạ đúng ạ. 18 lần thử nghiệm đều là kịch bản rút trộm dữ liệu — nhưng mục đích chính là cô lập biến số để đo lường ảnh hưởng của Context Enrichment lên đánh giá của AI. Cụ thể, em thay đổi 3 biến kiểm soát: công cụ (gsutil/Python SDK), IP (Việt Nam/nước ngoài), và thời gian (trong/ngoài giờ) — trong khi giữ nguyên hành vi tấn công. Nếu thay đổi cả kịch bản tấn công, sẽ không biết sự khác biệt trong đánh giá severity đến từ enrichment hay từ kịch bản khác nhau.*
>
> *Đây là hạn chế em ghi nhận: chưa mở rộng sang kịch bản leo thang đặc quyền hay di chuyển ngang — là hướng phát triển."*

**⚠️ Lưu ý từ thầy:** Cần bổ sung ý "cô lập biến số" ngắn gọn lên slide.

---

## Câu 3 ⭐ — HMAC-SHA256 và chặn tin nhắn Telegram

**Câu hỏi:** *"Tại sao chọn HMAC-SHA256? Nếu kẻ tấn công chặn (intercept) tin nhắn Telegram, chúng có thể sửa tham số URL để khóa tài khoản khác không?"*

**Trả lời gợi ý:**

> *"Dạ, HMAC-SHA256 được chọn vì là chuẩn công nghiệp, dùng rộng rãi trong xác thực API như AWS Signature V4, Stripe Webhooks.*
>
> *Nếu kẻ tấn công chặn được tin nhắn và sửa tham số — ví dụ đổi service_account_email sang tài khoản khác — chữ ký HMAC sẽ không khớp. Vì chữ ký được tính từ chuỗi `incident_id|service_account_email|severity|issued_at` — thay đổi bất kỳ trường nào đều làm chữ ký sai. Webhook kiểm tra bằng hàm `hmac.compare_digest`, nếu không khớp sẽ trả về lỗi PermissionError.*
>
> *Về khả năng chặn tin nhắn Telegram: Telegram mã hóa end-to-end cho Secret Chat, và mã hóa client-server cho chat thường. Bot API dùng HTTPS — dữ liệu được mã hóa trên đường truyền. Tuy không phải end-to-end, nhưng kẻ tấn công cần xâm nhập được server Telegram hoặc thiết bị của admin — đây là rủi ro rất thấp trong thực tế."*

**⚠️ Đính chính:** Câu trả lời ban đầu nói "bảo mật Telegram bật nhất" — chưa chính xác. Telegram Bot API **không** dùng end-to-end encryption, chỉ client-server TLS. Nên nhấn mạnh vào HMAC chống giả mạo thay vì dựa vào bảo mật Telegram.

---

## Câu 4 ⭐ — Giảm MTTD xuống dưới 10 giây

**Câu hỏi:** *"Nếu muốn giảm MTTD xuống dưới 10 giây (real-time tuyệt đối), sửa kiến trúc như thế nào? Đánh đổi gì?"*

**Trả lời gợi ý:**

> *"Dạ, chuyển từ Cloud Monitoring sang Log Router Sink đẩy log trực tiếp vào Cloud Function. Em đã thử nghiệm và MTTD giảm xuống khoảng 15 giây.*
>
> *Đánh đổi có 2 mặt:*
>
> *Về chi phí: Mỗi dòng audit log kích hoạt 1 Cloud Function riêng — nếu kẻ tấn công tải 100 file, hệ thống gọi 100 hàm + 100 lần Gemini API. Chi phí tăng tuyến tính theo số sự kiện thay vì gom lại 1 lần như Cloud Monitoring.*
>
> *Về độ chính xác: Log Sink không có khái niệm ngưỡng — mỗi sự kiện đơn lẻ đều kích hoạt. Tải 1 file cũng cảnh báo → dễ false positive. Cần thêm logic đếm trong Function hoặc dùng bộ nhớ tạm (Memorystore/Redis) để gom sự kiện trước khi cảnh báo — tăng độ phức tạp kiến trúc.*
>
> *Đề xuất: dùng kết hợp — Log Sink cho tài sản trọng yếu (Crown Jewel), Cloud Monitoring cho phần còn lại."*

---

## Câu 5 — Stateless + One-Time-Use Guard

**Câu hỏi:** *"Hệ thống serverless là stateless — làm sao biết sự cố đã được xử lý khi admin bấm Approve lần 2?"*

**Trả lời gợi ý:**

> *"Dạ, webhook kiểm tra trạng thái Service Account bằng hàm `_is_sa_already_disabled` — gọi IAM API lấy thuộc tính `disabled` của SA. Nếu `disabled == True`, nghĩa là đã được xử lý trước đó, webhook ngắt luồng ngay lập tức và trả về thông báo 'Already Remediated'.*
>
> *Cách tiếp cận này dùng chính trạng thái SA trên GCP làm cờ one-time-use — không cần database riêng. Ưu điểm: đơn giản, không tốn thêm hạ tầng. Hạn chế: nếu SA bị enable lại bởi người khác rồi link cũ được nhấn, link sẽ hoạt động lại — nhưng lúc đó link đã hết hạn TTL 1 giờ nên rủi ro rất thấp."*

---

## Câu 6 — Tại sao dùng null_resource thay vì google_storage_bucket_object

**Câu hỏi:** *"Tại sao tạo 55 file Honeypot bằng null_resource + gsutil chứ không dùng resource google_storage_bucket_object?"*

**Trả lời gợi ý:**

> *"Dạ, nếu dùng google_storage_bucket_object, Terraform sẽ quản lý từng file như một resource. Mỗi lần chạy `terraform plan` hoặc `terraform apply`, Terraform phải đọc trạng thái 55 file — tức gọi objects.get 55 lần — điều này kích hoạt Cloud Monitoring vượt ngưỡng 25 lần → gây false positive, hệ thống cảnh báo nhầm.*
>
> *null_resource chỉ chạy gsutil 1 lần lúc tạo, sau đó Terraform không theo dõi các file đó nữa. Bucket vẫn được quản lý bởi Terraform, nhưng các file bên trong thì không — tránh được false positive khi quản lý hạ tầng."*

---

## Câu 7 — Giả mạo User Agent

**Câu hỏi:** *"Nếu hacker đổi User Agent thành Google-Cloud-HealthCheck-Bot để giả dạng dịch vụ hệ thống, AI có bị đánh lừa hạ severity xuống LOW không?"*

**Trả lời gợi ý:**

> *"Dạ, đầu tiên về mặt kỹ thuật: User Agent được ghi nhận bởi Cloud Audit Log từ phía server — ghi lại giá trị mà client gửi lên. Kẻ tấn công dùng gcloud CLI hoặc SDK có thể đặt User Agent tùy ý, nên về lý thuyết có thể giả mạo.*
>
> *Tuy nhiên, AI của nhóm đã thể hiện tư duy suy luận tốt — như thầy đã nhận xét ở mục 3.6.2b trong báo cáo: 'Bất kể IP nội địa hay công cụ gì, hành vi tải hàng loạt từ Honeypot Bucket luôn là dấu hiệu bất thường nghiêm trọng.' Tức là AI xét tổng thể ngữ cảnh — honeypot + số lượng file + thời gian — chứ không chỉ dựa vào User Agent. Dù User Agent trông vô hại, việc tải 55 file từ honeypot vẫn bất thường → AI vẫn giữ severity HIGH/CRITICAL.*
>
> *Nhưng em thẳng thắn ghi nhận: đây là trường hợp chưa được kiểm chứng cụ thể trong 18 lần thử nghiệm — là hướng nghiên cứu mở."*

**⚠️ Đính chính:** Câu trả lời ban đầu nói "khả năng sửa User Agent gần như bằng 0" — **sai**. User Agent CÓ THỂ bị giả mạo. Điểm mạnh thực sự là AI xét tổng thể ngữ cảnh chứ không phụ thuộc 1 yếu tố.

---

## Câu 8 — 1.000 IP tấn công cùng lúc

**Câu hỏi:** *"Nếu hacker tấn công từ 1.000 IP khác nhau cùng lúc, Cloud Functions và Gemini API có bị Quota Limit hoặc bùng nổ chi phí không?"*

**Trả lời gợi ý:**

> *"Dạ, cần phân biệt 2 trường hợp:*
>
> *Nếu 1.000 IP dùng chung 1 Service Account bị lộ: Cloud Monitoring đếm theo số lần objects.get của SA đó — dù từ bao nhiêu IP, chỉ tạo 1 Incident duy nhất và trigger Cloud Function 1 lần. Khi Function chạy, nó query Audit Log lấy IP gần nhất. Chi phí không tăng, Gemini chỉ gọi 1 lần.*
>
> *Nếu 1.000 IP dùng 1.000 Service Account khác nhau: mỗi SA tạo metric riêng, Cloud Monitoring có thể tạo 1.000 Incident → 1.000 Cloud Function → 1.000 lần gọi Gemini. Lúc này sẽ bị Gemini free tier rate limit (khoảng 15 lượt/phút). Hệ thống fallback sang GPT cho các request bị từ chối. Nếu cần xử lý quy mô lớn, phải chuyển sang Gemini trả phí để nâng quota."*

---

## Câu 9 ⭐ — Rủi ro RCE trên Cloud Function

**Câu hỏi:** *"Service Account soar-orchestrator-sa có quyền iam.serviceAccountAdmin. Nếu Cloud Function bị RCE (Remote Code Execution), hacker chiếm được quyền gì?"*

**Trả lời gợi ý:**

> *"Dạ, em thừa nhận đây là rủi ro leo thang đặc quyền (Privilege Escalation). Nếu Cloud Function bị RCE, kẻ tấn công chiếm được SA có quyền iam.serviceAccountAdmin — có thể tạo, khóa, xóa bất kỳ Service Account nào trong project. Đây là quyền rộng và nguy hiểm.*
>
> *Biện pháp giảm thiểu theo nguyên tắc Least Privilege: thay vì cấp role iam.serviceAccountAdmin (bao gồm tạo/xóa), chỉ cấp quyền iam.serviceAccounts.disable duy nhất bằng custom role — đủ để khóa SA mà không cho phép tạo hay xóa. Ngoài ra, có thể bổ sung VPC Service Controls để giới hạn phạm vi truy cập của Function."*

---

## Câu 10 — Mở rộng sang Lateral Movement / Privilege Escalation

**Câu hỏi:** *"Mô hình này có hoạt động được với kịch bản di chuyển ngang (Lateral Movement) hay leo thang đặc quyền không?"*

**Trả lời gợi ý:**

> *"Dạ, kiến trúc event-driven có thể mở rộng — nhưng cần bổ sung cho từng kịch bản: Thứ nhất, viết thêm Cloud Function mới với logic enrichment phù hợp. Thứ hai, tạo log-based metric khác — ví dụ đếm setIamPolicy cho privilege escalation, hoặc phát hiện SA mới tạo bất thường cho lateral movement. Thứ ba, điều chỉnh prompt AI để phân tích đúng ngữ cảnh kịch bản mới.*
>
> *Tuy nhiên, mục đích 18 lần thử nghiệm là PoC chứng minh tính khả thi của pipeline — đo lường ảnh hưởng Context Enrichment trong 1 kịch bản cố định. Thời gian có hạn nên nhóm chưa xây dựng được SOC toàn diện — đây là hạn chế và hướng phát triển."*

---

## Câu 11 ⭐ — Gemini CRITICAL vs GPT LOW — nghe AI nào?

**Câu hỏi:** *"Nếu Gemini đánh giá CRITICAL (cần khóa ngay) mà khi Fallback sang GPT lại đánh giá LOW (bỏ qua), hệ thống nghe theo AI nào?"*

**Trả lời gợi ý:**

> *"Dạ, trường hợp xung đột này không xảy ra trong thiết kế hiện tại — vì hệ thống chỉ sử dụng 1 mô hình AI cho 1 lần phân tích: nếu Gemini thành công thì dùng kết quả Gemini, chỉ khi Gemini lỗi mới chuyển sang GPT. Không bao giờ có 2 kết quả cùng tồn tại.*
>
> *Tuy nhiên, trên tinh thần tự phản biện: giả sử thiết kế theo mô hình song song (chạy cả 2 rồi so sánh), thì cần cơ chế arbitration — ví dụ lấy mức severity cao hơn vì ưu tiên an toàn, hoặc chạy mô hình thứ 3 làm trọng tài. Đây là câu hỏi nghiên cứu mở.*
>
> *Nhưng quan trọng nhất: đây là mô hình Human-in-the-Loop — dù AI phán HIGH hay CRITICAL, quyền quyết định cuối cùng vẫn nằm ở nút bấm Approve của con người."*

**⚠️ Đính chính:** Câu trả lời ban đầu nói "khả năng xung đột là không thể" — **chưa chính xác về mặt học thuật**. Nên nói "không xảy ra trong thiết kế hiện tại" thay vì "không thể", và bổ sung tinh thần tự phản biện như thầy khuyên.

---

## Tóm tắt điểm cần đính chính

| # | Câu trả lời ban đầu (sai/thiếu) | Đính chính |
|---|---|---|
| 3 | "Bảo mật Telegram bật nhất" | Telegram Bot API chỉ dùng TLS, không end-to-end. Nhấn mạnh HMAC chống giả mạo |
| 7 | "Khả năng sửa User Agent gần như bằng 0" | User Agent CÓ THỂ bị giả mạo. Rào chắn là AI xét tổng thể ngữ cảnh |
| 11 | "Khả năng xung đột là không thể" | Nói "không xảy ra trong thiết kế hiện tại" + tự phản biện + human-in-the-loop |

---

## Việc cần làm trước bảo vệ

- [ ] Bổ sung ý "cô lập biến số để đo lường Context Enrichment" lên slide kết quả thực nghiệm
- [ ] Tập vẽ minh họa Tumbling vs Hop Window trên giấy/bảng
- [ ] Nhấn mạnh giá trị học thuật: pipeline SOAR serverless + AI kép + enrichment 4 lớp + IaC tái lập
- [ ] Ghi nhớ tinh thần: sẵn sàng tự phản biện → đưa ra hướng phát triển
