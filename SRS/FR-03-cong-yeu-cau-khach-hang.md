# Feature Spec: Cổng yêu cầu khách hàng (nhánh rủi ro, nhánh tính phí)

**Mã**: FR-03  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-03)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: UC-driven (+US ánh xạ)  
**User Story liên quan**: US-10, US-11, US-12, US-13  
**Khó khăn nguồn**: KK04 · **Rx nguồn**: R2, R3, R11, R12, R13, R14, R15, R28 · **Giả định liên quan**: G05, G06, G07, G08, G19

> **Quy ước:** `Rx` là mã của Backbone mục 5.1. `[BA đề xuất]` là bổ sung chưa nằm trong Backbone v1.1 (kèm mã `TH-..` ở `Tinh-huong-phat-sinh-va-edge-case.md`). Tên trạng thái đúng với Phần A mục A.4c.

---

## 1. Mục đích
Gom mọi yêu cầu của bệnh viện (cổng, điện thoại, tin nhắn, email, nhân viên triển khai) vào một bản ghi có nguồn gốc, đi qua luồng duyệt chuẩn: IT bệnh viện duyệt → Công ty A đánh giá → cổng rủi ro (lãnh đạo bệnh viện xác nhận) và cổng tính phí (PGĐ Chuyên môn, báo giá, miễn/giảm do Giám đốc) → xử lý. Mục tiêu: không còn yêu cầu thất lạc, không còn tranh cãi "ai đã đồng ý", không làm miễn phí việc lẽ ra thu tiền.

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Nội dung yêu cầu, tệp đính kèm, bệnh viện, nguồn, dự án/sản phẩm liên quan; duyệt của IT bệnh viện; phân loại rủi ro/tính phí; xác nhận của lãnh đạo bệnh viện; duyệt tính phí, miễn/giảm; báo giá và chấp nhận; kết quả xử lý |
| Xử lý | Tạo yêu cầu; chuyển trạng thái theo cổng; chạy hai cổng song song; chặn giao Phần mềm khi còn cổng; giao việc; nhắc quá hạn; lưu vết mọi phê duyệt |
| Output | Yêu cầu với trạng thái, người phụ trách và bằng chứng phê duyệt; việc cho CSKH, Phần mềm, Kinh doanh; thông báo cho khách; nhật ký lưu vết |

Sơ đồ: `[MODEL: Sequence]` Hình A.3.2 · `[MODEL: State]` Yêu cầu (Hình A.4c) · `[MODEL: ERD]` `YEU_CAU`, `PHE_DUYET`, `DAU_VIEC` (Hình A.5)

## 3. Use Case chi tiết

### UC-03.1 — Gửi và duyệt yêu cầu
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Người dùng bệnh viện (hoặc CSKH nhập hộ) gửi yêu cầu; IT bệnh viện duyệt hoặc từ chối trước khi sang Công ty A |
| **Actor chính** | Người dùng bệnh viện, IT bệnh viện |
| **Actor phụ** | CSKH (nhập hộ), Hệ thống |
| **Trigger** | Bệnh viện có nhu cầu |
| **Tiền điều kiện** | Người dùng có tài khoản cổng (hoặc yêu cầu đến từ kênh ngoài cổng) |
| **Hậu điều kiện thành công** | Yêu cầu ở "Chờ đánh giá", CSKH thấy |
| **Hậu điều kiện thất bại** | Yêu cầu "Đóng - bị từ chối" hoặc chưa được tạo |
| **Business Rule liên quan** | BR-03.1, BR-03.2, BR-03.4 đến BR-03.7 |
| **Sub-feature liên quan** | FR-03.1, FR-03.2 |

**Luồng chính (Happy):**
1. Người dùng bệnh viện nhập nội dung, đính kèm và gửi. → 2. Hệ thống tạo yêu cầu với nguồn "cổng", trạng thái "Chờ IT bệnh viện duyệt". → 3. IT bệnh viện duyệt. → 4. Hệ thống chuyển "Chờ đánh giá" và giao CSKH.

**Luồng phụ (Alternative):**
- 1a. Yêu cầu đến qua điện thoại, tin nhắn, email hoặc nhân viên triển khai: CSKH nhập hộ, ghi nguồn và người nhập; yêu cầu vẫn vào "Chờ IT bệnh viện duyệt" (G05).

**Luồng ngoại lệ (Exception):**
- E1. (bước 1) Thiếu trường bắt buộc → báo lỗi, không tạo yêu cầu.
- E2. (bước 3) IT bệnh viện từ chối → bắt buộc lý do; yêu cầu "Đóng - bị từ chối", thông báo người gửi.
- E3. (bước 3) IT bệnh viện không phản hồi quá hạn tham số → nhắc IT bệnh viện, CSKH thấy tình trạng.

