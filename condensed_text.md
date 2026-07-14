# ✂️ Hướng dẫn rút gọn báo cáo FINAL (145 → 110 trang)

## Chiến lược 3 mũi

| Mũi | Hành động | Tiết kiệm |
|---|---|---|
| **A** | Rút gọn Chương 2 (viết lại ngắn gọn) | ~12p |
| **B** | Chuyển hình/bảng/YARA-L sang Phụ lục | ~17p |
| **C** | Bỏ trùng lặp Ch3↔Ch4 | ~7p |
| | **Tổng** | **~36p** |

---

# MŨI A — RÚT GỌN CHƯƠNG 2 (~12 trang)

Thao tác: XÓA đoạn cũ → PASTE đoạn mới bên dưới.

---

## A1. Mục 2.1.2 — Chức năng SOC | Tiết kiệm ~1p

**XÓA** para 377-412. **PASTE:**

> **2.1.2 Chức năng và quy trình hoạt động của SOC.**
>
> Quy trình hoạt động của SOC tuân theo bốn giai đoạn tuần tự:
>
> **Giai đoạn 1 — Thu thập và tổng hợp dữ liệu:** SOC thu thập log từ toàn bộ hạ tầng CNTT thông qua hệ thống SIEM. Dữ liệu được chuẩn hóa (normalization) từ các định dạng khác nhau về một mô hình chung và lưu trữ tập trung.
>
> **Giai đoạn 2 — Phát hiện và phân tích:** Hệ thống phân tích dữ liệu bằng ba phương pháp: so khớp mẫu tấn công đã biết (signature-based), phân tích hành vi bất thường bằng AI/ML (behavioral analysis), và tương quan dữ liệu đa nguồn (correlation).
>
> **Giai đoạn 3 — Sàng lọc và điều tra (Triage):** SOC Analyst xác minh cảnh báo là true positive hay false positive, đánh giá mức độ nghiêm trọng, và truy vết phạm vi ảnh hưởng.
>
> **Giai đoạn 4 — Phản ứng và khôi phục:** Cô lập hệ thống bị ảnh hưởng, tiêu diệt mối đe dọa, khôi phục từ bản sao lưu, và tự động hóa phản ứng đơn giản thông qua SOAR.

---

## A2. Mục 2.1.3 — Thách thức SOC | Tiết kiệm ~0.5p

**XÓA** para 413-433. **PASTE:**

> **2.1.3 Thách thức của SOC truyền thống.**
>
> SOC truyền thống đối mặt bốn thách thức: (1) *Quá tải dữ liệu* — hạ tầng on-premise giới hạn dung lượng, chi phí SIEM tính theo EPS buộc loại bỏ log, tạo "điểm mù"; (2) *Alert Fatigue* — rule-based sinh hàng nghìn cảnh báo/ngày, 80-99% là dương tính giả; (3) *Thiếu nhân lực* — 59% tổ chức thiếu nhân sự an ninh mạng (ISC², 2025), tỷ lệ burnout cao; (4) *Phản ứng chậm* — công cụ rời rạc, MTTD trung vị toàn cầu 11 ngày [5]. Những thách thức này đặt ra nhu cầu chuyển đổi sang nền tảng tích hợp SIEM–SOAR–AI.

---

## A3. Mục 2.2.2 — Kiến trúc SIEM | Tiết kiệm ~2p

**XÓA** para 468-563. **PASTE:**

> **2.2.2 Kiến trúc và chức năng chính của SIEM.**
>
> Kiến trúc SIEM hiện đại gồm bốn chức năng cốt lõi:
>
> **Thu thập và chuẩn hóa dữ liệu.** SIEM thu nhận log từ đa nguồn (firewall, server, endpoint, cloud) qua Syslog, API và agent. Dữ liệu thô được parser chuẩn hóa thành định dạng chung, sau đó được làm giàu (enrichment) bằng GeoIP, threat intelligence và asset inventory. Bước làm giàu dữ liệu này là nguyên lý mà đề tài áp dụng trong pipeline Context Enrichment (Mục 3.4.2).
>
> **Lưu trữ và đánh chỉ mục.** Dữ liệu được đánh chỉ mục để truy vấn nhanh trên hàng tỷ dòng log, phân tầng lưu trữ (hot/warm/cold) theo tần suất truy cập, và đảm bảo tính toàn vẹn WORM (Write Once, Read Many) cho điều tra số.
>
> **Phân tích và tương quan sự kiện.** Chức năng cốt lõi — sử dụng correlation rules để xâu chuỗi sự kiện rời rạc từ nhiều nguồn. SIEM thế hệ mới bổ sung UEBA và risk scoring để giảm alert fatigue.
>
> **Sinh cảnh báo và trực quan hóa.** Khi phát hiện trùng khớp luật, SIEM sinh alert kèm severity và ngữ cảnh, gửi thông báo qua email/SMS/ticketing. Dashboard trực quan hóa trạng thái an ninh theo thời gian thực.

