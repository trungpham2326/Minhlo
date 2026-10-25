# Feature Spec: Kênh A — dự án đối tác, BBNT đối tác, kỳ đối soát, BBĐS, tham số chia sẻ

**Mã**: FR-02  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-02)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: UC-driven (+US ánh xạ)  
**User Story liên quan**: US-06, US-07, US-08, US-09  
**Khó khăn nguồn**: KK02 · **Rx nguồn**: R3, R5, R9, R10, R16, R17, R26 · **Giả định liên quan**: G02, G03, G04, G22, G23, G24, G25

> **Quy ước:** `Rx` là mã của Backbone mục 5.1. Mã `BR-02.1` đến `BR-02.10` giữ nguyên theo Phần B (B.4); `BR-02.11` trở đi bổ sung khi viết spec. `[BA đề xuất]` là bổ sung chưa nằm trong Backbone v1.1 (kèm mã `TH-..` ở `Tinh-huong-phat-sinh-va-edge-case.md`).

---

## 1. Mục đích
Đưa dự án kênh A (bán qua đối tác) vào hệ thống với cùng cấu trúc như kênh B, và thay việc đối soát thủ công, phụ thuộc một người bằng quy trình kiểm chứng được: danh sách dự án do Chuyển giao ghi nhận làm nguồn đối chiếu độc lập, tỷ lệ chia sẻ là tham số có hiệu lực theo ngày, kỳ đã chốt không bị sửa.

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Dự án kênh A do đầu mối đối tác báo (đối tác, sản phẩm, giấy tờ điều động); BBNT đối tác gửi; số liệu doanh thu đối tác báo cuối kỳ; tỷ lệ chia sẻ, chu kỳ, ngoại lệ; giải thích dòng lệch |
| Xử lý | Tạo dự án kênh A; theo dõi BBNT đối tác; gom dự án vào kỳ theo ngày BBNT; tìm tỷ lệ theo thứ bậc; tính phần chia; đối chiếu; chốt BBĐS; tạo dòng điều chỉnh |
| Output | Kỳ đối soát và BBĐS; phần Công ty A được hưởng theo từng dòng; dòng lệch; thông tin để Kế toán xuất hóa đơn cho đối tác; nhật ký lưu vết |

Sơ đồ: `[MODEL: Sequence]` nhánh kênh A (Hình A.3.1) · `[MODEL: State]` trục doanh thu nhánh kênh A (Hình A.4b) · `[MODEL: ERD]` `TY_LE_CHIA_SE`, `KY_DOI_SOAT`, `DONG_DOI_SOAT`, `BBDS` (Hình A.5)

## 3. Use Case chi tiết

### UC-02.1 — Ghi nhận dự án và BBNT kênh A
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Chuyển giao tạo dự án kênh A theo đầu mối đối tác báo và ghi BBNT khi đối tác gửi |
| **Actor chính** | Nhân viên Chuyển giao |
| **Actor phụ** | Đầu mối đối tác (không đăng nhập), Trưởng phòng Chuyển giao, Hệ thống |
| **Trigger** | Đầu mối đối tác báo dự án mới; đối tác gửi BBNT |
| **Tiền điều kiện** | Đối tác và sản phẩm có trong danh mục; hợp đồng khung của đối tác còn hiệu lực |
| **Hậu điều kiện thành công** | Dự án kênh A ở "Mới tạo"; sau BBNT, trục doanh thu "Chờ đối soát" |
| **Hậu điều kiện thất bại** | Không tạo dự án/BBNT; cảnh báo quá hạn được phát |
| **Business Rule liên quan** | BR-02.12, BR-02.13, BR-02.17, BR-02.18, BR-02.19 |
| **Sub-feature liên quan** | FR-02.1 |

**Luồng chính (Happy):**
1. Đầu mối đối tác báo dự án cho Chuyển giao. → 2. Nhân viên Chuyển giao tạo Dự án: kênh A, đối tác, sản phẩm, đính giấy tờ điều động. → 3. Hệ thống tạo dự án ở "Mới tạo", không yêu cầu hợp đồng hay báo giá. → 4. Sau triển khai, trục doanh thu vào "Chờ nghiệm thu". → 5. Đối tác gửi BBNT; nhân viên Chuyển giao ghi BBNT. → 6. Hệ thống chuyển "Đã nghiệm thu" rồi "Chờ đối soát" và gán dự án vào kỳ theo ngày BBNT.

**Luồng phụ (Alternative):**
- 2a. Bệnh viện đã có dự án kênh B hoặc dự án của đối tác khác: tạo dự án riêng gắn cùng khách hàng.

**Luồng ngoại lệ (Exception):**
- E1. (bước 2) Chọn kênh A nhưng thiếu đối tác → hệ thống chặn.
- E2. (bước 4) Quá hạn gửi BBNT của đối tác → nhắc nhân viên Chuyển giao và Trưởng phòng Chuyển giao.
- E3. (bước 2) Đối tác hết hiệu lực hợp đồng khung → không cho tạo dự án mới `[BA đề xuất — TH-A16]`.