### UC-03.2 — Đánh giá và xác nhận rủi ro
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | CSKH phân loại, Phần mềm xác nhận mức rủi ro; yêu cầu rủi ro phải được lãnh đạo bệnh viện xác nhận trước khi xử lý |
| **Actor chính** | CSKH, Phần mềm |
| **Actor phụ** | Lãnh đạo bệnh viện, Hệ thống |
| **Trigger** | Yêu cầu ở "Chờ đánh giá" |
| **Tiền điều kiện** | IT bệnh viện đã duyệt |
| **Hậu điều kiện thành công** | Yêu cầu qua cổng rủi ro (xác nhận) hoặc được xác định là phổ thông |
| **Hậu điều kiện thất bại** | Yêu cầu "Đóng - bị từ chối" |
| **Business Rule liên quan** | BR-03.3, BR-03.8 đến BR-03.12 |
| **Sub-feature liên quan** | FR-03.3 |

**Luồng chính (Happy):**
1. CSKH phân loại sơ bộ theo danh mục rủi ro và cờ "có thể tính phí". → 2. Phần mềm xác nhận hoặc nâng mức. → 3. Nếu có rủi ro, hệ thống chuyển "Chờ lãnh đạo bệnh viện xác nhận" và không giao Phần mềm. → 4. Lãnh đạo bệnh viện xác nhận. → 5. Hệ thống ghi người, thời điểm, nội dung; nếu mọi cổng khác đã xong thì sang "Đã duyệt".

**Luồng phụ (Alternative):**
- 1a. Yêu cầu phổ thông: không có cổng, sang "Đã duyệt" ngay.
- 3a. Yêu cầu vừa rủi ro vừa tính phí: hai cổng chạy song song (UC-03.3).

**Luồng ngoại lệ (Exception):**
- E1. (bước 4) Lãnh đạo bệnh viện từ chối → yêu cầu "Đóng - bị từ chối".
- E2. (bước 4) Lãnh đạo nghỉ hoặc đổi người → dùng danh mục người có thẩm quyền theo bệnh viện `[BA đề xuất — TH-C06]`.
- E3. (sau bước 5) Phát hiện phân loại sai khi đang xử lý → thu hồi về "Chờ đánh giá" `[BA đề xuất — TH-C08]`.
- E4. (bước 3) Có người yêu cầu "cứ làm, sẽ gửi giấy sau" → hệ thống không giao Phần mềm khi chưa có xác nhận ghi trên hệ thống.

### UC-03.3 — Tính phí và báo giá
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | PGĐ Chuyên môn duyệt tính phí; Kinh doanh báo giá; bệnh viện chấp nhận; miễn/giảm do Giám đốc duyệt |
| **Actor chính** | PGĐ Chuyên môn, Nhân viên Kinh doanh |
| **Actor phụ** | Giám đốc, Bệnh viện, Hệ thống |
| **Trigger** | Yêu cầu được đánh dấu "có thể tính phí" |
| **Tiền điều kiện** | IT bệnh viện đã duyệt; Phần mềm đã xác nhận |
| **Hậu điều kiện thành công** | Cổng tính phí hoàn tất (báo giá được chấp nhận hoặc miễn/giảm được duyệt) |
| **Hậu điều kiện thất bại** | Yêu cầu "Đóng - bị từ chối" |
| **Business Rule liên quan** | BR-03.3, BR-03.13 đến BR-03.17 |
| **Sub-feature liên quan** | FR-03.4 |

**Luồng chính (Happy):**
1. PGĐ Chuyên môn duyệt tính phí. → 2. Hệ thống tạo việc lập báo giá cho Kinh doanh. → 3. Kinh doanh gửi báo giá cho bệnh viện. → 4. Bệnh viện chấp nhận, hệ thống ghi bằng chứng. → 5. Cổng tính phí hoàn tất; nếu mọi cổng khác xong thì sang "Đã duyệt".

**Luồng phụ (Alternative):**
- 1a. Đề xuất miễn, giảm hoặc ưu đãi: PGĐ Chuyên môn chuyển Giám đốc; Giám đốc duyệt thì miễn (sang cổng xong) hoặc giảm (tiếp tục báo giá).

**Luồng ngoại lệ (Exception):**
- E1. (bước 1–4) Có người cố giao Phần mềm làm khi cổng tính phí chưa xong → hệ thống chặn.
- E2. (bước 4) Báo giá hết hạn hiệu lực → yêu cầu "Đóng"; bệnh viện hủy sau khi chấp nhận → Kinh doanh ghi lý do.
- E3. (bước 3) Bệnh viện kênh A → thông báo đầu mối đối tác, không báo giá trực tiếp (G08).

