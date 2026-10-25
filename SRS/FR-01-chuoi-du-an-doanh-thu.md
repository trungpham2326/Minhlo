# Feature Spec: Chuỗi Dự án → BBNT → Hợp đồng → Hóa đơn → Thu tiền (kênh B), mốc và cảnh báo

**Mã**: FR-01  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-01)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: UC-driven (+US ánh xạ)  
**User Story liên quan**: US-01, US-02, US-03, US-04, US-05  
**Khó khăn nguồn**: KK01 · **Rx nguồn**: R2, R3, R9, R16, R18–R22, R25 · **Giả định liên quan**: G01, G09, G10, G11

> **Quy ước dùng trong file:** `Rx` là mã nguyên tử yêu cầu của Backbone mục 5.1 (không đánh số lại cục bộ). `[BA đề xuất]` là bổ sung của BA chưa nằm trong Backbone v1.1 (kèm mã tình huống `TH-..` ở `Tinh-huong-phat-sinh-va-edge-case.md`), chờ xác nhận. Tên trạng thái đúng với Phần A mục A.4.

---

## 1. Mục đích
Giúp mọi phòng cùng nhìn một dữ liệu để trả lời câu hỏi "dự án kênh B này đã xong, đã nghiệm thu, đã ký hợp đồng, đã xuất hóa đơn, đã thu tiền chưa, và đang kẹt ở ai". Tính năng giữ chuỗi doanh thu kênh B qua các lần bàn giao giữa Chuyển giao, Kinh doanh, Kế toán; mỗi mốc có một chủ và hệ thống nhắc, leo thang thay cho việc người trễ tự báo.

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Thông tin dự án kênh B (khách hàng, sản phẩm, loại dự án, căn cứ triển khai); bốn mốc ngày; BBNT (ngày ký, tệp); hợp đồng; đề nghị xuất hóa đơn; số hóa đơn, ngày, giá trị; ngày gửi hồ sơ; xác nhận khách nhận; hợp đồng gốc đã về; lần đòi nợ; thu tiền; tham số thời hạn |
| Xử lý | Quản lý trạng thái trục *Nghiệm thu & doanh thu* của Dự án; giao việc cho người phụ trách từng bước; kiểm tra điều kiện chuyển trạng thái; tính hạn theo tham số; tạo nhắc và leo thang; lưu vết |
| Output | Trạng thái dự án và người phụ trách hiện tại; việc trong "Việc của tôi"; cảnh báo quá hạn; hóa đơn, công nợ, thu tiền theo dự án; nhật ký lưu vết |

Sơ đồ: `[MODEL: State]` Dự án hai trục (Hình A.4a, A.4b) · `[MODEL: Sequence]` luồng kênh B (Hình A.3.1) · `[MODEL: ERD]` `DU_AN`, `BBNT`, `HOP_DONG`, `HOA_DON`, `THU_TIEN` (Hình A.5)

## 3. Use Case chi tiết

### UC-01.1 — Ghi mốc triển khai và BBNT
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Ghi mốc Vận hành, BBNT của dự án kênh B; chuyển trục doanh thu tới Chờ nghiệm thu, Đã nghiệm thu rồi nộp BBNT cho Kinh doanh |
| **Actor chính** | Trưởng nhóm triển khai (hoặc nhân viên Chuyển giao được giao) |
| **Actor phụ** | Bệnh viện (ký BBNT), Kinh doanh (nhận BBNT), Hệ thống |
| **Trigger** | Dự án xong kỹ thuật; bệnh viện ký BBNT |
| **Tiền điều kiện** | Dự án kênh B tồn tại, trục triển khai "Đang triển khai" |
| **Hậu điều kiện thành công** | Mốc và BBNT được ghi kèm người ghi và thời điểm; trục doanh thu đúng trạng thái; hạn nộp BBNT được tính |
| **Hậu điều kiện thất bại** | Không đổi trạng thái, không tạo bản ghi BBNT |
| **Business Rule liên quan** | BR-01.6, BR-01.7, BR-01.10, BR-01.11 |
| **Sub-feature liên quan** | FR-01.1, FR-01.2 |

**Luồng chính (Happy):**
1. Trưởng nhóm mở dự án và ghi mốc *Vận hành*. → 2. Hệ thống chuyển trục doanh thu sang *Chờ nghiệm thu*, đặt Trưởng nhóm là người phụ trách và bắt đầu đếm số ngày tồn. → 3. Bệnh viện ký BBNT; Trưởng nhóm tải tệp BBNT và nhập ngày ký. → 4. Hệ thống lưu BBNT, ghi mốc *Nghiệm thu* = ngày ký, chuyển trục doanh thu sang *Đã nghiệm thu* và tạo việc "Nộp BBNT cho Kinh doanh" với hạn 5 ngày làm việc kể từ ngày ký. → 5. Nhân viên Chuyển giao nộp BBNT; Kinh doanh xác nhận đã nhận (UC-01.2).

**Luồng phụ (Alternative):**
- 3a. Nghiệm thu theo giai đoạn: có nhiều BBNT cho một dự án `[BA đề xuất — TH-B10]`.
- 1a. Trưởng nhóm đổi giữa chừng: Trưởng phòng Chuyển giao đổi người; hệ thống ghi sự kiện, trạng thái giữ nguyên.