### UC-02.2 — Đối soát kỳ và chốt BBĐS
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | PGĐ Tài chính đối chiếu danh sách dự án của Chuyển giao với số liệu đối tác, xử lý dòng lệch và chốt BBĐS |
| **Actor chính** | PGĐ Tài chính |
| **Actor phụ** | Kế toán (nhập số liệu đối tác), đầu mối đối tác, Hệ thống |
| **Trigger** | Kỳ đối soát kết thúc và đối tác gửi số liệu |
| **Tiền điều kiện** | Đối tác có chu kỳ và tỷ lệ cấu hình; có dự án kênh A "Chờ đối soát" trong kỳ |
| **Hậu điều kiện thành công** | BBĐS đã chốt; dự án trong kỳ "Đã đối soát" |
| **Hậu điều kiện thất bại** | Kỳ chưa chốt; hệ thống nêu dòng lệch hoặc tỷ lệ còn thiếu |
| **Business Rule liên quan** | BR-02.1, BR-02.2, BR-02.9, BR-02.10, BR-02.14, BR-02.15, BR-02.18, BR-02.22 |
| **Sub-feature liên quan** | FR-02.2, FR-02.3 |

**Luồng chính (Happy):**
1. Kỳ kết thúc, hệ thống lập Kỳ đối soát liệt kê dự án của đối tác theo ngày BBNT. → 2. Kế toán nhập số liệu doanh thu đối tác báo. → 3. Hệ thống tìm tỷ lệ theo thứ bậc, tính phần chia từng dòng và đánh dấu khớp hoặc lệch. → 4. Mọi dòng khớp; PGĐ Tài chính chốt BBĐS. → 5. Hệ thống chuyển các dự án trong kỳ sang "Đã đối soát". → 6. Kế toán xuất hóa đơn cho đối tác phần Công ty A được hưởng (trên phần mềm kế toán) và ghi số hóa đơn.

**Luồng phụ (Alternative):**
- 3a. Dòng lệch: PGĐ Tài chính trao đổi với đối tác, ghi giải thích hoặc điều chỉnh có căn cứ rồi chốt.
- 3b. Một kỳ chứa hai tỷ lệ vì đổi giữa kỳ (mỗi dòng ghi tỷ lệ đã áp).

**Luồng ngoại lệ (Exception):**
- E1. (bước 3) Thiếu tỷ lệ cho đối tác × sản phẩm tại ngày BBNT và đối tác không có tỷ lệ mặc định → chặn chốt, nêu thiếu gì.
- E2. (bước 4) Còn dòng lệch chưa giải thích → không cho chốt.
- E3. (bước 2) Đối tác không gửi số liệu quá hạn chốt → nhắc PGĐ Tài chính.
- E4. (sau bước 4) Cần sửa kỳ đã chốt → không sửa trực tiếp, tạo dòng điều chỉnh (UC-02.3).

### UC-02.3 — Đổi tỷ lệ chia sẻ và duyệt ngoại lệ
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Đề xuất, duyệt tỷ lệ mới hoặc ngoại lệ; xử lý hồi tố bằng dòng điều chỉnh |
| **Actor chính** | Kế toán (đề xuất), PGĐ Tài chính (duyệt) |
| **Actor phụ** | Giám đốc (ngoại lệ vượt ngưỡng), IT nội bộ (chỉ quản trị kỹ thuật), Hệ thống |
| **Trigger** | Đối tác đổi tỷ lệ hoặc có thỏa thuận riêng cho một dự án |
| **Tiền điều kiện** | Có căn cứ bằng văn bản |
| **Hậu điều kiện thành công** | Bản ghi tỷ lệ/ngoại lệ mới có hiệu lực theo ngày; bản ghi cũ giữ nguyên |
| **Hậu điều kiện thất bại** | Không tạo bản ghi; thông báo lý do chặn |
| **Business Rule liên quan** | BR-02.3 đến BR-02.8, BR-02.11, BR-02.20 |
| **Sub-feature liên quan** | FR-02.4, FR-02.3 |

**Luồng chính (Happy):**
1. Kế toán đề xuất tỷ lệ mới (đối tác, sản phẩm, % , ngày hiệu lực, căn cứ). → 2. Hệ thống kiểm tra đủ trường, ngày hiệu lực sau bản hiện hành, tỷ lệ 0–100%. → 3. PGĐ Tài chính (không phải người đề xuất) duyệt. → 4. Hệ thống thêm bản ghi mới, đặt bản cũ hết hạn vào ngày liền trước, lưu vết. → 5. Dự án có BBNT từ ngày hiệu lực dùng tỷ lệ mới.

**Luồng phụ (Alternative):**
- 1a. Ngoại lệ cho một dự án hoặc một khoảng ngày: cùng luồng, thêm căn cứ; vượt ngưỡng thì Giám đốc duyệt.
- 1b. Hồi tố vào kỳ đã chốt: không sửa kỳ; tạo dòng điều chỉnh vào kỳ kế tiếp hoặc BBĐS bổ sung (BR-02.8).