### UC-03.4 — Xử lý và đóng yêu cầu
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Phần mềm xử lý yêu cầu đã duyệt; CSKH cập nhật khách và đóng |
| **Actor chính** | Phần mềm, CSKH |
| **Actor phụ** | Người dùng bệnh viện, Hệ thống |
| **Trigger** | Yêu cầu sang "Đã duyệt" |
| **Tiền điều kiện** | Mọi cổng bắt buộc đã qua |
| **Hậu điều kiện thành công** | Yêu cầu "Đóng" sau khi khách xác nhận |
| **Hậu điều kiện thất bại** | Yêu cầu bị thu hồi hoặc quá hạn, có cảnh báo |
| **Business Rule liên quan** | BR-03.18 đến BR-03.21 |
| **Sub-feature liên quan** | FR-03.5 |

**Luồng chính (Happy):**
1. Phần mềm nhận xử lý ("Đang xử lý"). → 2. Phần mềm xử lý xong ("Đã xử lý"). → 3. CSKH cập nhật cho khách. → 4. Khách xác nhận; yêu cầu "Đóng".

**Luồng phụ (Alternative):**
- 1a. Khách đổi nội dung sau khi duyệt: tạo yêu cầu mới liên kết yêu cầu cũ `[BA đề xuất — TH-C09]`.

**Luồng ngoại lệ (Exception):**
- E1. (bước 1) Quá SLA nhận hoặc xử lý → nhắc người phụ trách, báo cáo nội bộ, không phạt tự động.
- E2. (bước 3) Khách chưa xác nhận quá hạn → quy tắc tự đóng theo tham số `[CẦN LÀM RÕ VỚI KH]`.

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-03.1 | R2, R3, R12, R28 | Gửi yêu cầu qua cổng và CSKH nhập hộ | Chưa ước lượng | Draft | |
| FR-03.2 | R13 | IT bệnh viện duyệt | Chưa ước lượng | Draft | |
| FR-03.3 | R14, R28 | Đánh giá và cổng rủi ro | Chưa ước lượng | Draft | |
| FR-03.4 | R11, R28 | Cổng tính phí, báo giá, miễn/giảm | Chưa ước lượng | Draft | |
| FR-03.5 | R15 | Xử lý, cập nhật khách và đóng | Chưa ước lượng | Draft | |

---

### FR-03.1 — Gửi yêu cầu qua cổng và CSKH nhập hộ
- **Mô tả**: Tạo yêu cầu từ cổng hoặc do CSKH nhập hộ; ghi nguồn; gắn dự án/sản phẩm.
- **Input – Xử lý – Output**: Input: nội dung, tệp, bệnh viện, nguồn, dự án/sản phẩm. Xử lý: kiểm tra trường bắt buộc, tạo yêu cầu, giao IT bệnh viện duyệt. Output: yêu cầu "Chờ IT bệnh viện duyệt", trạng thái cho người gửi.
- **Màn hình UI**: `[WIREFRAME: Cổng gửi yêu cầu]` · `[WIREFRAME: CSKH nhập hộ yêu cầu]`

**Luồng xử lý:**
- **Happy**: người dùng gửi qua cổng → "Chờ IT bệnh viện duyệt".
- **Exception**: thiếu trường bắt buộc; người dùng ngoài bệnh viện.
- **Edge/Boundary**: nguồn ngoài cổng; chọn sai sản phẩm; bệnh viện dùng cả HIS (kênh A) và LIS (kênh B); gửi trùng; tệp đính kèm lớn.
- **Alternative**: CSKH nhập hộ.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-03.1 | Mọi yêu cầu là một bản ghi duy nhất có nguồn (cổng, điện thoại, tin nhắn, email, nhân viên triển khai); không xử lý ngoài luồng (R3, G05) |
| BR-03.4 | Yêu cầu bắt buộc: nội dung, bệnh viện, nguồn; gắn dự án/sản phẩm liên quan; CSKH gán lại nếu người gửi chọn sai |
| BR-03.5 | Nguồn ngoài cổng do CSKH nhập hộ, ghi nguồn và người nhập; yêu cầu vẫn phải được IT bệnh viện xác nhận trước khi chuyển xử lý (G05) |
| BR-03.22 | Cổng phục vụ cả bệnh viện kênh A và B; kênh chỉ quyết định hợp đồng và doanh thu, không quyết định ai xử lý yêu cầu kỹ thuật (G08) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-03.1.1 | Happy | BR-03.1, BR-03.4 | US-10 A1 / UC-03.1 | **Given** người dùng bệnh viện đã đăng nhập cổng **When** nhập nội dung, đính kèm và gửi **Then** yêu cầu ở "Chờ IT bệnh viện duyệt" với nguồn "cổng" và người gửi thấy được trạng thái |
| AC-03.1.2 | Exception | BR-03.4 | US-10 A2 / UC-03.1 E1 | **Given** thiếu nội dung bắt buộc **When** gửi **Then** hệ thống báo lỗi và không tạo yêu cầu |
| AC-03.1.3 | Edge | BR-03.2 | US-10 A3 / UC-03.1 | **Given** yêu cầu chưa được IT bệnh viện duyệt **When** CSKH của Công ty A tìm trong hàng đợi xử lý **Then** không thấy yêu cầu đó |
| AC-03.1.4 | Happy | BR-03.5 | UC-03.1 | **Given** bệnh viện báo yêu cầu qua điện thoại **When** CSKH nhập hộ với nguồn "điện thoại" **Then** yêu cầu có nguồn và người nhập, vào "Chờ IT bệnh viện duyệt" |
| AC-03.1.5 | Edge | BR-03.4, BR-03.22 | UC-03.1 | **Given** bệnh viện dùng HIS (kênh A) và LIS (kênh B) và người gửi chọn sai sản phẩm **When** CSKH rà soát **Then** CSKH gán lại dự án/sản phẩm đúng và ghi vết |
| AC-03.1.6 | Edge | BR-03.4 | UC-03.1 | **Given** nhiều người cùng bệnh viện gửi yêu cầu giống nhau **When** CSKH nhận **Then** CSKH có thể liên kết hoặc gộp yêu cầu trùng (cảnh báo trùng tự động ở giai đoạn sau) `[CẦN LÀM RÕ VỚI KH]` |

