# KỊCH BẢN DEMO — V2 RÚT GỌN (~3.5 PHÚT)

> **Nguyên tắc:** Demo chỉ để **chứng minh hệ thống chạy thật**. Lý thuyết đã trình bày ở slide 5–9.

---

## Chuẩn bị trước khi quay

- [ ] Terminal: font ≥ 14pt, nền tối
- [ ] Tab 1: Terminal (chạy tấn công)
- [ ] Tab 2: GCP Console → Cloud Run → orchestrator-bot → Logs
- [ ] Tab 3: Telegram Web (hoặc điện thoại)
- [ ] Tab 4: GCP Console → IAM → Service Accounts
- [ ] Tab 5: GCP Console → Security Command Center → Findings
- [ ] Enable lại SA trước khi quay:
```
gcloud iam service-accounts enable victim-employee@linen-flash-490013-u3.iam.gserviceaccount.com --project=linen-flash-490013-u3
```

---

## CẢNH 1 — Tấn công (0:00 – 0:40)

📺 Màn hình: Terminal

🎙 *"Em bắt đầu demo. Giả định kẻ tấn công đã lấy được key JSON của Service Account từ nguồn lộ lọt. Em dùng key này đăng nhập và chạy script tải toàn bộ 55 file từ honeypot bucket."*

```
gcloud auth activate-service-account victim-employee@linen-flash-490013-u3.iam.gserviceaccount.com --key-file=victim_key.json

python attack_simulation.py
```

Chờ script chạy xong (55 files downloaded).

🎙 *"55 file đã tải xong. Mỗi file tạo 1 bản ghi audit log. Khi vượt ngưỡng 25 file trong 60 giây, Cloud Monitoring sẽ trigger cảnh báo. Hệ thống cần khoảng 4 phút để tổng hợp metric."*

⏱️ ~40 giây

---

## CẢNH 2 — Chờ + Cloud Function Logs (0:40 – 1:30)

📺 Màn hình: Chuyển sang GCP Console → Cloud Run → orchestrator-bot → Logs

🎙 *"Em mở log của Orchestrator Bot để theo dõi."*

> ⏸ **Tua nhanh / cắt ghép** phần chờ ~4 phút. Chèn text: *"~4 phút sau — Cloud Monitoring trigger alert"*

Khi log xuất hiện, **lướt nhanh** và chỉ vào 3 điểm:

🎙 *"Log cho thấy pipeline xử lý tuần tự:*
- *Enrichment 4 lớp — IP, location, tool, thời gian — hoàn tất*
- *Gemini 2.5 Flash phân tích — severity HIGH, confidence 0.9*
- *Gửi Telegram thành công*

*Toàn bộ pipeline chỉ mất khoảng 12 giây — đúng như em trình bày ở slide trước."*

⏱️ ~20 giây thực (sau khi cắt phần chờ)

---

## CẢNH 3 — Telegram + Approve (1:30 – 2:10)

📺 Màn hình: Chuyển sang Telegram

🎙 *"Đây là cảnh báo trên Telegram. AI đánh giá severity HIGH, confidence 0.9. Phần Reason cho thấy AI đã tổng hợp cả 4 lớp enrichment: IP Việt Nam, gcloud-python, thời gian truy cập, và honeypot bucket — để đưa ra kết luận.*

*Với vai trò SOC Admin, em nhấn Approve Remediation."*

**Nhấn nút Approve** → Chờ trang HTML hiện ra.

🎙 *"Remediation thành công: Service Account đã bị disabled, SCC Finding đã được tạo, và Audit Log đã được ghi nhận."*

⏱️ ~40 giây

---

## CẢNH 4 — Xác minh (2:10 – 3:10)

### 4a. Service Account disabled
📺 Màn hình: GCP Console → IAM → Service Accounts

🎙 *"Xác minh trên GCP Console: Service Account victim-employee đang ở trạng thái Disabled."*

⏱️ ~15 giây

### 4b. SCC Finding
📺 Màn hình: Security Command Center → Findings

🎙 *"Trên Security Command Center, Finding mới được tạo tự động — category Data Exfiltration, severity High, với MITRE ATT&CK mapping."*

⏱️ ~15 giây

### 4c. Hacker bị chặn
📺 Màn hình: Quay lại Terminal

```
gcloud storage ls
```

→ ERROR: The account for the specified service account is disabled.

🎙 *"Kẻ tấn công hoàn toàn bị chặn. Dù vẫn giữ key JSON, tài khoản đã bị vô hiệu hóa."*

⏱️ ~15 giây

---

## CẢNH 5 — Tổng kết (3:10 – 3:30)

🎙 *"Tóm lại, toàn bộ pipeline từ tấn công đến khóa tài khoản diễn ra trong khoảng 5 phút. 96% thời gian là do Cloud Monitoring tổng hợp metric. Pipeline ứng dụng — bao gồm enrichment và AI — chỉ mất 12 giây. Em xin kết thúc phần demo."*

⏱️ ~20 giây

---

## Tổng thời gian

| Cảnh | Nội dung | Thời gian |
|---|---|---|
| 1 | Tấn công | ~40 giây |
| 2 | Cloud Function Logs (sau cắt ghép) | ~20 giây |
| 3 | Telegram + Approve | ~40 giây |
| 4 | Xác minh (SA + SCC + chặn) | ~45 giây |
| 5 | Tổng kết | ~20 giây |
| **Tổng** | | **~2:45 – 3:30** |

---

## So sánh với bản cũ

| Phần | Bản cũ | Bản mới | Lý do cắt |
|---|---|---|---|
| Giới thiệu kiến trúc | ~3 phút | ❌ Cắt | Slide 5 đã trình bày |
| Terraform output | ~1 phút | ❌ Cắt | Backup B2 đã có |
| Do thám (gcloud compute) | ~1 phút | ❌ Cắt | Không cần thiết |
| Tấn công | ~2 phút | ~40 giây | Rút gọn lời thoại |
| Cloud Function Logs | ~3 phút | ~20 giây | Chỉ highlight 3 điểm |
| Telegram | ~2 phút | ~40 giây | Không đọc từng trường |
| Xác minh | ~2 phút | ~45 giây | Chỉ show kết quả |

---

## ⚠️ Sau khi quay xong

```powershell
# Enable lại SA
gcloud iam service-accounts enable victim-employee@linen-flash-490013-u3.iam.gserviceaccount.com --project=linen-flash-490013-u3
```

Kiểm tra video:
- [ ] Tiếng micro rõ ràng
- [ ] Font terminal đọc được
- [ ] Không lộ API key / secret
- [ ] Telegram hiển thị đầy đủ
- [ ] Trang Remediation Successful hiện rõ