**Luồng ngoại lệ (Exception):**
- E1. (bước 3) Người đề xuất tự duyệt → chặn.
- E2. (bước 1) Ngoại lệ không có căn cứ → chặn gửi duyệt.
- E3. (bước 2) Ngày hiệu lực không sau bản hiện hành → chặn, hướng dẫn dùng điều chỉnh.
- E4. (bước 2) Ngoại lệ chồng khoảng hiệu lực cùng phạm vi → chặn `[BA đề xuất — TH-A08]`.

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-02.1 | R10, R16, R17, R26 | Dự án kênh A và BBNT đối tác | Chưa ước lượng | Draft | |
| FR-02.2 | R26 | Kỳ đối soát và đối chiếu | Chưa ước lượng | Draft | |
| FR-02.3 | R9, R26 | Chốt BBĐS, điều chỉnh sau chốt, hóa đơn cho đối tác | Chưa ước lượng | Draft | |
| FR-02.4 | R5, R26 | Tham số chia sẻ: tỷ lệ, chu kỳ, ngoại lệ | Chưa ước lượng | Draft | |

---

### FR-02.1 — Dự án kênh A và BBNT đối tác
- **Mô tả**: Tạo dự án kênh A, theo dõi BBNT do đối tác gửi, nhắc khi quá hạn.
- **Input – Xử lý – Output**: Input: đối tác, sản phẩm, giấy tờ điều động, BBNT. Xử lý: kiểm tra ràng buộc kênh A, tính hạn gửi BBNT, chuyển trạng thái. Output: dự án kênh A, trục doanh thu "Chờ đối soát", cảnh báo.
- **Màn hình UI**: `[WIREFRAME: Tạo dự án kênh A]` · `[WIREFRAME: Ghi BBNT kênh A]`

**Luồng xử lý:**
- **Happy**: tạo dự án kênh A → triển khai → ghi BBNT đối tác gửi → "Chờ đối soát".
- **Exception**: thiếu đối tác; quá hạn gửi BBNT; đối tác hết hiệu lực hợp đồng khung.
- **Edge/Boundary**: bệnh viện có dự án ở cả hai kênh hoặc hai đối tác; đối tác chưa có hạn gửi BBNT cấu hình; đối tác muốn tự nhập.
- **Alternative**: không có.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-02.12 | Dự án kênh A bắt buộc có đối tác và không có hợp đồng hay báo giá; kênh B ngược lại; hai bên loại trừ nhau (D1, R10) |
| BR-02.13 | Hạn đối tác gửi BBNT là tham số theo đối tác `[CẦN LÀM RÕ VỚI KH]`, không đặt số; quá hạn → nhắc nhân viên Chuyển giao và báo Trưởng phòng Chuyển giao; chưa cấu hình thì không tính quá hạn và hiển thị cảnh báo "thiếu tham số" (G02) |
| BR-02.17 | `[BA đề xuất — TH-A16]` Hợp đồng khung hết hiệu lực: dự án đã tạo giữ tỷ lệ theo ngày BBNT và vẫn đối soát đến khi hoàn tất; không tạo dự án mới với đối tác hết hiệu lực |
| BR-02.19 | Trong MVP đối tác không đăng nhập; đầu mối đối tác báo dự án, BBNT và số liệu qua Chuyển giao/Kế toán; người nhập ghi nguồn "đối tác báo" (G24) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-02.1.1 | Happy | BR-02.12 | US-06 A1 / UC-02.1 | **Given** đầu mối đối tác X báo một dự án **When** nhân viên Chuyển giao tạo Dự án kênh A với đối tác X, sản phẩm và giấy tờ điều động **Then** dự án ở "Mới tạo" và hệ thống không yêu cầu hợp đồng hay báo giá |
| AC-02.1.2 | Exception | BR-02.12 | US-06 A2 / UC-02.1 E1 | **Given** chọn kênh A nhưng chưa chọn đối tác **When** lưu **Then** hệ thống chặn |
| AC-02.1.3 | Edge | BR-02.12 | US-06 A3 / UC-02.1 | **Given** bệnh viện M đã có dự án kênh B (LIS) **When** tạo dự án HIS kênh A với đối tác X **Then** hai dự án cùng thuộc khách hàng M; báo cáo theo M thấy cả hai, đối soát của X chỉ thấy dự án HIS |
| AC-02.1.4 | Exception | BR-02.13 | US-07 A1 / UC-02.1 E2 | **Given** dự án kênh A ở "Chờ nghiệm thu" và đối tác có hạn gửi BBNT đã cấu hình **When** quá hạn **Then** hệ thống nhắc nhân viên phụ trách và Trưởng phòng Chuyển giao, hiện trong "Việc của tôi" |
| AC-02.1.5 | Happy | BR-02.13, BR-02.18 | US-07 A2 / UC-02.1 | **Given** đối tác gửi BBNT **When** nhân viên Chuyển giao ghi BBNT **Then** dự án chuyển "Đã nghiệm thu" rồi "Chờ đối soát" và thuộc kỳ theo ngày BBNT |
| AC-02.1.6 | Edge | BR-02.13 | US-07 A3 / UC-02.1 | **Given** đối tác chưa có hạn gửi BBNT được cấu hình **When** dự án ở "Chờ nghiệm thu" **Then** hệ thống không tính quá hạn và hiển thị cảnh báo "thiếu tham số" cho người quản trị tham số |
| AC-02.1.7 | Exception | BR-02.17 | UC-02.1 E3 | **Given** hợp đồng khung của đối tác đã hết hiệu lực **When** nhân viên Chuyển giao tạo dự án mới với đối tác đó **Then** hệ thống chặn; dự án đã tạo trước đó vẫn đối soát bình thường `[BA đề xuất]` |
| AC-02.1.8 | Edge | BR-02.19 | UC-02.1 | **Given** đối tác đề nghị tự nhập BBNT trên hệ thống **When** nhân viên Chuyển giao ghi BBNT thay **Then** bản ghi có nguồn "đối tác báo" và người nhập; việc đối tác đăng nhập thuộc giai đoạn sau |