**Luồng ngoại lệ (Exception):**
- E1. (bước 3) Thiếu ngày ký hoặc tệp BBNT → hệ thống chặn, nêu trường thiếu.
- E2. (bước 1) Dự án không ở "Đang triển khai" (đang Tạm dừng) → hệ thống không cho ghi mốc Vận hành.
- E3. (bước 2) Chờ nghiệm thu quá hạn tham số → nhắc Trưởng nhóm, leo thang Trưởng phòng Chuyển giao.

### UC-01.2 — Hoàn tất hợp đồng sau nghiệm thu
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Kinh doanh nhận BBNT và ký hợp đồng trong hạn; dự án đấu thầu hoặc đã có hợp đồng bỏ qua bước này |
| **Actor chính** | Nhân viên Kinh doanh |
| **Actor phụ** | Nhân viên Chuyển giao, Trưởng phòng Kinh doanh, Hệ thống |
| **Trigger** | BBNT được nộp cho Kinh doanh |
| **Tiền điều kiện** | Trục doanh thu "Đã nghiệm thu"; dự án kênh B |
| **Hậu điều kiện thành công** | Hợp đồng ghi nhận đã ký; trục doanh thu "Chờ xuất hóa đơn" |
| **Hậu điều kiện thất bại** | Trục doanh thu giữ "Chờ hợp đồng"; cảnh báo quá hạn được phát |
| **Business Rule liên quan** | BR-01.12 đến BR-01.16 |
| **Sub-feature liên quan** | FR-01.2 |

**Luồng chính (Happy):**
1. Kinh doanh xác nhận đã nhận BBNT. → 2. Hệ thống ghi *ngày nhận*, chuyển trục doanh thu sang *Chờ hợp đồng*, tạo việc "Hoàn tất hợp đồng" cho nhân viên Kinh doanh phụ trách với hạn 10 ngày kể từ ngày nhận. → 3. Kinh doanh ký hợp đồng và cập nhật hợp đồng đã ký. → 4. Hệ thống chuyển trục doanh thu sang *Chờ xuất hóa đơn*.

**Luồng phụ (Alternative):**
- 2a. Dự án đấu thầu (hợp đồng có trước) → bỏ qua bước 2–4, sang *Chờ xuất hóa đơn* ngay khi BBNT được nhận.
- 2b. Dự án không đấu thầu nhưng hợp đồng đã ký trước đó → như 2a `[BA đề xuất — TH-B04]`.

**Luồng ngoại lệ (Exception):**
- E1. (bước 2) Quá 10 ngày chưa ký → hệ thống nhắc nhân viên, báo Trưởng phòng Kinh doanh, ghi vết, không phạt tự động.
- E2. (bước 1) Nhân viên Chuyển giao chưa nộp BBNT quá 5 ngày làm việc kể từ ngày ký → nhắc nhân viên Chuyển giao, báo Trưởng phòng Chuyển giao.

### UC-01.3 — Xuất hóa đơn, gửi hồ sơ và thu tiền
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Kinh doanh đề nghị xuất hóa đơn; Kế toán ghi hóa đơn; Kinh doanh gửi hồ sơ; Kế toán đòi nợ và ghi thu tiền |
| **Actor chính** | Nhân viên Kinh doanh, Kế toán |
| **Actor phụ** | Bệnh viện, Trưởng phòng Kinh doanh, Trưởng phòng Kế toán, Hệ thống |
| **Trigger** | Trục doanh thu vào "Chờ xuất hóa đơn" |
| **Tiền điều kiện** | Đã BBNT; đã có hợp đồng hoặc là dự án đấu thầu |
| **Hậu điều kiện thành công** | Trục doanh thu "Đã thu"; toàn bộ hóa đơn của dự án được thu đủ |
| **Hậu điều kiện thất bại** | Trục doanh thu dừng ở bước hiện tại; cảnh báo và leo thang theo mốc |
| **Business Rule liên quan** | BR-01.17 đến BR-01.26 |
| **Sub-feature liên quan** | FR-01.3, FR-01.4 |

**Luồng chính (Happy):**
1. Kinh doanh đề nghị xuất hóa đơn. → 2. Hệ thống kiểm tra điều kiện, giao việc cho Kế toán. → 3. Kế toán xuất hóa đơn trên phần mềm kế toán pháp định và ghi số hóa đơn, ngày, giá trị vào hệ thống; trục doanh thu sang *Đã xuất hóa đơn*. → 4. Kinh doanh gửi hồ sơ và hóa đơn, ghi ngày gửi; trục doanh thu sang *Đã gửi hồ sơ*; hệ thống tạo việc gọi xác nhận sau 1–2 ngày. → 5. Kinh doanh ghi khách đã nhận; trục doanh thu sang *Đang công nợ*. → 6. Cứ 15 ngày, hệ thống tạo việc đòi nợ cho Kế toán; Kế toán ghi từng lần đòi nợ. → 7. Khi thu đủ mọi hóa đơn, Kế toán ghi thu; trục doanh thu sang *Đã thu*.

**Luồng phụ (Alternative):**
- 3a. Thu từng phần, nhiều đợt: mỗi hóa đơn có nhiều lần thu; vẫn "Đang công nợ" đến khi thu đủ.
- 3b. Hóa đơn tạm ứng/theo mốc trước nghiệm thu `[BA đề xuất — TH-B14]`.