---

## A4. Mục 2.2.3 — Vai trò SIEM | Tiết kiệm ~0.8p

**XÓA toàn bộ** mục 2.2.3 (para 564-583). Thêm 1 câu vào cuối 2.2.2:

> Trong SOC, SIEM đóng ba vai trò then chốt: trung tâm dữ liệu an ninh, hỗ trợ phát hiện sớm qua tương quan thời gian thực, và phục vụ điều tra số với dữ liệu lịch sử toàn vẹn.

---

## A5. Mục 2.2.4 — Hạn chế SIEM | Tiết kiệm ~0.5p

**XÓA** para 584-605. **PASTE:**

> **2.2.4 Hạn chế của SIEM truyền thống.**
>
> SIEM truyền thống có ba hạn chế chính: (1) *Chi phí phi tuyến tính* — tính phí theo EPS/GB buộc loại bỏ log, tạo "điểm mù"; (2) *Kiến trúc không co giãn* — on-premise khó mở rộng khi log tăng đột biến; (3) *Phụ thuộc rule tĩnh* — bất lực trước zero-day, sinh nhiều false positive do thiếu ngữ cảnh. Đây là động lực cho các nền tảng SIEM cloud-native như Google Security Operations (Mục 2.4).

---

## A6. Mục 2.3.3 — SIEM-SOAR-SOC | Tiết kiệm ~1p

**XÓA** para 651-698. **PASTE:**

> **2.3.3 Mối quan hệ SIEM – SOAR – SOC.**
>
> SIEM, SOAR và SOC tạo thành hệ sinh thái cộng sinh: SIEM là "bộ não" phát hiện — thu thập, tương quan và sinh cảnh báo; SOAR là "cánh tay" phản ứng — tự động làm giàu ngữ cảnh, sàng lọc false positive, thực thi playbook; SOC là tổ chức vận hành nơi con người giám sát và ra quyết định trong tình huống phức tạp. AI đóng vai trò chất xúc tác ở cả ba tầng: UEBA cho SIEM, gợi ý playbook cho SOAR, và ngôn ngữ tự nhiên cho analyst. Đề tài hiện thực hóa mối quan hệ này qua kiến trúc serverless: Cloud Monitoring (SIEM), Cloud Functions + AI triage (SOAR), và Telegram Bot + Human-in-the-loop (SOC).

---

## A7. Mục 2.5 — UDM | Tiết kiệm ~1.5p

- **2.5.1:** Rút còn 1 đoạn ngắn nêu vấn đề log không chuẩn hóa
- **2.5.3:** Bỏ liệt kê chi tiết từng trường, giữ mô tả tổng quan cấu trúc phân cấp
- **2.5.4:** Gộp vào cuối 2.5.3 thành 1 câu kết luận

---

## A8. Mục 2.6.5 — Ưu/nhược YARA-L | Tiết kiệm ~0.8p

**XÓA** para 960-986. **PASTE bảng:**

> **2.6.5 Ưu điểm và hạn chế của YARA-L.**
>
> | Ưu điểm | Hạn chế |
> |---|---|
> | Cú pháp khai báo rõ ràng, dễ quản lý như mã nguồn (Detection-as-Code) | Phụ thuộc hoàn toàn vào hệ sinh thái Google Chronicle |
> | Xử lý dữ liệu lớn nhờ hạ tầng Google, tương quan thời gian thực trên petabyte | Rào cản nhập môn cao — phải nắm vững cấu trúc UDM |
> | Dễ chia sẻ, tái sử dụng, kế thừa tư duy từ YARA | Chất lượng rule phụ thuộc vào parser chuẩn hóa |
>
> Tư duy Detection-as-Code của YARA-L có thể chuyển đổi sang ngôn ngữ truy vấn khác — đây là cơ sở cho thử nghiệm YARA-L → SQL tại Mục 3.7.