---

### FR-02.2 — Kỳ đối soát và đối chiếu
- **Mô tả**: Gom dự án kênh A vào kỳ theo ngày BBNT, tính phần chia theo tỷ lệ hiệu lực, đối chiếu với số liệu đối tác.
- **Input – Xử lý – Output**: Input: chu kỳ đối tác, số liệu đối tác báo, giải thích dòng lệch. Xử lý: lập kỳ, tìm tỷ lệ theo thứ bậc, tính phần chia, đánh dấu khớp/lệch. Output: kỳ đối soát với từng dòng và trạng thái.
- **Màn hình UI**: `[WIREFRAME: Kỳ đối soát và đối chiếu]`

**Luồng xử lý:**
- **Happy**: kỳ kết thúc → nhập số liệu đối tác → tính phần chia → mọi dòng khớp.
- **Exception**: thiếu tỷ lệ; đối tác không gửi số liệu quá hạn.
- **Edge/Boundary**: một kỳ chứa hai tỷ lệ; sản phẩm mới chưa có tỷ lệ riêng; đối tác báo thiếu hoặc thừa dự án; BBNT đúng ngày đầu/cuối kỳ; chênh lệch nhỏ do làm tròn.
- **Alternative**: không có.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-02.1 | Phần chia của dòng = giá trị dự án (theo cơ sở tính) × tỷ lệ hiệu lực tại **ngày BBNT** của dự án; tỷ lệ là phần doanh thu thuộc Công ty A, phần còn lại thuộc đối tác; chiều tỷ lệ theo hợp đồng khung `[CẦN LÀM RÕ VỚI KH]` (G03, G04) |
| BR-02.2 | Thứ bậc tìm tỷ lệ: (1) ngoại lệ của dự án còn hiệu lực đã duyệt; (2) tỷ lệ đối tác × sản phẩm tại ngày BBNT; (3) tỷ lệ mặc định của đối tác (hợp đồng khung) tại ngày BBNT; (4) không có thì chặn chốt kỳ |
| BR-02.9 | Thiếu tỷ lệ theo BR-02.2 thì chặn chốt kỳ, nêu chỗ thiếu, không tự đoán |
| BR-02.10 | Danh sách dự án do Chuyển giao ghi nhận là nguồn đối chiếu độc lập; số đối tác báo không thay thế danh sách này; dòng lệch gồm thiếu bên hệ thống, thiếu bên đối tác, lệch giá trị |
| BR-02.18 | Dự án vào kỳ theo **ngày BBNT** và chu kỳ của đối tác (tháng hoặc quý) (G04) |
| BR-02.22 | Hạn chốt BBĐS sau cuối kỳ là tham số `[CẦN LÀM RÕ VỚI KH]`; quá hạn mà đối tác chưa gửi số liệu → nhắc PGĐ Tài chính |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-02.2.1 | Happy | BR-02.1, BR-02.18 | US-08 A1 / UC-02.2 | **Given** đối tác Y có chu kỳ quý và có dự án kênh A "Chờ đối soát" trong quý **When** kỳ kết thúc và số liệu đối tác đã được nhập **Then** hệ thống lập Kỳ đối soát liệt kê dự án, tính phần chia theo tỷ lệ hiệu lực tại ngày BBNT và đánh dấu từng dòng khớp hoặc lệch |
| AC-02.2.2 | Exception | BR-02.9 | US-08 A3 / UC-02.2 E1 | **Given** thiếu tỷ lệ cho đối tác × sản phẩm tại ngày BBNT và đối tác không có tỷ lệ mặc định **When** hệ thống tính **Then** hệ thống không tự đoán, chặn chốt kỳ và chỉ rõ tỷ lệ còn thiếu |
| AC-02.2.3 | Edge | BR-02.2 | UC-02.2 | **Given** sản phẩm mới chưa có tỷ lệ riêng nhưng đối tác có tỷ lệ mặc định **When** hệ thống tính **Then** dòng dùng tỷ lệ mặc định của đối tác và ghi tỷ lệ đã áp |
| AC-02.2.4 | Edge | BR-02.10 | UC-02.2 | **Given** đối tác báo thiếu một dự án mà Chuyển giao đã ghi BBNT (hoặc ngược lại) **When** hệ thống đối chiếu **Then** dòng được đánh dấu lệch theo loại (thiếu bên hệ thống, thiếu bên đối tác, lệch giá trị) |
| AC-02.2.5 | Edge | BR-02.1 | UC-02.2 | **Given** tỷ lệ đổi từ 30% sang 35% hiệu lực 15/08, dự án P1 BBNT 10/08 giá trị 100 và P2 BBNT 20/08 giá trị 200 **When** hệ thống tính **Then** P1 dùng 30% (phần chia 30), P2 dùng 35% (phần chia 70), kỳ có hai tỷ lệ và tổng phần chia 100 (số minh họa, B.4.6) |
| AC-02.2.6 | Edge | BR-02.18 | UC-02.2 | **Given** dự án có ngày BBNT đúng ngày cuối của kỳ **When** hệ thống gán kỳ **Then** dự án thuộc kỳ chứa ngày BBNT đó |
| AC-02.2.7 | Exception | BR-02.22 | UC-02.2 E3 | **Given** quá hạn chốt BBĐS mà đối tác chưa gửi số liệu **When** đến hạn **Then** hệ thống nhắc PGĐ Tài chính và ghi vết |