---

### FR-03.2 — IT bệnh viện duyệt
- **Mô tả**: IT bệnh viện duyệt hoặc từ chối trước khi yêu cầu sang Công ty A.
- **Input – Xử lý – Output**: Input: duyệt/từ chối, lý do. Xử lý: chuyển trạng thái, nhắc quá hạn. Output: "Chờ đánh giá" hoặc "Đóng - bị từ chối".
- **Màn hình UI**: `[WIREFRAME: IT bệnh viện duyệt yêu cầu]`

**Luồng xử lý:**
- **Happy**: IT bệnh viện duyệt → "Chờ đánh giá".
- **Exception**: từ chối không lý do; IT bệnh viện không phản hồi.
- **Edge/Boundary**: yêu cầu nhập hộ chưa có xác nhận; nhiều IT bệnh viện; đúng hạn nhắc.
- **Alternative**: không có.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-03.2 | Chỉ yêu cầu đã được IT bệnh viện duyệt mới sang Công ty A (R13) |
| BR-03.6 | Từ chối bắt buộc có lý do; yêu cầu sang "Đóng - bị từ chối" và thông báo người gửi |
| BR-03.7 | IT bệnh viện quá hạn phản hồi (hạn là tham số `[CẦN LÀM RÕ VỚI KH]`) → nhắc IT bệnh viện; CSKH thấy tình trạng; quy tắc tự đóng chưa chốt |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-03.2.1 | Happy | BR-03.2 | US-11 A1 / UC-03.1 | **Given** yêu cầu "Chờ IT bệnh viện duyệt" **When** IT bệnh viện duyệt **Then** yêu cầu sang "Chờ đánh giá" và CSKH thấy |
| AC-03.2.2 | Exception | BR-03.6 | US-11 A2 / UC-03.1 E2 | **Given** IT bệnh viện từ chối **When** không nhập lý do **Then** hệ thống chặn; có lý do thì đóng yêu cầu và thông báo người gửi |
| AC-03.2.3 | Edge | BR-03.5 | US-11 A3 / UC-03.1 | **Given** yêu cầu qua điện thoại do CSKH nhập hộ **When** chưa có xác nhận của IT bệnh viện **Then** yêu cầu không được chuyển sang xử lý |
| AC-03.2.4 | Exception | BR-03.7 | UC-03.1 E3 | **Given** IT bệnh viện không phản hồi quá hạn tham số **When** đến hạn **Then** hệ thống nhắc IT bệnh viện và hiển thị tình trạng cho CSKH; không tự đóng khi chưa có quy tắc `[CẦN LÀM RÕ VỚI KH]` |

---

### FR-03.3 — Đánh giá và cổng rủi ro
- **Mô tả**: CSKH phân loại, Phần mềm xác nhận; yêu cầu rủi ro chờ lãnh đạo bệnh viện xác nhận trước khi xử lý.
- **Input – Xử lý – Output**: Input: phân loại rủi ro, cờ tính phí, xác nhận/từ chối của lãnh đạo. Xử lý: chuyển cổng, chặn giao Phần mềm, ghi bằng chứng. Output: cổng rủi ro xong hoặc yêu cầu đóng.
- **Màn hình UI**: `[WIREFRAME: Phân loại yêu cầu]` · `[WIREFRAME: Lãnh đạo bệnh viện xác nhận]`