**Luồng ngoại lệ (Exception):**
- E1. (bước 1–2) Dự án chưa đủ điều kiện (chưa BBNT, hoặc chưa hợp đồng khi không đấu thầu) → hệ thống từ chối đề nghị, nêu điều kiện còn thiếu.
- E2. (bước 5) Quá 1–2 ngày chưa ghi kết quả gọi xác nhận → nhắc Kinh doanh.
- E3. (bước 5) Quá 2 tuần chưa nhận hợp đồng gốc → báo Trưởng phòng Kinh doanh (không chặn công nợ).
- E4. (bước 6) Đã đòi 3 lần chưa thu → báo Trưởng phòng Kế toán.

### UC-01.4 — Xem chuỗi dự án và biết đang kẹt ở ai
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Trưởng phòng/PGĐ mở dự án để biết bước hiện tại, người phụ trách và số ngày đã ở bước đó |
| **Actor chính** | Trưởng phòng, PGĐ |
| **Actor phụ** | Hệ thống |
| **Trigger** | Cần theo dõi hoặc nhắc một dự án |
| **Tiền điều kiện** | Người dùng có quyền xem dự án (phạm vi vai trò) |
| **Hậu điều kiện thành công** | Thấy đủ hai trục trạng thái, các mốc, người phụ trách, số ngày tồn |
| **Hậu điều kiện thất bại** | Không thấy dự án ngoài phạm vi quyền |
| **Business Rule liên quan** | BR-01.1, BR-01.2, BR-01.3, BR-01.9 |
| **Sub-feature liên quan** | FR-01.1 |

**Luồng chính (Happy):** 1. Người dùng mở dự án. → 2. Hệ thống hiển thị hai trục trạng thái, các mốc đã ghi, người phụ trách bước hiện tại và số ngày đã ở bước đó.
**Luồng phụ:** 2a. Dự án kênh A: trục doanh thu rẽ nhánh *Chờ đối soát* (chi tiết ở FR-02).
**Luồng ngoại lệ:** E1. (bước 1) Dự án ngoài phạm vi quyền → hệ thống không hiển thị.

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-01.1 | R2, R16, R18, R25 | Dự án kênh B, hai trục trạng thái, 4 mốc | Chưa ước lượng | Draft | |
| FR-01.2 | R19, R20, R25 | Nghiệm thu BBNT và hợp đồng (5 ngày, 10 ngày, đấu thầu) | Chưa ước lượng | Draft | |
| FR-01.3 | R9, R21 | Hóa đơn và hồ sơ gửi khách | Chưa ước lượng | Draft | |
| FR-01.4 | R22 | Công nợ và thu tiền | Chưa ước lượng | Draft | |

Cảnh báo, leo thang và "Việc của tôi" dùng cơ chế chung ở FR-05; phân quyền, tham số, lưu vết ở FR-06.

---

### FR-01.1 — Dự án kênh B, hai trục trạng thái, 4 mốc
- **Mô tả**: Tạo Dự án kênh B; quản lý trục *Triển khai & bàn giao* (phần bàn giao ở FR-04) và trục *Nghiệm thu & doanh thu*; ghi bốn mốc ngày; xem chuỗi.
- **Input – Xử lý – Output**: Input: khách hàng, sản phẩm, loại dự án, căn cứ triển khai, mốc ngày. Xử lý: kiểm tra điều kiện, chuyển trạng thái, đếm ngày tồn, lưu vết. Output: dự án với hai trục trạng thái, người phụ trách, mốc.
- **Màn hình UI**: `[WIREFRAME: Danh sách và chi tiết dự án]` · `[WIREFRAME: Ghi mốc và BBNT]`

**Luồng xử lý:**
- **Happy**: Kinh doanh tạo dự án (có căn cứ triển khai) → Chuyển giao điều động → Trưởng nhóm ghi mốc Bắt đầu, Vận hành → trục doanh thu sang Chờ nghiệm thu → người dùng có quyền xem chuỗi.
- **Exception**: thiếu trường bắt buộc; ghi mốc khi dự án đang Tạm dừng; người dùng ngoài phạm vi quyền.
- **Edge/Boundary**: đổi trưởng nhóm giữa chừng; dự án kênh A hiển thị trên cùng màn hình; sửa mốc đã ghi (tính lại số ngày); hai người cùng sửa một mốc.
- **Alternative**: dự án đấu thầu tạo kèm hợp đồng.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-01.1 | Kênh gắn ở Dự án; mỗi dự án thuộc đúng một kênh; FR-01 xử lý dự án kênh B (D1, R16) |
| BR-01.2 | Dự án có hai trục trạng thái độc lập: *Triển khai & bàn giao* và *Nghiệm thu & doanh thu*; dự án chỉ "Đóng" khi cả hai trục hoàn tất (D2) |
| BR-01.3 | Mỗi mốc có một chủ (người phụ trách bước hiện tại) và một nguồn ghi ngày; khi quá hạn hệ thống nhắc người phụ trách và báo Trưởng phòng; không phạt tự động (D3, R8) |
| BR-01.4 | Mọi thời hạn là tham số (mặc định ở bảng phụ lục BR-01.4) và đổi được không cần phát hành bản mới (R5, NFR-G7.1) |
| BR-01.5 | Tạo dự án kênh B bắt buộc: khách hàng, sản phẩm, loại dự án (thường/đấu thầu) và căn cứ triển khai (báo giá được chấp thuận, đơn đặt hàng hoặc thỏa thuận đã chốt); dự án đấu thầu bắt buộc có hợp đồng (G01) |
| BR-01.6 | Bốn mốc: Bắt đầu triển khai, Vận hành, Nghiệm thu (= ngày ký BBNT), Rút; mỗi mốc ghi người ghi và thời điểm; sửa mốc phải lưu vết trước/sau và tính lại số ngày (G09, NFR-G10.1) |
| BR-01.7 | Ghi mốc Vận hành chuyển trục doanh thu "Chưa phát sinh" → "Chờ nghiệm thu"; người phụ trách là Trưởng nhóm; hệ thống đếm số ngày tồn và cảnh báo khi quá tham số (G10) |
| BR-01.8 | Đổi trưởng nhóm là sự kiện ghi thời điểm, người cũ, người mới; trạng thái giữ nguyên; ngày làm việc ghi theo từng người |
| BR-01.9 | Người dùng chỉ xem/sửa dự án trong phạm vi vai trò và dữ liệu được cấp (NFR-G2.1) |