---

### FR-02.3 — Chốt BBĐS, điều chỉnh sau chốt, hóa đơn cho đối tác
- **Mô tả**: Chốt BBĐS khi mọi dòng được xử lý; không sửa kỳ đã chốt; ghi hóa đơn cho đối tác.
- **Input – Xử lý – Output**: Input: giải thích dòng lệch, dòng điều chỉnh có căn cứ, số hóa đơn. Xử lý: kiểm tra điều kiện chốt, khóa kỳ, tạo điều chỉnh, chuyển trạng thái. Output: BBĐS đã chốt, dự án "Đã đối soát", "Đã xuất hóa đơn cho đối tác".
- **Màn hình UI**: `[WIREFRAME: Chốt BBĐS]` · `[WIREFRAME: Dòng điều chỉnh]` · `[WIREFRAME: Ghi hóa đơn cho đối tác]`

**Luồng xử lý:**
- **Happy**: mọi dòng khớp → PGĐ Tài chính chốt → Kế toán ghi hóa đơn cho đối tác → thu tiền.
- **Exception**: còn dòng lệch; chưa chốt đã đề nghị xuất hóa đơn đối tác; sửa trực tiếp kỳ đã chốt.
- **Edge/Boundary**: hồi tố vào kỳ đã chốt; BBNT đến muộn sau chốt; hoàn doanh thu hoặc đổi giá trị; chênh lệch nhỏ.
- **Alternative**: dòng lệch được giải thích rồi chốt.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-02.8 | Không tính lại kỳ đã chốt BBĐS; hồi tố hoặc thay đổi sau chốt tạo **dòng điều chỉnh** có căn cứ vào kỳ kế tiếp hoặc BBĐS bổ sung, PGĐ Tài chính duyệt (G23) |
| BR-02.14 | Chỉ chốt BBĐS khi mọi dòng khớp hoặc dòng lệch đã được giải thích/điều chỉnh có căn cứ; mặc định không dung sai (G25); quy ước làm tròn `[CẦN LÀM RÕ VỚI KH]` |
| BR-02.15 | Kế toán chỉ ghi hóa đơn cho đối tác sau khi BBĐS đã chốt; hệ thống ghi số hóa đơn, ngày, giá trị và không phát hành hóa đơn pháp định (R9, G03) |
| BR-02.16 | `[BA đề xuất — TH-A13, TH-A19]` Khách trả lại/hủy sau BBNT hoặc giá trị dự án đổi sau BBNT: tạo dòng điều chỉnh (âm hoặc dương) có căn cứ, liên kết dự án gốc; chưa chốt kỳ thì sửa có lưu vết |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-02.3.1 | Happy | BR-02.14 | UC-02.2 | **Given** mọi dòng của kỳ đều khớp và đủ tỷ lệ **When** PGĐ Tài chính chốt BBĐS **Then** kỳ bị khóa, các dự án trong kỳ chuyển "Đã đối soát" |
| AC-02.3.2 | Exception | BR-02.14 | US-08 A2 / UC-02.2 E2 | **Given** kỳ còn dòng lệch **When** PGĐ Tài chính chốt BBĐS **Then** hệ thống chỉ cho chốt khi mọi dòng lệch đã được giải thích hoặc điều chỉnh có căn cứ |
| AC-02.3.3 | Edge | BR-02.8 | US-08 A4 / UC-02.2 E4 | **Given** BBĐS đã chốt **When** cần sửa số liệu **Then** hệ thống không cho sửa trực tiếp mà chỉ cho tạo điều chỉnh có căn cứ |
| AC-02.3.4 | Happy | BR-02.15 | UC-02.2 | **Given** BBĐS đã chốt **When** Kế toán ghi số hóa đơn, ngày, giá trị cho đối tác **Then** dự án chuyển "Đã xuất hóa đơn cho đối tác" và hệ thống không phát hành hóa đơn pháp định |
| AC-02.3.5 | Exception | BR-02.15 | UC-02.2 | **Given** BBĐS chưa chốt **When** Kế toán ghi hóa đơn cho đối tác **Then** hệ thống từ chối và nêu "chưa chốt BBĐS" |
| AC-02.3.6 | Edge | BR-02.8 | UC-02.3 | **Given** đối tác gửi BBNT trễ khi kỳ chứa ngày BBNT đã chốt **When** Kế toán ghi BBNT **Then** hệ thống tạo dòng điều chỉnh vào kỳ kế tiếp, không sửa kỳ đã chốt |
| AC-02.3.7 | Edge | BR-02.16 | UC-02.3 | **Given** khách trả lại sản phẩm sau BBNT **When** Kế toán tạo điều chỉnh có căn cứ **Then** hệ thống tạo dòng điều chỉnh âm trong kỳ kế tiếp, liên kết dự án gốc `[BA đề xuất]` |
| AC-02.3.8 | Edge | BR-02.14 | UC-02.2 | **Given** chênh lệch rất nhỏ do làm tròn **When** PGĐ Tài chính chốt **Then** hệ thống vẫn yêu cầu giải thích vì mặc định không dung sai; dung sai là tham số `[CẦN LÀM RÕ VỚI KH]` |