---

## A9. Mục 2.7.2 — Gemini in Chronicle | Tiết kiệm ~0.5p

**XÓA** para 1011-1037. **PASTE:**

> **2.7.2 Gemini in Chronicle.**
>
> Gemini in Security Operations là trợ lý AI tích hợp vào Google Chronicle [11], cung cấp bốn tính năng: (1) tóm tắt sự cố tự động, (2) tìm kiếm bằng ngôn ngữ tự nhiên, (3) sinh luật YARA-L tự động, và (4) điều tra hội thoại. Đề tài kế thừa tư duy này trong pipeline AI triage (Mục 3.4.2) — sử dụng Gemini API độc lập để tóm tắt sự cố và gợi ý phản ứng.

---

## A10. Mục 2.7.3 — Đánh giá AI | Tiết kiệm ~0.5p

**XÓA** para 1038-1070. **PASTE:**

> **2.7.3 Đánh giá vai trò AI trong SOC.**
>
> AI mang lại ba lợi ích chiến lược [12][13]: tăng tốc phát hiện (giảm MTTD/MTTR), nâng cao độ chính xác (giảm false positive), và giải phóng sức lao động. Tuy nhiên, AI cũng tiềm ẩn rủi ro: hallucination, thiên lệch dữ liệu, tính "hộp đen", và nguy cơ data poisoning. Triết lý cốt lõi là *AI hỗ trợ chứ không thay thế con người* [12]. Đề tài hiện thực hóa qua cơ chế Human-in-the-loop (Mục 3.4.3): AI đánh giá và gợi ý, nhưng quyết định Approve/Reject thuộc về SOC Admin.

---

## A11. Mục 2.8 — Tổng kết | Tiết kiệm ~0.7p

**XÓA** 2.8.1 + 2.8.2. **PASTE:**

> **2.8 Tổng kết chương.**
>
> Chương này trình bày khung lý thuyết theo chuỗi logic: SOC → SIEM → SOAR → Google SecOps → UDM → YARA-L → AI. Mỗi khái niệm là tiền đề cho khái niệm tiếp theo, tạo nền tảng cho thiết kế hệ thống tại Chương 3: kiến trúc Cloud-Native định hình hạ tầng; UDM và YARA-L cung cấp phương pháp chuẩn hóa và phát hiện; Context Enrichment và Prompt Engineering được hiện thực hóa trong pipeline AI triage; SOAR với Human-in-the-loop là cơ sở cho luồng phản ứng sự cố.

---

# MŨI B — CHUYỂN SANG PHỤ LỤC (~17 trang)

Phụ lục KHÔNG tính vào giới hạn 110 trang.

---

## B1. Mục 3.7.3 + 3.7.4 → Phụ lục B | Tiết kiệm ~5p (text + 2 hình + code)

**CẮT** toàn bộ 3.7.3 (Off-Hours Access) và 3.7.4 (Suspicious Tool Access) → **DÁN** vào Phụ lục B.

Tại vị trí cũ (sau 3.7.2), thêm 1 đoạn chuyển tiếp trước 3.7.5:

> Ngoài luật Mass Download Detection, nhóm đã thiết kế thêm hai luật bổ sung — Off-Hours Access (phát hiện truy cập ngoài giờ hành chính) và Suspicious Tool Access (nhận diện công cụ tự động đáng ngờ). Chi tiết thiết kế, mã YARA-L, SQL tương đương và kết quả thực thi được trình bày tại **Phụ lục B**.

**Cấu trúc Phụ lục B:**
```
Phụ lục B: Luật phát hiện YARA-L bổ sung
  B.1 Luật 2: Off-Hours Access (nguyên văn 3.7.3 cũ)
  B.2 Luật 3: Suspicious Tool Access (nguyên văn 3.7.4 cũ)
```

---