**Phụ lục BR-01.4 — tham số thời hạn mặc định (lấy từ quy định hiện hành, R19–R22):**
| Tham số | Giá trị mặc định | Ghi chú |
|---|---|---|
| Hạn nộp BBNT cho Kinh doanh | 5 ngày làm việc kể từ ngày ký | R19 |
| Hạn ký hợp đồng | 10 ngày kể từ ngày Kinh doanh nhận BBNT | R20; ngày lịch hay ngày làm việc `[CẦN LÀM RÕ VỚI KH]` |
| Gọi xác nhận sau khi gửi hồ sơ | 1–2 ngày | R21 |
| Chờ hợp đồng gốc | 2 tuần | R21 |
| Nhịp đòi nợ | 15 ngày/lần; báo sau 3 lần | R22 |
| Hạn Chờ nghiệm thu (nhắc) | `[CẦN LÀM RÕ VỚI KH]` | Đề không nêu (G10) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-01.1.1 | Happy | BR-01.1, BR-01.5 | UC-01.4 | **Given** Kinh doanh có căn cứ triển khai **When** tạo dự án với khách hàng, sản phẩm, loại thường và căn cứ **Then** dự án ở "Mới tạo" (trục triển khai), trục doanh thu "Chưa phát sinh", kênh là B |
| AC-01.1.2 | Exception | BR-01.5 | UC-01.4 | **Given** dự án không đấu thầu và chưa có căn cứ triển khai **When** lưu **Then** hệ thống chặn và nêu trường còn thiếu |
| AC-01.1.3 | Happy | BR-01.6, BR-01.7 | US-02 A1 / UC-01.1 | **Given** dự án "Đang triển khai" **When** Trưởng nhóm ghi mốc Vận hành **Then** trục doanh thu chuyển "Chờ nghiệm thu", người phụ trách là Trưởng nhóm, số ngày tồn bắt đầu đếm |
| AC-01.1.4 | Exception | BR-01.6 | UC-01.1 E2 | **Given** dự án đang "Tạm dừng" **When** Trưởng nhóm ghi mốc Vận hành **Then** hệ thống không cho ghi và nêu lý do |
| AC-01.1.5 | Edge | BR-01.8 | US-02 A3 / UC-01.1 | **Given** dự án đổi trưởng nhóm giữa chừng **When** Trưởng phòng Chuyển giao đổi **Then** hệ thống ghi sự kiện (thời điểm, người cũ, người mới), trạng thái không đổi, ngày làm việc ghi theo từng người |
| AC-01.1.6 | Happy | BR-01.2, BR-01.3 | US-01 A1 / UC-01.4 | **Given** dự án kênh B ở trục doanh thu "Chờ hợp đồng", người phụ trách là Kinh doanh K **When** Trưởng phòng mở chi tiết **Then** hiển thị hai trục trạng thái, các mốc, người phụ trách bước hiện tại và số ngày đã ở bước đó |
| AC-01.1.7 | Edge | BR-01.1 | US-01 A2 / UC-01.4 | **Given** dự án thuộc kênh A **When** mở chi tiết **Then** trục doanh thu rẽ nhánh "Chờ đối soát" và không hiển thị bước hợp đồng, hóa đơn của kênh B |
| AC-01.1.8 | Exception | BR-01.9 | US-01 A3 / UC-01.4 | **Given** người dùng chỉ có quyền trong phạm vi phòng mình **When** tìm dự án ngoài phạm vi **Then** hệ thống không hiển thị dự án đó |
| AC-01.1.9 | Edge | BR-01.6 | UC-01.1 | **Given** một mốc đã ghi **When** người có quyền sửa ngày mốc **Then** hệ thống lưu giá trị trước/sau, người sửa, thời điểm và tính lại số ngày tồn |

---

### FR-01.2 — Nghiệm thu BBNT và hợp đồng
- **Mô tả**: Ghi BBNT, nộp cho Kinh doanh, Kinh doanh nhận và hoàn tất hợp đồng trong hạn; xử lý dự án đấu thầu.
- **Input – Xử lý – Output**: Input: BBNT (ngày ký, tệp), xác nhận nhận, hợp đồng đã ký. Xử lý: chuyển trạng thái, tính hạn 5 và 10 ngày, giao việc, cảnh báo. Output: trục doanh thu "Đã nghiệm thu" → "Chờ hợp đồng" → "Chờ xuất hóa đơn", việc và cảnh báo.
- **Màn hình UI**: `[WIREFRAME: Ghi BBNT]` · `[WIREFRAME: Hoàn tất hợp đồng]`