**Luồng xử lý:**
- **Happy**: phân loại có rủi ro → lãnh đạo xác nhận → cổng xong.
- **Exception**: lãnh đạo từ chối; xác nhận qua điện thoại không ghi trên hệ thống.
- **Edge/Boundary**: vừa rủi ro vừa tính phí; phân loại sai; lãnh đạo đổi người; yêu cầu "gấp".
- **Alternative**: yêu cầu phổ thông không có cổng.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-03.3 | Yêu cầu chỉ sang "Đã duyệt" khi mọi cổng bắt buộc đã qua; cổng rủi ro và cổng tính phí độc lập, chạy song song (G19) |
| BR-03.8 | CSKH phân loại sơ bộ theo danh mục rủi ro cấu hình được (mặc định: sửa/xóa dữ liệu, đổi quy trình); Phần mềm xác nhận hoặc nâng mức (G06) |
| BR-03.9 | Yêu cầu rủi ro cần lãnh đạo bệnh viện xác nhận; hệ thống ghi người, thời điểm, nội dung; chưa xác nhận thì không giao Phần mềm; mức "gấp" chỉ rút ngắn bằng cảnh báo ưu tiên, không bỏ cổng (R14) |
| BR-03.10 | Lãnh đạo bệnh viện từ chối thì yêu cầu "Đóng - bị từ chối" và thông báo người gửi |
| BR-03.11 | `[BA đề xuất — TH-C06]` Danh mục người có thẩm quyền xác nhận theo từng bệnh viện, cập nhật khi đổi người |
| BR-03.12 | `[BA đề xuất — TH-C08]` Phần mềm hoặc CSKH có thể thu hồi yêu cầu đang xử lý về "Chờ đánh giá" khi phát hiện phân loại sai; bắt buộc lý do, dừng xử lý, ghi vết |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-03.3.1 | Happy | BR-03.8, BR-03.9 | US-12 A1 / UC-03.2 | **Given** yêu cầu "xóa dữ liệu" đã qua IT bệnh viện **When** CSKH phân loại rủi ro và Phần mềm xác nhận **Then** yêu cầu sang "Chờ lãnh đạo bệnh viện xác nhận" và không được giao Phần mềm |
| AC-03.3.2 | Happy | BR-03.9, BR-03.3 | US-12 A2 / UC-03.2 | **Given** lãnh đạo bệnh viện xác nhận **When** ghi nhận **Then** hệ thống lưu người, thời điểm, nội dung xác nhận; nếu mọi cổng khác đã xong thì yêu cầu sang "Đã duyệt" |
| AC-03.3.3 | Exception | BR-03.10 | US-12 A3 / UC-03.2 E1 | **Given** lãnh đạo bệnh viện từ chối **When** ghi nhận **Then** yêu cầu "Đóng - bị từ chối" và thông báo người gửi |
| AC-03.3.4 | Edge | BR-03.3 | US-12 A4 / UC-03.2 | **Given** yêu cầu vừa rủi ro vừa có thể tính phí **When** lãnh đạo xác nhận nhưng PGĐ Chuyên môn chưa duyệt tính phí **Then** yêu cầu chưa sang "Đã duyệt" |
| AC-03.3.5 | Edge | BR-03.11 | UC-03.2 E2 | **Given** lãnh đạo bệnh viện đã nghỉ hoặc đổi người **When** hệ thống chọn người xác nhận **Then** chỉ người trong danh mục có thẩm quyền của bệnh viện được xác nhận `[BA đề xuất]` |
| AC-03.3.6 | Edge | BR-03.12 | UC-03.2 E3 | **Given** yêu cầu phân loại phổ thông đang "Đang xử lý" nhưng thực chất rủi ro **When** Phần mềm thu hồi kèm lý do **Then** yêu cầu về "Chờ đánh giá", dừng xử lý và ghi vết `[BA đề xuất]` |
| AC-03.3.7 | Exception | BR-03.9 | UC-03.2 E4 | **Given** bệnh viện nói qua điện thoại "cứ làm, sẽ gửi giấy sau" cho yêu cầu rủi ro **When** CSKH ghi nhận cuộc gọi **Then** hệ thống không giao Phần mềm cho đến khi có xác nhận của người có thẩm quyền ghi trên hệ thống |

---

### FR-03.4 — Cổng tính phí, báo giá, miễn/giảm
- **Mô tả**: Duyệt tính phí, báo giá, chấp nhận; miễn/giảm do Giám đốc duyệt.
- **Input – Xử lý – Output**: Input: duyệt tính phí, báo giá, chấp nhận (bằng chứng), đề xuất miễn/giảm. Xử lý: giao việc báo giá, chặn giao Phần mềm khi chưa xong, ghi bằng chứng. Output: cổng tính phí xong hoặc yêu cầu đóng.
- **Màn hình UI**: `[WIREFRAME: PGĐ Chuyên môn duyệt tính phí]` · `[WIREFRAME: Báo giá yêu cầu]` · `[WIREFRAME: Giám đốc duyệt miễn giảm]`