## B2. Ảnh chụp mục 3.4 → Phụ lục C | Tiết kiệm ~2p

7 ảnh hiện tại → **giữ 3 ảnh** (pipeline tổng quan, Telegram alert mẫu, webhook) → **chuyển 4 ảnh** (JSON output, Terraform output, code snippet, v.v.) sang Phụ lục C.

---

## B3. Ảnh chụp mục 3.5 → Phụ lục C | Tiết kiệm ~1.5p

4 ảnh terminal → **giữ 1 ảnh** đại diện → **chuyển 3 ảnh** sang Phụ lục C.

---

## B4. Ảnh mục 3.6.3 → Gộp thành figure 2×2 | Tiết kiệm ~1p

4 ảnh Telegram (approve/reject/expired/tampered) → **gộp** thành 1 figure duy nhất có 4 panel (a)(b)(c)(d).

Trong Word: Insert Table 2×2 → đặt mỗi ảnh vào 1 cell → thu nhỏ → thêm caption.

---

## B5. Bảng 3.6.2: Gộp Table 13+14+15 → 1 bảng | Tiết kiệm ~2p

3 bảng riêng biệt + 3 đoạn nhận xét → **1 bảng tổng hợp + 1 đoạn kết luận:**

> Bảng 3.x: So sánh ảnh hưởng từng lớp enrichment đến severity
>
> | Lớp | Cặp so sánh | Kịch bản A | Kịch bản B | Kết luận |
> |---|---|---|---|---|
> | **IP Geolocation** | gsutil + giờ HC | #1: VN → HIGH | #3: NL → CRITICAL | Ảnh hưởng mạnh |
> | | SDK + ngoài giờ | #7: VN → HIGH | #8: NL → CRITICAL | (3/3 nhất quán) |
> | | SDK + cuối tuần | #10: VN → HIGH | #11: JP → CRITICAL | |
> | **User Agent** | VN + giờ HC | #1: gsutil → HIGH | #2: SDK → CRITICAL ⚠ | Ảnh hưởng yếu |
> | | VN + ngoài giờ | #6: gsutil → HIGH | #7: SDK → HIGH | (không nhất quán) |
> | | VN + cuối tuần | #9: gsutil → CRIT | #10: SDK → HIGH ⚠ | |
> | **Time-of-Day** | VN + gsutil | #1: HC → HIGH | #9: Cuối tuần → CRIT | Ảnh hưởng TB |
> | | VN + SDK | #2: HC → CRIT ⚠ | #10: Cuối tuần → HIGH | (không nhất quán) |

Sau bảng: 1 đoạn kết luận thứ hạng IP Geo > Time > User Agent. **Giữ** Table 16 (xếp hạng, rất ngắn).

---

## B6. Ảnh 3.7 → Phụ lục B | Tiết kiệm ~1p

3 ảnh kết quả BigQuery → giữ 1 ảnh (Mass Download), chuyển 2 ảnh còn lại sang Phụ lục B cùng với luật tương ứng.

---

## B7. Bảng Ch2 không thiết yếu → Phụ lục | Tiết kiệm ~2p

- Bảng so sánh SOC truyền thống vs hiện đại (2.1) → Phụ lục
- Bảng cấu trúc UDM chi tiết (2.5) → Phụ lục
- Bảng so sánh SIEM vs SOAR (nếu có) → Phụ lục

---

# MŨI C — BỎ TRÙNG LẶP CH3↔CH4 (~7 trang)

---

## C1. Mục 4.1.2 — 8 mục tiêu → Bảng tóm tắt | Tiết kiệm ~3.5p

**XÓA** para 1803-1848. **PASTE:**