**Luồng xử lý:**
- **Happy**: Chuyển giao ghi BBNT → nộp trong 5 ngày → Kinh doanh nhận → ký hợp đồng trong 10 ngày → "Chờ xuất hóa đơn".
- **Exception**: thiếu ngày ký/tệp; quá hạn nộp; quá hạn ký.
- **Edge/Boundary**: đúng ngày thứ 5 và thứ 10; dự án đấu thầu; hợp đồng đã ký trước dù không đấu thầu; nhiều BBNT.
- **Alternative**: nghiệm thu theo giai đoạn `[BA đề xuất]`.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-01.10 | BBNT bắt buộc có ngày ký và tệp; mốc Nghiệm thu bằng ngày ký (G09) |
| BR-01.11 | Chuyển giao nộp BBNT cho Kinh doanh trong 5 ngày làm việc kể từ ngày ký (mặc định, tham số); quá hạn → nhắc nhân viên Chuyển giao và báo Trưởng phòng Chuyển giao (R19) |
| BR-01.12 | Trục doanh thu chuyển "Đã nghiệm thu" → "Chờ hợp đồng" khi Kinh doanh xác nhận đã nhận BBNT (kênh B không đấu thầu, chưa có hợp đồng); ngày nhận là mốc tính hạn 10 ngày (G09) |
| BR-01.13 | Dự án đấu thầu (hợp đồng có trước): bỏ qua "Chờ hợp đồng", sang "Chờ xuất hóa đơn" khi BBNT được nhận (R20, G01) |
| BR-01.14 | `[BA đề xuất — TH-B04]` Dự án không đấu thầu nhưng hợp đồng đã ký tại thời điểm nhận BBNT: tự bỏ qua "Chờ hợp đồng" |
| BR-01.15 | Kinh doanh ký hợp đồng trong 10 ngày kể từ ngày nhận BBNT; quá hạn → nhắc nhân viên và báo Trưởng phòng Kinh doanh, ghi vết (R20, R8) |
| BR-01.16 | `[BA đề xuất — TH-B10]` Nghiệm thu theo giai đoạn: một dự án có nhiều BBNT; hóa đơn theo đợt gắn BBNT từng đợt; trạng thái trục doanh thu của dự án lấy theo đợt chậm nhất |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-01.2.1 | Happy | BR-01.10, BR-01.11 | US-02 A1 / UC-01.1 | **Given** trục doanh thu "Chờ nghiệm thu" **When** Trưởng nhóm ghi BBNT kèm ngày ký và tệp **Then** trục doanh thu chuyển "Đã nghiệm thu", mốc Nghiệm thu bằng ngày ký, và hệ thống tạo việc "Nộp BBNT cho Kinh doanh" với hạn 5 ngày làm việc kể từ ngày ký |
| AC-01.2.2 | Exception | BR-01.10 | US-02 A2 / UC-01.1 E1 | **Given** BBNT chưa có ngày ký **When** lưu **Then** hệ thống chặn và yêu cầu nhập ngày ký |
| AC-01.2.3 | Exception | BR-01.11 | UC-01.2 E2 | **Given** quá 5 ngày làm việc kể từ ngày ký mà BBNT chưa nộp **When** đến hạn **Then** hệ thống nhắc nhân viên Chuyển giao phụ trách, báo Trưởng phòng Chuyển giao và ghi vết |
| AC-01.2.4 | Happy | BR-01.12, BR-01.15 | US-03 A1 / UC-01.2 | **Given** BBNT của dự án kênh B không đấu thầu đã được nộp **When** Kinh doanh xác nhận đã nhận **Then** trục doanh thu chuyển "Chờ hợp đồng" và hệ thống tạo việc "Hoàn tất hợp đồng" cho nhân viên phụ trách với hạn 10 ngày kể từ ngày nhận, hiện trong "Việc của tôi" |
| AC-01.2.5 | Exception | BR-01.15 | US-03 A2 / UC-01.2 E1 | **Given** việc hoàn tất hợp đồng đã quá 10 ngày **When** đến hạn **Then** hệ thống nhắc nhân viên, báo Trưởng phòng Kinh doanh, ghi vết và không phạt tự động |
| AC-01.2.6 | Edge | BR-01.13 | US-03 A3 / UC-01.2 | **Given** dự án đấu thầu **When** BBNT được nhận **Then** hệ thống không tạo việc hoàn tất hợp đồng và chuyển thẳng "Chờ xuất hóa đơn" |
| AC-01.2.7 | Edge | BR-01.14 | UC-01.2 | **Given** dự án không đấu thầu nhưng hợp đồng đã ký trước **When** Kinh doanh nhận BBNT **Then** hệ thống bỏ qua "Chờ hợp đồng" và chuyển "Chờ xuất hóa đơn" `[BA đề xuất]` |
| AC-01.2.8 | Edge | BR-01.16 | UC-01.1 | **Given** dự án nghiệm thu theo hai giai đoạn **When** đợt 1 có BBNT, đợt 2 chưa **Then** trục doanh thu của dự án theo đợt chậm nhất và hóa đơn đợt 1 vẫn được đề nghị `[BA đề xuất]` |
| AC-01.2.9 | Edge | BR-01.15 | UC-01.2 | **Given** hạn 10 ngày rơi vào ngày nghỉ **When** hệ thống tính hạn **Then** áp quy ước ngày lịch hoặc ngày làm việc theo tham số `[CẦN LÀM RÕ VỚI KH]` |