**Luồng xử lý:**
- **Happy**: duyệt tính phí → báo giá → bệnh viện chấp nhận → cổng xong.
- **Exception**: đề xuất miễn/giảm chưa được duyệt; giao Phần mềm khi cổng chưa xong.
- **Edge/Boundary**: báo giá hết hạn; hủy sau chấp nhận; bệnh viện kênh A; lỗi trong bảo hành.
- **Alternative**: Giám đốc duyệt giảm rồi tiếp tục báo giá.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-03.13 | Quyết định "có tính phí" thuộc PGĐ Chuyên môn; Kinh doanh lập báo giá; cổng tính phí hoàn tất khi bệnh viện chấp nhận và có bằng chứng (G07) |
| BR-03.14 | Miễn, giảm, ưu đãi do Giám đốc duyệt; chưa duyệt thì không giao Phần mềm (R11); ngưỡng giá trị `[CẦN LÀM RÕ VỚI KH]` |
| BR-03.15 | Báo giá có hạn hiệu lực (tham số `[CẦN LÀM RÕ VỚI KH]`); hết hạn thì "Đóng"; hủy sau khi chấp nhận cần Kinh doanh ghi lý do |
| BR-03.16 | Yêu cầu tính phí của bệnh viện kênh A thông báo cho đầu mối đối tác, không báo giá trực tiếp (G08) |
| BR-03.17 | `[BA đề xuất — TH-C10]` Yêu cầu có trường loại (lỗi, thay đổi, hỗ trợ); lỗi của Công ty A trong thời gian bảo hành không qua cổng tính phí |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-03.4.1 | Happy | BR-03.13 | US-13 A1 / UC-03.3 | **Given** yêu cầu được đánh dấu "có thể tính phí" **When** PGĐ Chuyên môn duyệt tính phí **Then** hệ thống tạo việc lập báo giá cho Kinh doanh; sau khi báo giá được gửi và bệnh viện chấp nhận (có bằng chứng), cổng tính phí hoàn tất |
| AC-03.4.2 | Exception | BR-03.14 | US-13 A2 / UC-03.3 | **Given** có đề xuất miễn hoặc giảm **When** PGĐ Chuyên môn chuyển Giám đốc **Then** việc chỉ được thực hiện sau khi Giám đốc duyệt; chưa duyệt thì không giao Phần mềm |
| AC-03.4.3 | Exception | BR-03.3, BR-03.13 | US-13 A3 / UC-03.3 E1 | **Given** yêu cầu chưa qua cổng tính phí **When** có người cố giao Phần mềm làm **Then** hệ thống chặn và nêu cổng còn lại |
| AC-03.4.4 | Edge | BR-03.15 | UC-03.3 E2 | **Given** báo giá quá hạn hiệu lực **When** đến hạn **Then** yêu cầu "Đóng"; hủy sau khi chấp nhận thì Kinh doanh phải ghi lý do |
| AC-03.4.5 | Edge | BR-03.16 | UC-03.3 E3 | **Given** bệnh viện thuộc kênh A gửi yêu cầu có tính phí **When** PGĐ Chuyên môn duyệt tính phí **Then** hệ thống tạo thông báo cho đầu mối đối tác thay vì giao Kinh doanh báo giá trực tiếp |
| AC-03.4.6 | Edge | BR-03.17 | UC-03.3 | **Given** yêu cầu loại "lỗi" của dự án đang trong bảo hành **When** phân loại **Then** yêu cầu không vào cổng tính phí `[BA đề xuất]` |

---

### FR-03.5 — Xử lý, cập nhật khách và đóng
- **Mô tả**: Phần mềm xử lý yêu cầu đã duyệt; CSKH cập nhật khách; khách xác nhận; đóng.
- **Input – Xử lý – Output**: Input: nhận xử lý, xử lý xong, cập nhật khách, xác nhận của khách. Xử lý: chuyển trạng thái, nhắc SLA. Output: "Đang xử lý" → "Đã xử lý" → "Đóng".
- **Màn hình UI**: `[WIREFRAME: Hàng đợi xử lý của Phần mềm]` · `[WIREFRAME: CSKH cập nhật và đóng yêu cầu]`