> **4.1.2 Mục tiêu thực tiễn.**
>
> | STT | Mục tiêu | Kết quả | Tham chiếu |
> |---|---|---|---|
> | 1 | Xây dựng môi trường trên Google Cloud | ✅ 7 module Terraform, tái tạo bằng 1 lệnh | Mục 3.2 |
> | 2 | Giả lập kịch bản tấn công | ✅ 18 kịch bản, ma trận 3 biến | Mục 3.5 |
> | 3 | Cơ chế phát hiện event-driven | ✅ Log Metric → Alert → Pub/Sub → Cloud Function | Mục 3.4.1 |
> | 4 | Tích hợp Context Enrichment | ✅ Pipeline 4 lớp (Logging, GeoIP, UA, Time) | Mục 3.4.2 |
> | 5 | Thử nghiệm AI phân tích sự cố | ✅ Dual-AI, Prompt Engineering, Structured Output | Mục 3.4.2, 4.3 |
> | 6 | SOAR với Human-in-the-loop | ✅ Orchestrator + Webhook, Semi-Automated | Mục 3.4.3 |
> | 7 | Cơ chế bảo mật | ✅ HMAC-SHA256, TTL, chống replay | Mục 3.6.3 |
> | 8 | YARA-L trên BigQuery | ✅ 3 luật YARA-L → SQL, khớp dữ liệu thực | Mục 3.7 |
>
> Đề tài hoàn thành 8/8 mục tiêu. Chi tiết kết quả đã trình bày tại các mục tham chiếu tương ứng.

---

## C2. Mục 4.3.1 — Context Enrichment | Tiết kiệm ~1p

Rút còn 3-4 dòng kết luận + tham chiếu mục 3.6.2. Xóa phần giải thích lại.

## C3. Mục 4.4.1 — Hạn chế | Tiết kiệm ~1p

Giữ danh sách bullet ngắn, xóa đoạn giải thích dài (đã giải thích ở Ch3).

---

# ✅ CHECKLIST TỔNG KẾT

| # | Mục | Hành động | Tiết kiệm |
|---|---|---|---|
| | **MŨI A — Rút gọn Ch2** | | |
| 1 | 2.1.2 + 2.1.3 | Rút gọn ví dụ | ~1.5p |
| 2 | 2.2.2 | Viết lại 4 chức năng | ~2p |
| 3 | 2.2.3 | Xóa, gộp 1 câu vào 2.2.2 | ~0.8p |
| 4 | 2.2.4 | Rút gọn 3 hạn chế | ~0.5p |
| 5 | 2.3.3 | Gộp 5 bước → 1 đoạn | ~1p |
| 6 | 2.5 | Gộp 2.5.1+2.5.2, rút 2.5.3+2.5.4 | ~1.5p |
| 7 | 2.6.5 | Bảng ưu/nhược | ~0.8p |
| 8 | 2.7.2 + 2.7.3 | Rút gọn ví dụ | ~1p |
| 9 | 2.8 | Gộp 2.8.1+2.8.2 | ~0.7p |
| | **MŨI B — Phụ lục** | | |
| 10 | 3.7.3 + 3.7.4 | 2 luật YARA-L → Phụ lục B | ~5p |
| 11 | 3.4 ảnh | 4/7 ảnh → Phụ lục C | ~2p |
| 12 | 3.5 ảnh | 3/4 ảnh → Phụ lục C | ~1.5p |
| 13 | 3.6.3 ảnh | Gộp 4 ảnh → figure 2×2 | ~1p |
| 14 | 3.6.2 bảng | Gộp Table 13+14+15 | ~2p |
| 15 | Ch2 bảng | 2-3 bảng lý thuyết → Phụ lục | ~2p |
| | **MŨI C — Bỏ trùng lặp** | | |
| 16 | 4.1.2 | Bảng tóm tắt 8 mục tiêu | ~3.5p |
| 17 | 4.3.1 + 4.4.1 | Bỏ trùng lặp với Ch3 | ~2p |
| | | **TỔNG** | **~36p** |

> [!IMPORTANT]
> Cấu trúc Phụ lục sau khi sửa:
> - Phụ lục A: Dữ liệu kiểm thử đầy đủ 18 kịch bản (đã có)
> - **Phụ lục B: Luật phát hiện YARA-L bổ sung** (Off-Hours + Suspicious Tool + 2 ảnh BigQuery)
> - **Phụ lục C: Hình ảnh minh họa bổ sung** (ảnh terminal, JSON output, Terraform...)
> - Phụ lục D: Bảng biểu tham khảo (bảng UDM chi tiết, so sánh SOC, v.v.) — tùy chọn

> [!CAUTION]
> **KHÔNG CẮT:** Dual-AI Fallback (4.3.3), Bottleneck Analysis (4.2.4), Luật Mass Download (3.7.2)