---

### FR-02.4 — Tham số chia sẻ: tỷ lệ, chu kỳ, ngoại lệ
- **Mô tả**: Đề xuất và duyệt tỷ lệ chia sẻ, chu kỳ đối soát, ngoại lệ có căn cứ; hiệu lực theo ngày; chỉ thêm, không sửa.
- **Input – Xử lý – Output**: Input: đối tác, sản phẩm, %, ngày hiệu lực, căn cứ. Xử lý: kiểm tra, duyệt hai người, thêm bản ghi, đóng bản cũ. Output: tham số hiệu lực theo ngày; nhật ký lưu vết.
- **Màn hình UI**: `[WIREFRAME: Tham số chia sẻ]` · `[WIREFRAME: Duyệt tỷ lệ và ngoại lệ]`

**Luồng xử lý:**
- **Happy**: Kế toán đề xuất → PGĐ Tài chính duyệt → bản ghi mới, bản cũ hết hạn.
- **Exception**: tự duyệt; ngoại lệ thiếu căn cứ; ngày hiệu lực không sau bản hiện hành; tỷ lệ ngoài 0–100%; IT nội bộ sửa giá trị.
- **Edge/Boundary**: ngày hiệu lực trùng ngày BBNT; ngoại lệ vượt ngưỡng; ngoại lệ chồng khoảng hiệu lực; đổi chu kỳ giữa chừng.
- **Alternative**: ngoại lệ theo dự án hoặc khoảng ngày.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-02.3 | Chỉ tạo ngoại lệ khi đủ: (a) căn cứ bằng văn bản đính kèm; (b) phạm vi rõ (một dự án, một sản phẩm hoặc một khoảng ngày); (c) có ngày hiệu lực và hết hạn; (d) duyệt theo ngưỡng (vượt ngưỡng do Giám đốc, ngưỡng `[CẦN LÀM RÕ VỚI KH]`) |
| BR-02.4 | Không cho ngoại lệ khi thiếu căn cứ; khi người đề xuất tự duyệt; hoặc khi muốn sửa tỷ lệ của kỳ đã chốt (dùng BR-02.8) |
| BR-02.5 | Người đề xuất khác người duyệt cho mọi thay đổi tham số và ngoại lệ (G22) |
| BR-02.6 | Mọi thay đổi tạo **bản ghi mới** có ngày hiệu lực sau bản hiện hành; bản cũ giữ nguyên và hết hạn vào ngày liền trước; tỷ lệ phải trong 0–100%; lưu vết (NFR-G10.1) |
| BR-02.7 | Đổi giữa kỳ: dự án có BBNT trước ngày hiệu lực dùng tỷ lệ cũ, từ ngày hiệu lực dùng tỷ lệ mới; một kỳ có thể chứa hai tỷ lệ |
| BR-02.11 | `[BA đề xuất — TH-A08]` Không cho tạo ngoại lệ chồng khoảng hiệu lực cùng phạm vi; thứ bậc BR-02.2 chỉ lấy một ngoại lệ |
| BR-02.20 | Đổi chu kỳ đối soát có hiệu lực từ **kỳ kế tiếp**, không đổi giữa kỳ |
| BR-02.21 | Quyền tham số: Kế toán đề xuất; PGĐ Tài chính duyệt; Giám đốc duyệt ngoại lệ vượt ngưỡng; IT nội bộ chỉ quản trị khung, không sửa giá trị nghiệp vụ; Chuyển giao, Kinh doanh không xem tỷ lệ (G22) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-02.4.1 | Happy | BR-02.6, BR-02.7 | US-09 A1 / UC-02.3 | **Given** Kế toán đề xuất tỷ lệ mới cho đối tác Y × sản phẩm HIS, hiệu lực từ ngày N, kèm căn cứ **When** PGĐ Tài chính duyệt **Then** hệ thống tạo bản ghi mới, bản cũ giữ nguyên và hết hạn vào ngày N−1, tỷ lệ mới áp cho dự án có BBNT từ ngày N |
| AC-02.4.2 | Exception | BR-02.5 | US-09 A2 / UC-02.3 E1 | **Given** người đề xuất cũng là người duyệt **When** bấm duyệt **Then** hệ thống chặn |
| AC-02.4.3 | Exception | BR-02.3, BR-02.4 | US-09 A3 / UC-02.3 E2 | **Given** ngoại lệ cho một dự án không đính căn cứ **When** gửi duyệt **Then** hệ thống chặn |
| AC-02.4.4 | Edge | BR-02.21 | US-09 A4 / UC-02.3 | **Given** người dùng là IT nội bộ **When** mở màn tham số nghiệp vụ **Then** chỉ thấy khung tham số, không có quyền sửa giá trị |
| AC-02.4.5 | Edge | BR-02.6 | UC-02.3 | **Given** tỷ lệ mới có hiệu lực từ ngày N và một dự án có BBNT đúng ngày N **When** hệ thống tìm tỷ lệ **Then** dự án dùng tỷ lệ mới |
| AC-02.4.6 | Exception | BR-02.6, BR-02.8 | UC-02.3 E3 | **Given** ngày hiệu lực không sau bản hiện hành **When** gửi duyệt **Then** hệ thống chặn và hướng dẫn dùng dòng điều chỉnh có căn cứ |
| AC-02.4.7 | Exception | BR-02.6 | UC-02.3 | **Given** tỷ lệ nhập âm hoặc lớn hơn 100% **When** lưu **Then** hệ thống chặn |
| AC-02.4.8 | Edge | BR-02.11 | UC-02.3 E4 | **Given** đã có ngoại lệ cho dự án D trong khoảng 01/09–30/09 **When** đề xuất ngoại lệ khác cho D trong khoảng chồng **Then** hệ thống chặn `[BA đề xuất]` |
| AC-02.4.9 | Edge | BR-02.20 | UC-02.3 | **Given** đối tác đổi chu kỳ từ tháng sang quý **When** PGĐ Tài chính duyệt **Then** chu kỳ mới có hiệu lực từ kỳ kế tiếp và kỳ đang chạy giữ chu kỳ cũ |
| AC-02.4.10 | Edge | BR-02.3 | UC-02.3 | **Given** ngoại lệ có giá trị vượt ngưỡng cấu hình **When** PGĐ Tài chính duyệt **Then** hệ thống chuyển Giám đốc duyệt trước khi có hiệu lực |