**Luồng xử lý:**
- **Happy**: Phần mềm nhận → xử lý xong → CSKH cập nhật → khách xác nhận → "Đóng".
- **Exception**: giao Phần mềm khi còn cổng; quá SLA.
- **Edge/Boundary**: khách đổi nội dung; khách chưa xác nhận quá hạn; yêu cầu bị thu hồi.
- **Alternative**: yêu cầu phổ thông sang "Đã duyệt" ngay.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-03.18 | Yêu cầu phổ thông sang "Đã duyệt" ngay sau phân loại; Phần mềm nhận và xử lý; ERP chỉ quản lý quy trình và bằng chứng phê duyệt, không thực thi trên hệ thống của bệnh viện (R15) |
| BR-03.19 | "Đã xử lý" → CSKH cập nhật khách → khách xác nhận → "Đóng"; chờ khách quá hạn → quy tắc tự đóng theo tham số `[CẦN LÀM RÕ VỚI KH]` |
| BR-03.20 | SLA xử lý chỉ dùng cảnh báo và báo cáo nội bộ; không phạt tự động (R8) |
| BR-03.21 | `[BA đề xuất — TH-C09]` Khách đổi nội dung sau khi duyệt: tạo yêu cầu mới liên kết yêu cầu cũ; yêu cầu cũ đóng; các cổng chạy lại theo phân loại mới |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-03.5.1 | Happy | BR-03.18, BR-03.19 | UC-03.4 | **Given** yêu cầu phổ thông đã qua IT bệnh viện và được phân loại **When** Phần mềm nhận và xử lý xong, CSKH cập nhật và khách xác nhận **Then** yêu cầu đi qua "Đã duyệt", "Đang xử lý", "Đã xử lý" và "Đóng" |
| AC-03.5.2 | Exception | BR-03.3 | UC-03.4 | **Given** yêu cầu còn một cổng bắt buộc chưa xong **When** Phần mềm bấm nhận xử lý **Then** hệ thống chặn và nêu cổng còn lại |
| AC-03.5.3 | Exception | BR-03.20 | UC-03.4 E1 | **Given** yêu cầu quá SLA xử lý **When** đến hạn **Then** hệ thống nhắc người phụ trách và ghi vào báo cáo nội bộ, không tự áp chế tài |
| AC-03.5.4 | Edge | BR-03.19 | UC-03.4 E2 | **Given** khách chưa xác nhận quá hạn tham số **When** đến hạn **Then** hệ thống nhắc CSKH; tự đóng chỉ khi đã có quy tắc cấu hình `[CẦN LÀM RÕ VỚI KH]` |
| AC-03.5.5 | Edge | BR-03.21 | UC-03.4 | **Given** khách đổi nội dung sau khi yêu cầu đã duyệt **When** CSKH tạo yêu cầu mới liên kết **Then** yêu cầu cũ đóng, yêu cầu mới chạy lại các cổng theo phân loại mới `[BA đề xuất]` |

---