---

### FR-01.3 — Hóa đơn và hồ sơ gửi khách
- **Mô tả**: Kinh doanh đề nghị xuất hóa đơn; Kế toán ghi hóa đơn; Kinh doanh gửi hồ sơ, gọi xác nhận, theo dõi hợp đồng gốc.
- **Input – Xử lý – Output**: Input: đề nghị xuất hóa đơn, số hóa đơn/ngày/giá trị, ngày gửi hồ sơ, xác nhận khách nhận, cờ hợp đồng gốc. Xử lý: kiểm tra điều kiện, giao việc, tính hạn 1–2 ngày và 2 tuần. Output: trục doanh thu "Đã xuất hóa đơn" → "Đã gửi hồ sơ" → "Đang công nợ".
- **Màn hình UI**: `[WIREFRAME: Đề nghị xuất hóa đơn]` · `[WIREFRAME: Ghi hóa đơn]` · `[WIREFRAME: Gửi hồ sơ và xác nhận]`

**Luồng xử lý:**
- **Happy**: Kinh doanh đề nghị → Kế toán ghi hóa đơn → Kinh doanh gửi hồ sơ → gọi xác nhận → "Đang công nợ".
- **Exception**: đề nghị khi chưa đủ điều kiện; quá hạn gọi xác nhận; quá 2 tuần chưa có hợp đồng gốc.
- **Edge/Boundary**: nhiều hóa đơn theo đợt; hóa đơn hủy/điều chỉnh; hóa đơn tạm ứng.
- **Alternative**: kênh A xuất hóa đơn cho đối tác (FR-02).

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-01.17 | Chỉ nhận đề nghị xuất hóa đơn khi dự án ở "Chờ xuất hóa đơn" (đã BBNT và đã có hợp đồng hoặc là dự án đấu thầu); không đủ thì từ chối và nêu điều kiện thiếu (G11) |
| BR-01.18 | Kế toán ghi số hóa đơn, ngày, giá trị; hệ thống không phát hành hóa đơn pháp định và không thay phần mềm kế toán (R9, G11) |
| BR-01.19 | Sau khi ghi hóa đơn, Kinh doanh gửi hồ sơ và hóa đơn và ghi ngày gửi; hệ thống tạo việc gọi xác nhận khách đã nhận sau 1–2 ngày (R21) |
| BR-01.20 | Hợp đồng gốc có cờ "đã về"; quá 2 tuần chưa về → báo Trưởng phòng Kinh doanh; cờ không chặn tiến trình công nợ (R21) |
| BR-01.21 | `[BA đề xuất — TH-B06]` Hóa đơn có trạng thái (hiệu lực, đã điều chỉnh, đã hủy) và liên kết hóa đơn thay thế; hóa đơn pháp định vẫn ở phần mềm kế toán |
| BR-01.22 | `[BA đề xuất — TH-B14]` Hóa đơn tạm ứng/theo mốc là loại riêng; PGĐ Tài chính duyệt ngoại lệ có căn cứ trước khi chấp nhận đề nghị khi chưa có BBNT |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-01.3.1 | Happy | BR-01.17, BR-01.18 | US-05 A1 / UC-01.3 | **Given** dự án kênh B đã có BBNT và hợp đồng đã ký (hoặc đấu thầu) **When** Kế toán nhận đề nghị và ghi số hóa đơn, ngày, giá trị **Then** trục doanh thu chuyển "Đã xuất hóa đơn" và hệ thống không phát hành hóa đơn pháp định |
| AC-01.3.2 | Exception | BR-01.17 | US-05 A2 / UC-01.3 E1 | **Given** dự án chưa có BBNT **When** Kinh doanh gửi đề nghị xuất hóa đơn **Then** hệ thống từ chối và nêu điều kiện còn thiếu |
| AC-01.3.3 | Happy | BR-01.19 | US-04 A1 / UC-01.3 | **Given** hóa đơn đã được ghi **When** Kinh doanh ghi "Đã gửi hồ sơ" **Then** hệ thống tạo việc gọi xác nhận trong 1–2 ngày; khi ghi khách đã nhận, trục doanh thu chuyển "Đang công nợ" |
| AC-01.3.4 | Exception | BR-01.20 | US-04 A2 / UC-01.3 E3 | **Given** sau 2 tuần chưa nhận lại hợp đồng gốc **When** đến hạn **Then** hệ thống báo Trưởng phòng Kinh doanh; cờ "hợp đồng gốc đã về" chưa đánh dấu và không chặn công nợ |
| AC-01.3.5 | Exception | BR-01.19 | US-04 A3 / UC-01.3 E2 | **Given** quá hạn gọi xác nhận mà chưa ghi kết quả **When** đến hạn **Then** hệ thống nhắc nhân viên Kinh doanh phụ trách |
| AC-01.3.6 | Edge | BR-01.21 | UC-01.3 | **Given** một hóa đơn đã ghi bị hủy hoặc điều chỉnh **When** Kế toán ghi hóa đơn thay thế **Then** hóa đơn cũ ở trạng thái "đã hủy/đã điều chỉnh", liên kết hóa đơn mới và công nợ tính theo hóa đơn hiệu lực `[BA đề xuất]` |
| AC-01.3.7 | Edge | BR-01.22 | UC-01.3 | **Given** khách yêu cầu hóa đơn tạm ứng trước nghiệm thu **When** Kinh doanh gửi đề nghị kèm căn cứ **Then** đề nghị chờ PGĐ Tài chính duyệt ngoại lệ; chưa duyệt thì hệ thống từ chối `[BA đề xuất]` |