---

## 5. Business Rule chung của tính năng
Quy tắc nền áp cho cả tính năng: kênh gắn ở Dự án và một bệnh viện có thể thuộc nhiều đối tác (**BR-02.12**, D1); đối tác không đăng nhập trong MVP (**BR-02.19**); kỳ đã chốt không bị sửa (**BR-02.8**); thay đổi tham số luôn hai người và lưu vết (**BR-02.5**, **BR-02.6**). Mã `BR-02.n` duy nhất trong toàn tính năng.

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Khác biệt — dữ liệu đối soát và BBĐS là dữ liệu tài chính cần bất biến sau khi chốt để kiểm chứng được (KK02)`.

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| NFR-02.1 | Truy vết / Toàn vẹn dữ liệu | Sau khi BBĐS đã chốt, 100% thao tác sửa trực tiếp giá trị dòng đối soát hoặc tỷ lệ đã áp bị từ chối; mọi thay đổi chỉ qua dòng điều chỉnh, mỗi điều chỉnh tạo đúng 1 bản ghi vết (người, thời điểm, giá trị trước/sau, căn cứ). Đo bằng kiểm thử: thử mọi thao tác sửa trên kỳ đã chốt, kết quả mong đợi 100% bị từ chối, và đối chiếu số bản ghi vết với số điều chỉnh. *(nguồn: R26; BA đề xuất làm rõ NFR-G10.1; BR-02.8)* | Chặt hơn NFR-G10.1: vết chung chỉ ghi nhận thay đổi, ở đây thay đổi bị cấm sau chốt |

## 6b. Phân quyền
| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-02.1 Tạo dự án kênh A, ghi BBNT | Nhân viên Chuyển giao | C, X | Đối tác còn hiệu lực hợp đồng khung |
| FR-02.1 Nhắc BBNT quá hạn | Trưởng phòng Chuyển giao | V | Khi quá hạn |
| FR-02.2 Nhập số liệu đối tác | Kế toán | C, E | Trước khi chốt kỳ |
| FR-02.2 Xem kỳ đối soát | PGĐ Tài chính, Kế toán | V | — |
| FR-02.3 Chốt BBĐS | PGĐ Tài chính | A, X | Mọi dòng khớp hoặc đã giải thích |
| FR-02.3 Tạo dòng điều chỉnh | Kế toán | C | Có căn cứ; PGĐ Tài chính duyệt |
| FR-02.3 Ghi hóa đơn cho đối tác | Kế toán | C, X | Sau khi chốt BBĐS |
| FR-02.4 Đề xuất tỷ lệ, chu kỳ, ngoại lệ | Kế toán | C | Có căn cứ |
| FR-02.4 Duyệt tỷ lệ, chu kỳ, ngoại lệ dưới ngưỡng | PGĐ Tài chính | A | Khác người đề xuất |
| FR-02.4 Duyệt ngoại lệ vượt ngưỡng | Giám đốc | A | Ngưỡng `[CẦN LÀM RÕ VỚI KH]` |
| FR-02.4 Xem tỷ lệ | PGĐ Kinh doanh | V | `[CẦN LÀM RÕ VỚI KH]` |
| FR-02.4 Sửa giá trị tham số | IT nội bộ | — | Không có quyền (chỉ quản trị khung) |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| G03 (cơ sở tính, chiều dòng tiền) sai | Đổi cách tính `DONG_DOI_SOAT` và nghiệp vụ hóa đơn kênh A | Back-test bằng BBĐS kỳ trước (Phần B, B.5 H3) |
| G04 (khóa tỷ lệ, ngày áp) sai | Đổi khóa `TY_LE_CHIA_SE` và cách chia kỳ | Xác nhận với PGĐ Tài chính trước khi phát triển |
| Phụ thuộc FR-01 (bản ghi Dự án, BBNT) | Không có dự án thì không có dòng đối soát | Dùng chung `DU_AN` (A.5) |
| Phụ thuộc FR-06 (tham số, quyền, lưu vết) | Không cấu hình được tỷ lệ và hai người duyệt | Phát hành cùng FR-06 |
| Phụ thuộc FR-05 (nhắc, leo thang) | Không nhắc BBNT quá hạn và hạn chốt | Phát hành cùng FR-05 |
| Định dạng số liệu đối tác chưa biết | Nhập tay tốn công, dễ sai | MVP nhập tay; nhập từ tệp để Phát hành 2 |
| Không đối tác đăng nhập (G24) | Chuyển giao và Kế toán nhập thay | Ghi nguồn "đối tác báo" để truy vết |
| Nhiều người cùng sửa kỳ đối soát | Ghi đè dữ liệu | Chưa có khóa đồng thời `[CẦN LÀM RÕ VỚI KH]` (TH-E06) |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-02.4: mô hình tham số; back-test với BBĐS 1–2 kỳ trước; xác nhận G03, G04, G22 | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-02.1 và FR-02.2: dự án kênh A, BBNT, kỳ đối soát, dòng lệch | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-02.3: chốt BBĐS, điều chỉnh sau chốt, hóa đơn đối tác | Chưa bắt đầu |

## 9. Traceability (mini)
{{TRACE}}

**NFR liên quan:**
| Mã NFR | Nguồn | NFR Test Scenario |
|---|---|---|
| NFR-02.1 | Mục 6 tính năng này (R26, BR-02.8) | ⏳ |
| NFR-G2.1 | Backbone mục 3 (R4) | ⏳ |
| NFR-G7.1 | Backbone mục 3 (R5) | ⏳ |
| NFR-G8.1 | Backbone mục 3 (R1) | ⏳ |
| NFR-G10.1 | Backbone mục 3 (R4) | ⏳ |

## 10. Câu hỏi cần làm rõ với KH (tổng hợp)
| STT | Vị trí trong spec | Câu hỏi cần làm rõ | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | BR-02.1, BR-02.15 | Cơ sở tính là doanh thu thực thu hay phát sinh? Chiều tỷ lệ (phần Công ty A hay phần đối tác) và dòng tiền chiều nào? (G03) | Blocking (đang dùng giả định tạm) | Chưa hỏi |
| 2 | BR-02.1, BR-02.18 | Tỷ lệ áp theo đối tác, sản phẩm hay loại hợp đồng? Hiệu lực tính theo ngày nào? (G04) | Blocking (đang dùng giả định tạm) | Chưa hỏi |
| 3 | BR-02.13 | Hợp đồng khung có quy định hạn gửi BBNT của đối tác không? (G02) | Nice-to-know | Chưa hỏi |
| 4 | BR-02.3, BR-02.21 | Ngưỡng ngoại lệ cần Giám đốc; PGĐ Kinh doanh có được xem tỷ lệ không? (G22) | Nice-to-know | Chưa hỏi |
| 5 | BR-02.14 | Có chấp nhận dung sai chênh lệch không; quy ước làm tròn? (G25) | Nice-to-know | Chưa hỏi |
| 6 | BR-02.22 | Hạn chốt BBĐS sau cuối kỳ? | Nice-to-know | Chưa hỏi |
| 7 | BR-02.8 | Điều chỉnh sau khi chốt BBĐS, hóa đơn điều chỉnh xử lý thế nào ngoài hệ thống? (G23) | Nice-to-know | Chưa hỏi |
| 8 | BR-02.11, BR-02.16, BR-02.17 | Xác nhận các bổ sung `[BA đề xuất]` (ngoại lệ không chồng, điều chỉnh hoàn/đổi giá trị, đối tác hết hiệu lực hợp đồng khung) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1, Phần A, B (B.4) | — | Draft |