## 5. Business Rule chung của tính năng
Quy tắc nền: một bản ghi yêu cầu có nguồn, không xử lý ngoài luồng (**BR-03.1**); chỉ yêu cầu đã IT bệnh viện duyệt mới sang Công ty A (**BR-03.2**); hai cổng độc lập, mọi cổng bắt buộc phải qua mới "Đã duyệt" (**BR-03.3**); cổng phục vụ cả hai kênh (**BR-03.22**). Mã `BR-03.n` duy nhất trong toàn tính năng.

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Khác biệt — cổng có người dùng bên ngoài (bệnh viện) và dữ liệu có thể nhạy cảm, cần cô lập dữ liệu giữa các bệnh viện`.

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| NFR-03.1 | Bảo mật | Người dùng bệnh viện chỉ thấy yêu cầu, báo giá và tệp đính kèm của chính bệnh viện mình; 0 dữ liệu của bệnh viện khác xuất hiện ở mọi màn hình, tìm kiếm, thông báo và đường dẫn trực tiếp tới tệp. Đo bằng kiểm thử phân quyền: dùng tài khoản của bệnh viện A truy cập (kể cả sửa đường dẫn) dữ liệu của bệnh viện B, kết quả mong đợi 0 truy cập thành công. *(nguồn: R4; BA đề xuất vì có người dùng bên ngoài và dữ liệu y tế)* | Chặt hơn NFR-G2.1: NFR chung là phạm vi theo vai trò nội bộ, ở đây là cô lập giữa các tổ chức bên ngoài |

## 6b. Phân quyền
| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-03.1 Gửi yêu cầu qua cổng | Người dùng bệnh viện | C, V | Chỉ yêu cầu của bệnh viện mình (NFR-03.1) |
| FR-03.1 Nhập hộ yêu cầu ngoài cổng | CSKH | C | Ghi nguồn và người nhập |
| FR-03.2 Duyệt/từ chối | IT bệnh viện | A | Chỉ yêu cầu của bệnh viện mình |
| FR-03.3 Phân loại sơ bộ | CSKH | E | Yêu cầu "Chờ đánh giá" |
| FR-03.3 Xác nhận/nâng mức rủi ro, thu hồi | Phần mềm | A, X | Có lý do khi thu hồi |
| FR-03.3 Xác nhận rủi ro | Lãnh đạo bệnh viện | A | Thuộc danh mục người có thẩm quyền `[BA đề xuất]` |
| FR-03.4 Duyệt tính phí | PGĐ Chuyên môn | A | Cờ "có thể tính phí" |
| FR-03.4 Lập và gửi báo giá | Nhân viên Kinh doanh | C, X | Sau khi duyệt tính phí |
| FR-03.4 Chấp nhận báo giá | IT bệnh viện / người có thẩm quyền | A | Có bằng chứng |
| FR-03.4 Duyệt miễn/giảm/ưu đãi | Giám đốc | A | Ngưỡng `[CẦN LÀM RÕ VỚI KH]` |
| FR-03.5 Nhận xử lý, xử lý xong | Phần mềm | X | Mọi cổng đã qua |
| FR-03.5 Cập nhật khách, đóng | CSKH | X | Đã xử lý |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| Bệnh viện không dùng cổng hoặc IT bệnh viện chậm duyệt | Yêu cầu tồn ở bước đầu | Chạy thử thủ công với 1–2 bệnh viện (Phần B, B.5 H5); CSKH nhập hộ |
| Danh mục rủi ro chưa đầy đủ (G06) | Bỏ sót yêu cầu rủi ro | Danh mục cấu hình được; rà soát với CSKH, Phần mềm |
| Chưa có quản lý tài khoản người dùng bệnh viện | Không biết ai được dùng cổng | Giao cho IT bệnh viện quản lý người dùng của họ (FR-06) `[BA đề xuất — TH-C14]` |
| Phụ thuộc FR-05 (nhắc, SLA) và FR-06 (quyền, tham số) | Không nhắc, không cô lập dữ liệu | Phát hành cùng nhau trong MVP |
| Hai chiều với FR-01 (báo giá, hợp đồng, dự án liên quan) | Thay đổi dự án ảnh hưởng gắn yêu cầu | Dùng chung `DU_AN` |
| Dữ liệu bệnh nhân có nằm trong yêu cầu/tệp không | Có thể phải tuân thủ thêm quy định | Hỏi KH (NFR nhóm 9, G16) |
| Mức "gấp" trong sự cố bệnh viện | Áp lực bỏ qua cổng | Giữ nguyên cổng, rút ngắn bằng cảnh báo ưu tiên; luồng ngoại lệ khẩn cần quyết định của Ban giám đốc (TH-C03) |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-03.1, FR-03.2: gửi yêu cầu, IT bệnh viện duyệt; chạy thử thủ công | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-03.3: danh mục rủi ro, xác nhận của lãnh đạo; xác nhận G05, G06 | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-03.4, FR-03.5: tính phí, miễn/giảm, xử lý, đóng; xác nhận G07, G08 | Chưa bắt đầu |

## 9. Traceability (mini)
{{TRACE}}

**NFR liên quan:**
| Mã NFR | Nguồn | NFR Test Scenario |
|---|---|---|
| NFR-03.1 | Mục 6 tính năng này (R4) | ⏳ |
| NFR-G2.1 | Backbone mục 3 (R4) | ⏳ |
| NFR-G7.1 | Backbone mục 3 (R5) | ⏳ |
| NFR-G8.1 | Backbone mục 3 (R1) | ⏳ |
| NFR-G10.1 | Backbone mục 3 (R4) | ⏳ |

## 10. Câu hỏi cần làm rõ với KH (tổng hợp)
| STT | Vị trí trong spec | Câu hỏi cần làm rõ | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | BR-03.5, AC-03.2.3 | IT bệnh viện xác nhận yêu cầu ngoài cổng qua kênh nào (email, đường liên kết)? (G05) | Blocking (đang dùng giả định tạm) | Chưa hỏi |
| 2 | BR-03.8 | Danh mục rủi ro đầy đủ gồm những gì, ai đánh giá? (G06) | Blocking (đang dùng giả định tạm) | Chưa hỏi |
| 3 | BR-03.14 | Ngưỡng giá trị nào cần Giám đốc duyệt miễn/giảm? (G07) | Nice-to-know | Chưa hỏi |
| 4 | BR-03.16 | Phí yêu cầu thêm của bệnh viện kênh A thu và chia thế nào? (G08) | Nice-to-know | Chưa hỏi |
| 5 | BR-03.7, BR-03.15, BR-03.19 | Hạn phản hồi của IT bệnh viện, hạn hiệu lực báo giá, quy tắc tự đóng khi khách không xác nhận | Nice-to-know | Chưa hỏi |
| 6 | Mục 7 | Có cần luồng ngoại lệ khẩn cấp khi bệnh viện ngừng vận hành, và ai duyệt? (TH-C03) | Nice-to-know | Chưa hỏi |
| 7 | AC-03.1.6 | Có cần phát hiện yêu cầu trùng tự động? (TH-C13) | Nice-to-know | Chưa hỏi |
| 8 | BR-03.11, BR-03.12, BR-03.17, BR-03.21 | Xác nhận các bổ sung `[BA đề xuất]` (người có thẩm quyền theo bệnh viện, thu hồi, loại yêu cầu lỗi/thay đổi/hỗ trợ, yêu cầu đổi nội dung) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1, Phần A, B | — | Draft |