---

### FR-01.4 — Công nợ và thu tiền
- **Mô tả**: Theo dõi công nợ theo hóa đơn, nhắc đòi nợ 15 ngày/lần, báo sau 3 lần, ghi thu tiền và đóng trục doanh thu.
- **Input – Xử lý – Output**: Input: lần đòi nợ, số tiền thu. Xử lý: tạo việc đòi nợ theo nhịp, đếm số lần, cập nhật trạng thái. Output: trục doanh thu "Đang công nợ" → "Đã thu"; cảnh báo cho Trưởng phòng Kế toán.
- **Màn hình UI**: `[WIREFRAME: Công nợ theo dự án]` · `[WIREFRAME: Ghi thu tiền]`

**Luồng xử lý:**
- **Happy**: nhịp 15 ngày → Kế toán đòi nợ và ghi → thu đủ → "Đã thu".
- **Exception**: đã đòi 3 lần chưa thu; ghi thu vượt số còn nợ.
- **Edge/Boundary**: thu từng phần; nhiều hóa đơn; thu thiếu/thừa; ngày thứ 15 đúng ngày nghỉ.
- **Alternative**: bước leo thang sau lần thứ 3 theo tham số.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-01.23 | Một dự án có thể có nhiều hóa đơn theo đợt; mỗi hóa đơn có nhiều lần thu; trục doanh thu chỉ chuyển "Đã thu" khi mọi hóa đơn hiệu lực của dự án được thu đủ |
| BR-01.24 | Nhịp đòi nợ 15 ngày/lần: hệ thống tạo việc đòi nợ cho Kế toán; sau 3 lần chưa thu báo Trưởng phòng Kế toán (R22) |
| BR-01.25 | `[BA đề xuất — TH-B08]` Hiển thị chênh lệch khi thu thiếu/thừa; thiếu thì tiếp tục nhịp đòi nợ; thừa do Kế toán xử lý ngoài hệ thống |
| BR-01.26 | Bước leo thang sau lần đòi nợ thứ 3 (PGĐ Tài chính, ngừng hỗ trợ, pháp lý) là tham số `[CẦN LÀM RÕ VỚI KH]` (TH-B09) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-01.4.1 | Happy | BR-01.24 | US-05 A3 / UC-01.3 | **Given** hóa đơn ở "Đang công nợ" **When** đến nhịp 15 ngày **Then** hệ thống tạo việc đòi nợ cho Kế toán |
| AC-01.4.2 | Exception | BR-01.24 | US-05 A3 / UC-01.3 E4 | **Given** đã đòi nợ 3 lần mà chưa thu **When** đến nhịp kế tiếp **Then** hệ thống báo Trưởng phòng Kế toán và ghi vết |
| AC-01.4.3 | Happy | BR-01.23 | UC-01.3 | **Given** dự án có một hóa đơn **When** Kế toán ghi thu đủ **Then** trục doanh thu chuyển "Đã thu" |
| AC-01.4.4 | Edge | BR-01.23 | UC-01.3 | **Given** dự án có hai hóa đơn theo đợt, đã thu đủ một **When** xem trạng thái **Then** trục doanh thu vẫn "Đang công nợ" cho đến khi thu đủ cả hai |
| AC-01.4.5 | Edge | BR-01.25 | UC-01.3 | **Given** khách trả thiếu hoặc thừa **When** Kế toán ghi thu **Then** hệ thống hiển thị chênh lệch; thiếu thì giữ nhịp đòi nợ `[BA đề xuất]` |
| AC-01.4.6 | Exception | BR-01.26 | UC-01.3 E4 | **Given** sau lần đòi nợ thứ 3 vẫn chưa thu **When** đến bước leo thang kế tiếp **Then** áp bước theo tham số cấu hình; chưa cấu hình thì chỉ báo Trưởng phòng Kế toán `[CẦN LÀM RÕ VỚI KH]` |

---

## 5. Business Rule chung của tính năng
Các BR áp dụng cho toàn tính năng nằm ở FR-01.1: **BR-01.1** đến **BR-01.4** (kênh gắn ở Dự án, hai trục trạng thái, mỗi mốc một chủ, thời hạn là tham số). BR riêng từng sub-feature nằm trong từng sub; mã `BR-01.n` duy nhất trong toàn tính năng.

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Giống Backbone (không khác biệt)`. Tính năng dùng NFR chung `NFR-G2.1` (phạm vi dữ liệu), `NFR-G7.1` (thời hạn là tham số), `NFR-G8.1` (web trên máy tính và điện thoại), `NFR-G10.1` (lưu vết phê duyệt và thay đổi quan trọng).

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| — | — | Theo NFR chung (Backbone mục 3) | Không khác |

## 6b. Phân quyền
> Action: V xem · C tạo · E sửa · A duyệt/xác nhận · X thực hiện chuyển trạng thái. Phạm vi dữ liệu chi tiết do FR-06.

| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-01.1 Tạo dự án kênh B | Nhân viên Kinh doanh | C | Có căn cứ triển khai |
| FR-01.1 Ghi mốc Bắt đầu, Vận hành | Trưởng nhóm triển khai | X | Dự án "Đang triển khai" |
| FR-01.1 Xem chuỗi dự án | Trưởng phòng, PGĐ, các vai trò liên quan | V | Trong phạm vi quyền (NFR-G2.1) |
| FR-01.1 Sửa mốc đã ghi | Trưởng phòng Chuyển giao | E | Lưu vết trước/sau `[CẦN LÀM RÕ VỚI KH]` |
| FR-01.2 Ghi BBNT | Trưởng nhóm / nhân viên Chuyển giao | C, X | Có ngày ký và tệp |
| FR-01.2 Xác nhận nhận BBNT, ký hợp đồng | Nhân viên Kinh doanh | X | Đã nghiệm thu |
| FR-01.3 Đề nghị xuất hóa đơn, gửi hồ sơ, xác nhận khách nhận | Nhân viên Kinh doanh | C, X | Điều kiện BR-01.17 |
| FR-01.3 Ghi số hóa đơn | Kế toán | C, X | Có đề nghị hợp lệ |
| FR-01.3 Duyệt ngoại lệ hóa đơn tạm ứng | PGĐ Tài chính | A | `[BA đề xuất]` |
| FR-01.4 Ghi đòi nợ, ghi thu tiền | Kế toán | C, X | — |
| Nhận cảnh báo leo thang | Trưởng phòng Chuyển giao / Kinh doanh / Kế toán | V | Theo bước quá hạn |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| Giả định G01 (hợp đồng sau BBNT) sai | Đảo thứ tự "Chờ hợp đồng" ở A.4b; sửa FR-01.2 | Kiểm chứng bằng truy vết dự án mẫu (Phần B, B.5 H2) trước khi phát triển |
| Phụ thuộc FR-05 (nhắc, leo thang, "Việc của tôi") | Không có cảnh báo thì mốc chỉ là dữ liệu | Phát hành cùng FR-05 trong MVP |
| Phụ thuộc FR-06 (danh mục, tham số, phân quyền, lưu vết) | Không cấu hình được thời hạn và phạm vi quyền | Phát hành cùng FR-06 |
| Hai chiều với FR-04 (trục triển khai, Đóng dự án) và FR-02 (cùng bản ghi Dự án) | Đổi mô hình Dự án ảnh hưởng cả ba | Chốt `DU_AN`, hai trục trạng thái trước (A.5) |
| Người dùng không cập nhật mốc | Dữ liệu sai làm cảnh báo mất giá trị | Các bước chặn (hóa đơn cần BBNT), "Việc của tôi" là nơi nhận việc, chỉ số M1 theo dõi độ phủ |
| Quá nhiều dự án nhập từ dữ liệu cũ gây cảnh báo giả | Mất tin tưởng cảnh báo | "Ngày bắt đầu đo" ở FR-06.5 |
| Không có thời gian thực nhiều người cùng sửa | Ghi đè dữ liệu | Chưa có cơ chế khóa đồng thời `[CẦN LÀM RÕ VỚI KH]` (TH-E06) |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-01.1: bản ghi Dự án, hai trục trạng thái, 4 mốc, màn hình xem chuỗi; xác nhận G09, G10 | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-01.2: BBNT, hợp đồng, đấu thầu; xác nhận G01 và quy ước ngày | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-01.3 và FR-01.4: hóa đơn, hồ sơ, công nợ; xác nhận G11 và bước leo thang | Chưa bắt đầu |

## 9. Traceability (mini)
> Cột Model, Wireframe, Test để các bước sau điền (⏳).

{{TRACE}}

**NFR liên quan:**
| Mã NFR | Nguồn | NFR Test Scenario |
|---|---|---|
| NFR-G2.1 | Backbone mục 3 (R4) | ⏳ |
| NFR-G7.1 | Backbone mục 3 (R5) | ⏳ |
| NFR-G8.1 | Backbone mục 3 (R1) | ⏳ |
| NFR-G10.1 | Backbone mục 3 (R4) | ⏳ |

## 10. Câu hỏi cần làm rõ với KH (tổng hợp)
| STT | Vị trí trong spec | Câu hỏi cần làm rõ | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | BR-01.15, AC-01.2.9 | Hạn 10 ngày tính theo ngày lịch hay ngày làm việc? | Nice-to-know | Chưa hỏi |
| 2 | BR-01.4 (phụ lục) | Hạn nhắc cho trạng thái Chờ nghiệm thu là bao nhiêu? | Nice-to-know | Chưa hỏi |
| 3 | BR-01.26, AC-01.4.6 | Sau lần đòi nợ thứ 3, bước leo thang tiếp theo là gì? | Nice-to-know | Chưa hỏi |
| 4 | 6b (sửa mốc) | Ai được sửa mốc đã ghi? | Nice-to-know | Chưa hỏi |
| 5 | Mục 7 (đồng thời) | Có cần xử lý nhiều người cùng sửa một dự án? | Nice-to-know | Chưa hỏi |
| 6 | BR-01.12, BR-01.14, BR-01.16, BR-01.21, BR-01.22 | Xác nhận các bổ sung `[BA đề xuất]` (hợp đồng đã ký trước, nghiệm thu giai đoạn, trạng thái hóa đơn, hóa đơn tạm ứng) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1 và Phần A, B | — | Draft |
