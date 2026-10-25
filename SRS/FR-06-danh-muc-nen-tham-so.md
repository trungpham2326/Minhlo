# Feature Spec: Danh mục nền (Khách hàng, Đối tác, Sản phẩm, Nhân sự), phân quyền và cấu hình tham số

**Mã**: FR-06  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-06)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: Requirement thuần  
**User Story liên quan**: US-09 (một phần: cấu hình tham số, phần nghiệp vụ chia sẻ ở FR-02.4)  
**Khó khăn nguồn**: nền cho KK01…KK05 (không có khó khăn riêng) · **Rx nguồn**: R2, R4, R5, R17 · **Giả định liên quan**: G13, G16, G22, G24, G26 · **Quyết định nền**: D1, D4

> **Quy ước:** `Rx` là mã của Backbone mục 5.1. `[BA đề xuất]` là bổ sung chưa nằm trong Backbone v1.1, chờ xác nhận. FR-06 là **khung dùng chung**: nó cung cấp danh mục, vai trò, phạm vi dữ liệu, cơ chế tham số và lưu vết; quy tắc nghiệp vụ của từng tham số (ví dụ tỷ lệ chia sẻ) thuộc FR gốc (FR-02.4). Ma trận quyền tổng hợp (vai trò × chức năng × phạm vi) lập ở bước mô hình (skill 1.5); spec này chốt nguyên tắc, danh sách vai trò và loại phạm vi.

---

## 1. Mục đích
Cung cấp nền để FR-01…05 chạy được và đáng tin: dữ liệu gốc dùng chung (một khách hàng — nhiều dự án — hai kênh), mỗi người chỉ thấy đúng phần dữ liệu của vai trò mình, mọi con số nghiệp vụ là tham số có mặc định và ngoại lệ có căn cứ (R5), mọi phê duyệt và thay đổi quan trọng đều lưu vết (R4), và hệ thống bắt đầu đo từ một ngày rõ ràng nên không sinh cảnh báo giả từ dữ liệu cũ.

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Khách hàng, đối tác và hợp đồng khung, sản phẩm, phòng ban và nhân sự; tài khoản, vai trò, phạm vi dữ liệu; tham số (giá trị mặc định, ngoại lệ, căn cứ, hiệu lực); danh mục cấu hình (rủi ro, loại dự án, loại yêu cầu, lịch ngày làm việc); ngày bắt đầu đo; dữ liệu dự án đang mở để nhập ban đầu |
| Xử lý | Kiểm tra trùng và ràng buộc danh mục; cấp, đổi, thu hồi quyền; kiểm tra tách nhiệm vụ (đề xuất ≠ duyệt); tạo bản ghi tham số mới có hiệu lực theo ngày; ghi nhật ký bất biến; nhập ban đầu có kiểm tra |
| Output | Danh mục dùng chung; quyền theo vai trò và phạm vi cho mọi FR; tham số có hiệu lực theo ngày; nhật ký lưu vết tra cứu được; dự án đang mở được nạp với trạng thái đúng hoặc "Cần xác minh" |

Sơ đồ: `[MODEL: ERD]` `KHACH_HANG`, `DOI_TAC`, `HOP_DONG_KHUNG`, `SAN_PHAM`, `PHONG_BAN`, `NHAN_SU`, `THAM_SO` (Hình A.5) · `[MODEL: Permission Matrix]` thuộc skill 1.5

## 3. Yêu cầu chi tiết (Requirement thuần)

### 3.1 Khái niệm nền
| Khái niệm | Định nghĩa |
|---|---|
| Vai trò | Nhóm quyền theo đầu việc (Backbone mục 2): nội bộ (Giám đốc, PGĐ Kinh doanh/Chuyên môn/Tài chính, Trưởng phòng, Nhân viên Kinh doanh, CSKH, Trưởng phòng/Trưởng nhóm/Nhân viên Chuyển giao, Phần mềm, Test, Hạ tầng, Kế toán/Trưởng phòng Kế toán, Hành chính – Nhân sự, IT nội bộ) và bên ngoài (Người dùng bệnh viện, IT bệnh viện, Lãnh đạo bệnh viện) |
| Phạm vi dữ liệu | Giới hạn bản ghi một vai trò được thấy: toàn công ty · theo phòng · theo dự án được phân công · theo khách hàng (bệnh viện) cho người dùng bên ngoài · theo trường (ẩn một số trường, ví dụ tỷ lệ chia sẻ) |
| Quyền | Vai trò × chức năng × action (V xem, C tạo, E sửa, A duyệt/xác nhận, X chuyển trạng thái) × phạm vi dữ liệu |
| Tham số | Giá trị cấu hình được: tên, phạm vi áp dụng, giá trị mặc định, giá trị ngoại lệ + căn cứ, hiệu lực từ/đến, người đề xuất, người duyệt (`THAM_SO`) |
| Nhật ký (vết) | Bản ghi chỉ thêm, không sửa/xóa: người, vai trò, thời điểm, đối tượng, hành động, giá trị trước/sau, căn cứ |
| Ngày bắt đầu đo | Ngày hệ thống bắt đầu tính nhắc, leo thang và số liệu SLA; mốc có ngày gốc trước đó nằm ngoài phạm vi đo |

### 3.2 Phụ lục BR-06.19 — Danh mục tham số MVP
> Mọi tham số đổi qua giao diện, không cần phát hành bản mới (NFR-G7.1). "Chờ làm rõ" nghĩa là chưa có giá trị mặc định và quy tắc thiếu tham số áp dụng (BR-06.23).

| Mã | Tham số | Phạm vi áp dụng | Mặc định | FR dùng | Nhóm |
|---|---|---|---|---|---|
| P01 | Hạn nộp BBNT cho Kinh doanh | Toàn hệ thống | 5 ngày làm việc kể từ ngày ký (R19) | FR-01, QN-02 | Thường |
| P02 | Hạn ký hợp đồng | Toàn hệ thống | 10 ngày kể từ ngày nhận BBNT, loại ngày chờ làm rõ (R20) | FR-01, QN-03 | Thường |
| P03 | Gọi xác nhận sau gửi hồ sơ | Toàn hệ thống | 1–2 ngày (R21) | FR-01, QN-04 | Thường |
| P04 | Chờ hợp đồng gốc | Toàn hệ thống | 2 tuần (R21) | FR-01, QN-05 | Thường |
| P05 | Nhịp đòi nợ và số lần trước khi báo | Toàn hệ thống | 15 ngày/lần; báo sau 3 lần (R22) | FR-01, QN-06 | Thường |
| P06 | Hạn nhắc Chờ nghiệm thu | Toàn hệ thống / sản phẩm | Chờ làm rõ (G10) | FR-01, QN-01 | Thường |
| P07 | Hạn đối tác gửi BBNT | Theo đối tác | Chờ làm rõ, không đặt số (G02) | FR-02, QN-07 | Thường |
| P08 | Chu kỳ đối soát | Theo đối tác | Tháng hoặc quý, nhập theo hợp đồng khung; không có thì chặn lập kỳ | FR-02 | Tài chính |
| P09 | Tỷ lệ chia sẻ | Theo đối tác × sản phẩm | Theo hợp đồng khung, không đặt số mặc định chung (G04) | FR-02 | Tài chính |
| P10 | Ngoại lệ tỷ lệ | Theo dự án / sản phẩm / khoảng ngày | Không có; cần căn cứ (BR-02.3) | FR-02 | Tài chính |
| P11 | Dung sai đối soát và quy ước làm tròn | Toàn hệ thống | Không dung sai (G25); làm tròn chờ làm rõ | FR-02 | Tài chính |
| P12 | Hạn chốt BBĐS sau cuối kỳ | Theo đối tác | Chờ làm rõ | FR-02, QN-08 | Tài chính |
| P13 | Ngưỡng ngoại lệ cần Giám đốc duyệt | Toàn hệ thống | Chờ làm rõ (G22) | FR-02, FR-03 | Tài chính |
| P14 | Hạn phản hồi của IT bệnh viện | Toàn hệ thống / bệnh viện | Chờ làm rõ | FR-03, QN-09 | Thường |
| P15 | Hạn hiệu lực báo giá | Toàn hệ thống | Chờ làm rõ | FR-03 | Thường |
| P16 | Quy tắc tự đóng khi khách không xác nhận | Toàn hệ thống | Chờ làm rõ; chưa cấu hình thì không tự đóng | FR-03, QN-10 | Thường |
| P17 | SLA nhận và xử lý yêu cầu | Theo loại/mức yêu cầu | Chờ làm rõ (G14) | FR-03, QN-11 | Thường |
| P18 | Hạn rà soát bàn giao của CSKH | Toàn hệ thống | Chờ làm rõ | FR-04, QN-12 | Thường |
| P19 | Số tháng bảo hành / số tháng mốc | Theo sản phẩm (và kênh) | Chờ làm rõ (G20) | FR-04, FR-07 | Thường |
| P20 | Ngưỡng "sắp đến hạn" | Toàn hệ thống | 1 ngày làm việc `[BA đề xuất]` | FR-05 | Thường |
| P21 | Lịch ngày làm việc và ngày lễ | Toàn hệ thống | Chờ làm rõ | FR-05 | Thường |
| P22 | Ngày bắt đầu đo | Toàn hệ thống | Ngày chạy chính thức; đặt trước khi nhập ban đầu | FR-05, FR-06.5 | Thường |
| P23 | Định mức thưởng, công thức nghỉ bù | Theo loại dự án / phòng | Khung có sẵn, chưa cấu hình (FR-08, FR-09) | Phát hành sau | Thường |

*Nhóm Tài chính*: Kế toán đề xuất, PGĐ Tài chính duyệt, vượt ngưỡng thì Giám đốc duyệt (G22, BR-02.21). *Nhóm Thường*: xem BR-06.21.

### 3.3 Danh mục cấu hình khác (không phải số)
Danh mục rủi ro (mặc định: sửa/xóa dữ liệu, đổi quy trình — BR-03.8), loại dự án (thường/đấu thầu), loại yêu cầu (lỗi/thay đổi/hỗ trợ — BR-03.17), danh mục người có thẩm quyền xác nhận rủi ro theo bệnh viện (BR-03.11), quy tắc nhắc QN-xx (FR-05.2). Cùng cơ chế duyệt và lưu vết như tham số nhóm Thường.

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-06.1 | R2, R17 | Danh mục nền: Khách hàng, Đối tác và hợp đồng khung, Sản phẩm, Phòng ban và Nhân sự | Chưa ước lượng | Draft | |
| FR-06.2 | R4 | Tài khoản, vai trò, phạm vi dữ liệu và phân quyền | Chưa ước lượng | Draft | |
| FR-06.3 | R5 | Khung cấu hình tham số | Chưa ước lượng | Draft | |
| FR-06.4 | R4 | Lưu vết (nhật ký bất biến) và tra cứu | Chưa ước lượng | Draft | |
| FR-06.5 | R5 | Ngày bắt đầu đo và nhập dữ liệu ban đầu | Chưa ước lượng | Draft | |

Hồ sơ nhân sự chi tiết, lương/thưởng, tài sản thuộc FR-09 và FR-14 (không trong MVP). Thiết bị kết nối (`THIET_BI`) thuộc FR-10.

---

### FR-06.1 — Danh mục nền
- **Mô tả**: Quản lý Khách hàng, Đối tác (cùng hợp đồng khung), Sản phẩm, Phòng ban và Nhân sự dùng chung cho mọi FR.
- **Input – Xử lý – Output**: Input: thông tin từng danh mục. Xử lý: kiểm tra trùng, ràng buộc tham chiếu, ngừng dùng thay xóa, kiểm tra điều kiện khi nhân sự nghỉ. Output: danh mục sẵn sàng cho dự án, yêu cầu, điều động, phân quyền.
- **Màn hình UI**: `[WIREFRAME: Khách hàng]` · `[WIREFRAME: Đối tác và hợp đồng khung]` · `[WIREFRAME: Sản phẩm]` · `[WIREFRAME: Phòng ban và nhân sự]`

**Luồng xử lý:**
- **Happy**: tạo khách hàng/đối tác/sản phẩm/nhân sự → dùng trong dự án và yêu cầu.
- **Exception**: trùng khách hàng; xóa bản ghi đã được tham chiếu; nghỉ việc khi còn việc.
- **Edge/Boundary**: một bệnh viện thuộc cả hai kênh hoặc hai đối tác; hợp đồng khung hết hiệu lực; gộp khách hàng trùng; nhân sự đổi phòng; sản phẩm ngừng dùng.
- **Alternative**: nhập hàng loạt khi khởi tạo (FR-06.5).

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-06.1 | Khách hàng là trung tâm (R2): mỗi khách hàng một bản ghi duy nhất (mã, tên, loại bệnh viện/trung tâm y tế/phòng khám, địa chỉ, đầu mối); **kênh không phải thuộc tính của khách hàng** mà ở Dự án (D1), nên một bệnh viện có nhiều dự án thuộc kênh A và kênh B hoặc hai đối tác khác nhau; xem theo khách hàng thấy mọi dự án (R17) |
| BR-06.2 | Chống trùng khách hàng theo khóa nhận diện (mã số cơ sở/mã số thuế hoặc tên chuẩn hóa `[CẦN LÀM RÕ VỚI KH]`): cảnh báo khi tạo trùng; gộp khách hàng trùng do người quản trị danh mục thực hiện, giữ nguyên dự án và yêu cầu, lưu vết |
| BR-06.3 | Đối tác có mã, tên, đầu mối liên hệ, trạng thái; mỗi đối tác ≥ 1 hợp đồng khung (hiệu lực từ–đến, chu kỳ đối soát, hạn gửi BBNT); tỷ lệ chia sẻ nằm ở FR-02.4, không nằm trong danh mục; đầu mối đối tác là người liên hệ, không có tài khoản trong MVP (G24); hợp đồng khung hết hiệu lực thì không tạo dự án mới (BR-02.17) |
| BR-06.4 | Sản phẩm (HIS, LIS, EMR, PACS, app, dịch vụ hạ tầng, thuê máy chủ, bảo trì…) có mã, tên, nhóm, số tháng bảo hành mặc định (P19), trạng thái; sản phẩm đã có dự án chỉ ngừng dùng, không xóa |
| BR-06.5 | Phòng ban gồm Ban giám đốc và 9 phòng; IT nội bộ là vị trí kiêm nhiệm gắn nhân sự, không phải phòng; nhân sự có mã, họ tên, phòng, chức danh, quản lý trực tiếp/Trưởng phòng, trạng thái (đang làm/nghỉ), email; MVP không lưu lương, thưởng hay hồ sơ nhạy cảm (G16, FR-09, FR-14) |
| BR-06.6 | Quan hệ leo thang lấy từ cơ cấu: người leo thang mặc định của nhân sự là Trưởng phòng của phòng đó; Trưởng phòng leo thang lên PGĐ phụ trách mảng `[CẦN LÀM RÕ VỚI KH]`; cấu hình được (FR-05.2) |
| BR-06.7 | Nhân sự nghỉ hoặc chuyển phòng: hệ thống liệt kê đầu việc đang mở, dự án đang phụ trách và vai trò đang giữ, buộc Trưởng phòng chuyển cho người khác trước khi khóa tài khoản; không xóa nhân sự đã có lịch sử |
| BR-06.8 | Bản ghi danh mục đã được tham chiếu không xóa cứng; ngừng dùng kèm lý do; mọi tạo, sửa, gộp, ngừng dùng lưu vết (NFR-G10.1) |
| BR-06.9 | `[BA đề xuất]` Chủ sở hữu từng danh mục: Khách hàng do Kinh doanh, CSKH, Chuyển giao (khi đối tác báo bệnh viện mới) tạo, Trưởng phòng Kinh doanh gộp; Đối tác và hợp đồng khung do Kế toán nhập, PGĐ Tài chính duyệt; Sản phẩm do Trưởng phòng Kinh doanh quản lý; Nhân sự do Hành chính – Nhân sự quản lý; IT nội bộ chỉ thao tác kỹ thuật |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-06.1.1 | Happy | BR-06.1 | R2, R17 | **Given** bệnh viện M có dự án HIS (kênh A, đối tác X) và dự án LIS (kênh B) **When** mở khách hàng M **Then** thấy cả hai dự án với kênh từng dự án; khách hàng M không có trường kênh |
| AC-06.1.2 | Exception | BR-06.2 | R2 | **Given** đã có khách hàng cùng khóa nhận diện **When** người dùng tạo khách hàng mới trùng **Then** hệ thống cảnh báo trùng và chỉ cho tạo khi người có quyền xác nhận khác biệt |
| AC-06.1.3 | Exception | BR-06.8 | R2 | **Given** sản phẩm LIS đã gắn 5 dự án **When** người dùng xóa **Then** hệ thống chặn và chỉ cho ngừng dùng kèm lý do |
| AC-06.1.4 | Happy | BR-06.3 | R10 | **Given** đối tác X có hợp đồng khung hiệu lực 01/01–31/12, chu kỳ quý, hạn gửi BBNT đã cấu hình **When** Kế toán lưu và PGĐ Tài chính duyệt **Then** đối tác dùng được để tạo dự án kênh A (FR-02.1) |
| AC-06.1.5 | Edge | BR-06.3 | BR-02.17 | **Given** hợp đồng khung của đối tác Y đã hết hiệu lực **When** Chuyển giao tạo dự án mới cho Y **Then** hệ thống chặn; dự án cũ của Y vẫn đối soát bình thường |
| AC-06.1.6 | Edge | BR-06.2 | R2 | **Given** hai khách hàng trùng đã có dự án và yêu cầu riêng **When** Trưởng phòng Kinh doanh gộp **Then** mọi dự án và yêu cầu về một khách hàng, bản ghi bị gộp ngừng dùng và sự kiện gộp được ghi vết |
| AC-06.1.7 | Edge | BR-06.7 | R4 | **Given** nhân viên Kinh doanh nghỉ việc còn 3 đầu việc mở và 2 dự án phụ trách **When** Hành chính – Nhân sự chuyển trạng thái "nghỉ" **Then** hệ thống yêu cầu Trưởng phòng chuyển 5 mục sang người khác trước khi khóa tài khoản |
| AC-06.1.8 | Exception | BR-06.5 | G16 | **Given** người dùng thêm nhân sự **When** nhập lương hoặc hồ sơ nhạy cảm **Then** MVP không có trường này và không lưu dữ liệu đó |

---

### FR-06.2 — Tài khoản, vai trò, phạm vi dữ liệu và phân quyền
- **Mô tả**: Cấp tài khoản, gán vai trò và phạm vi dữ liệu, bảo đảm tách nhiệm vụ và cô lập dữ liệu giữa các bệnh viện.
- **Input – Xử lý – Output**: Input: tài khoản, vai trò, phạm vi, yêu cầu cấp/đổi quyền. Xử lý: kiểm tra quyền mỗi thao tác ở phía máy chủ, kiểm tra tách nhiệm vụ, ghi vết. Output: người dùng thấy và làm đúng phần của vai trò.
- **Màn hình UI**: `[WIREFRAME: Người dùng và vai trò]` · `[WIREFRAME: Quản lý người dùng bệnh viện]`

**Luồng xử lý:**
- **Happy**: cấp tài khoản → gán vai trò và phạm vi → người dùng thao tác trong phạm vi.
- **Exception**: truy cập ngoài phạm vi (kể cả sửa đường dẫn); tự duyệt; IT nội bộ sửa giá trị nghiệp vụ.
- **Edge/Boundary**: một người nhiều vai trò; Trưởng nhóm thay giữa chừng; nhân viên được điều động vào dự án; PGĐ Kinh doanh (G13); người dùng bệnh viện đổi người.
- **Alternative**: IT bệnh viện tự quản lý người dùng của viện.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-06.10 | Quyền = vai trò × chức năng × action × phạm vi dữ liệu (R4); một người có thể có nhiều vai trò, quyền là hợp của các vai trò |
| BR-06.11 | Mặc định từ chối: dữ liệu và chức năng không được cấp thì không hiển thị và bị từ chối cả khi gọi trực tiếp (đường dẫn, tệp, truy vấn), không chỉ ẩn nút (NFR-G2.1) |
| BR-06.12 | Phạm vi theo dự án gắn với điều động: nhân viên triển khai chỉ thấy dự án mình được điều động; Trưởng phòng thấy mọi dự án của phòng; đổi Trưởng nhóm giữa chừng chuyển quyền theo sự kiện đổi và người cũ giữ quyền xem lịch sử phần mình làm (BR-01.8); quyền riêng của PGĐ Kinh doanh chờ chốt (G13) |
| BR-06.13 | Tách nhiệm vụ: mọi thao tác loại "duyệt" yêu cầu người thực hiện khác người đề xuất (tham số, ngoại lệ, điều chỉnh đối soát — BR-02.5); Trưởng nhóm lập danh sách tồn không được xác nhận nhận (BR-04.12); kiểm tra ở máy chủ |
| BR-06.14 | IT nội bộ quản trị kỹ thuật (tài khoản, khung tham số, cấu hình hệ thống) nhưng không sửa giá trị tham số nghiệp vụ và không phê duyệt nghiệp vụ (G22) |
| BR-06.15 | Trường nhạy cảm kiểm soát theo trường: tỷ lệ chia sẻ, ngoại lệ, giá trị đối soát chỉ vai trò có quyền xem (Kế toán, PGĐ Tài chính, Giám đốc; PGĐ Kinh doanh chờ chốt); Chuyển giao và Kinh doanh không xem (BR-02.21) |
| BR-06.16 | Người dùng bệnh viện (Người dùng, IT bệnh viện, Lãnh đạo bệnh viện) thuộc đúng một khách hàng; mọi truy vấn của họ bị ràng buộc theo mã khách hàng; 0 dữ liệu của bệnh viện khác ở màn hình, tìm kiếm, thông báo, đường dẫn tệp (NFR-03.1) |
| BR-06.17 | IT bệnh viện tự quản lý người dùng của viện mình (tạo, khóa, gán Người dùng/Lãnh đạo); tài khoản IT bệnh viện đầu tiên do CSKH hoặc IT nội bộ cấp khi bệnh viện bắt đầu dùng cổng `[BA đề xuất — TH-C14]` |
| BR-06.18 | Tài khoản nội bộ gắn nhân sự đang làm; khóa khi nhân sự nghỉ (sau BR-06.7); phương thức xác thực (mật khẩu, xác thực hai lớp, đăng nhập một lần) `[CẦN LÀM RÕ VỚI KH]` (NFR nhóm 2) |
| BR-06.19a | Cấp, đổi, thu hồi vai trò/phạm vi do người đề xuất (Trưởng phòng hoặc Hành chính – Nhân sự) khác người thực hiện (IT nội bộ); vai trò duyệt tài chính và Ban giám đốc cần PGĐ/Giám đốc xác nhận; lưu vết trước/sau `[BA đề xuất]` |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-06.2.1 | Happy | BR-06.10, BR-06.12 | R4 | **Given** nhân viên triển khai T được điều động vào dự án P1 **When** T tìm dự án **Then** chỉ thấy P1 và các dự án khác T được điều động |
| AC-06.2.2 | Exception | BR-06.11 | NFR-G2.1 | **Given** người dùng không có quyền với dự án P2 **When** mở trực tiếp đường dẫn của P2 hoặc gọi truy vấn trực tiếp **Then** hệ thống từ chối và không trả dữ liệu |
| AC-06.2.3 | Exception | BR-06.13 | R4 | **Given** người đề xuất một tham số cũng là người duyệt **When** bấm duyệt **Then** hệ thống chặn ở máy chủ |
| AC-06.2.4 | Exception | BR-06.14 | G22 | **Given** người dùng là IT nội bộ **When** thử sửa giá trị tỷ lệ chia sẻ **Then** hệ thống từ chối; IT nội bộ chỉ thấy khung tham số |
| AC-06.2.5 | Edge | BR-06.15 | BR-02.21 | **Given** nhân viên Kinh doanh **When** mở màn đối tác hoặc đối soát **Then** không thấy tỷ lệ chia sẻ, ngoại lệ hay giá trị đối soát, kể cả qua tìm kiếm và xuất dữ liệu |
| AC-06.2.6 | Exception | BR-06.16 | NFR-03.1 | **Given** tài khoản của bệnh viện A **When** sửa đường dẫn để mở yêu cầu hoặc tệp của bệnh viện B **Then** truy cập thất bại, không lộ sự tồn tại của dữ liệu |
| AC-06.2.7 | Edge | BR-06.12 | BR-01.8 | **Given** Trưởng phòng Chuyển giao đổi Trưởng nhóm của dự án P **When** hệ thống ghi sự kiện đổi **Then** Trưởng nhóm mới có quyền ghi mốc của P và Trưởng nhóm cũ không còn quyền ghi nhưng xem được lịch sử phần mình |
| AC-06.2.8 | Happy | BR-06.17 | TH-C14 | **Given** CSKH cấp tài khoản IT bệnh viện cho bệnh viện A **When** IT bệnh viện tạo thêm 3 người dùng và gán Lãnh đạo cho 1 người **Then** 3 người dùng chỉ thấy dữ liệu bệnh viện A và sự kiện tạo được ghi vết `[BA đề xuất]` |
| AC-06.2.9 | Edge | BR-06.10 | R4 | **Given** một người có vai trò Trưởng phòng Chuyển giao và Trưởng nhóm một dự án **When** thao tác **Then** quyền là hợp hai vai trò nhưng quy tắc tách nhiệm vụ vẫn áp dụng cho từng thao tác duyệt |
| AC-06.2.10 | Edge | BR-06.19a | R4 | **Given** Trưởng phòng đề xuất cấp vai trò Kế toán cho một nhân viên **When** IT nội bộ thực hiện nhưng chưa có xác nhận PGĐ Tài chính **Then** hệ thống chưa gán vai trò `[BA đề xuất]` |

---

### FR-06.3 — Khung cấu hình tham số
- **Mô tả**: Cơ chế chung để mọi con số nghiệp vụ là tham số có mặc định, ngoại lệ có căn cứ, hiệu lực theo ngày, duyệt hai người và lưu vết.
- **Input – Xử lý – Output**: Input: giá trị, phạm vi, hiệu lực, căn cứ. Xử lý: kiểm tra hợp lệ, duyệt, thêm bản ghi mới, đóng bản cũ, cảnh báo thiếu tham số. Output: tham số hiệu lực tại ngày bất kỳ; danh mục P01…P23.
- **Màn hình UI**: `[WIREFRAME: Danh sách tham số]` · `[WIREFRAME: Đề xuất và duyệt tham số]` · `[WIREFRAME: Cảnh báo thiếu tham số]`

**Luồng xử lý:**
- **Happy**: người đề xuất nhập giá trị → người duyệt duyệt → bản ghi mới có hiệu lực từ ngày chỉ định.
- **Exception**: giá trị ngoài khoảng; ngày hiệu lực không sau bản hiện hành; tự duyệt; thiếu căn cứ khi ngoại lệ.
- **Edge/Boundary**: đổi tham số khi đầu việc đang mở; ngoại lệ chồng khoảng; tham số thiếu; đổi theo đối tác.
- **Alternative**: ngoại lệ theo phạm vi hẹp.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-06.19 | Mọi tham số (danh mục ở mục 3.2) là bản ghi có: mã, tên, phạm vi áp dụng (toàn hệ thống / đối tác / sản phẩm / dự án / phòng), giá trị mặc định, giá trị ngoại lệ kèm căn cứ, hiệu lực từ–đến, người đề xuất, người duyệt (R5, NFR-G7.1) |
| BR-06.20 | Đổi giá trị luôn tạo **bản ghi mới** có ngày hiệu lực sau bản hiện hành; bản cũ giữ nguyên và hết hạn vào ngày liền trước; giá trị áp theo ngày của sự kiện gốc (ví dụ ngày BBNT, ngày sinh đầu việc); đầu việc đang mở giữ hạn đã tính (BR-05.17, BR-02.6) |
| BR-06.21 | Hai nhóm duyệt: *Tài chính* (P08–P13): Kế toán đề xuất, PGĐ Tài chính duyệt, Giám đốc duyệt khi vượt ngưỡng (FR-02.4, G22). *Thường* (các tham số còn lại, danh mục cấu hình, QN): Trưởng phòng liên quan đề xuất, PGĐ phụ trách mảng duyệt `[BA đề xuất]`, PGĐ phụ trách từng phòng `[CẦN LÀM RÕ VỚI KH]`; người đề xuất khác người duyệt |
| BR-06.22 | Giá trị phải hợp lệ theo kiểu và khoảng (tỷ lệ 0–100%, số ngày > 0, ngày hiệu lực hợp lệ); sai thì chặn lưu |
| BR-06.23 | Thiếu tham số: hệ thống không tự đoán, không tính quá hạn hay chặn nghiệp vụ không có căn cứ; hiển thị cảnh báo "thiếu tham số" cho người quản trị tham số của nhóm (Kế toán cho nhóm Tài chính; Trưởng phòng liên quan cho nhóm Thường) (BR-02.9, BR-02.13, BR-05.15) |
| BR-06.24 | Ngoại lệ bắt buộc căn cứ (tệp hoặc ghi chú có tham chiếu văn bản), phạm vi rõ, ngày hiệu lực và hết hạn; không chồng khoảng hiệu lực cùng phạm vi (BR-02.3, BR-02.11) |
| BR-06.25 | Quyền xem: tham số tài chính chỉ vai trò có quyền (BR-06.15); tham số thời hạn không nhạy cảm thì người dùng liên quan thấy (ví dụ nhân viên Kinh doanh thấy hạn 10 ngày) để hiểu hạn của mình |
| BR-06.26 | Giá trị mới áp dụng ngay khi đến ngày hiệu lực, không cần phát hành bản mới; có chức năng "xem giá trị hiệu lực tại ngày X" để kiểm tra (NFR-G7.1) |
| BR-06.27 | Danh mục cấu hình không phải số (mục 3.3) dùng cùng cơ chế hiệu lực, duyệt và vết; danh mục rủi ro và loại yêu cầu thay đổi không làm đổi trạng thái yêu cầu đang xử lý |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-06.3.1 | Happy | BR-06.19, BR-06.20, BR-06.26 | R5, US-09 | **Given** Trưởng phòng Chuyển giao đề xuất P01 từ 5 xuống 3 ngày làm việc hiệu lực 01/12 và PGĐ phụ trách duyệt **When** đến 01/12 **Then** bản ghi mới có hiệu lực, bản cũ hết hạn 30/11 và không cần phát hành bản mới |
| AC-06.3.2 | Exception | BR-06.21 | G22 | **Given** Kế toán đề xuất tỷ lệ chia sẻ mới **When** chính Kế toán bấm duyệt **Then** hệ thống chặn; PGĐ Tài chính duyệt thì có hiệu lực |
| AC-06.3.3 | Exception | BR-06.22 | R5 | **Given** nhập P01 bằng 0 hoặc -3 ngày **When** lưu **Then** hệ thống chặn và nêu khoảng hợp lệ |
| AC-06.3.4 | Exception | BR-06.20 | BR-02.6 | **Given** ngày hiệu lực không sau bản hiện hành **When** gửi duyệt **Then** hệ thống chặn |
| AC-06.3.5 | Edge | BR-06.23 | BR-02.13 | **Given** đối tác chưa có P07 **When** dự án kênh A ở "Chờ nghiệm thu" **Then** không tính quá hạn và cảnh báo "thiếu tham số" tới Trưởng phòng liên quan |
| AC-06.3.6 | Edge | BR-06.20 | BR-05.17 | **Given** đầu việc nộp BBNT sinh ngày 28/11 hạn 5 ngày **When** P01 đổi thành 3 ngày hiệu lực 01/12 **Then** đầu việc đang mở giữ hạn cũ; đầu việc sinh từ 01/12 dùng 3 ngày |
| AC-06.3.7 | Edge | BR-06.24 | BR-02.11 | **Given** đã có ngoại lệ tỷ lệ cho dự án D trong 01/09–30/09 **When** đề xuất ngoại lệ chồng khoảng cho D **Then** hệ thống chặn |
| AC-06.3.8 | Edge | BR-06.25 | BR-06.15 | **Given** nhân viên Kinh doanh **When** mở màn tham số **Then** thấy hạn ký hợp đồng (P02) và không thấy tham số nhóm Tài chính |
| AC-06.3.9 | Happy | BR-06.26 | NFR-G7.1 | **Given** tham số đổi hiệu lực 01/12 **When** người dùng chọn "xem tại ngày 15/11" và "xem tại ngày 05/12" **Then** hiển thị giá trị cũ cho 15/11 và giá trị mới cho 05/12 |
| AC-06.3.10 | Edge | BR-06.27 | R5 | **Given** Phần mềm đang xử lý yêu cầu phân loại phổ thông **When** thêm "xóa hàng loạt" vào danh mục rủi ro **Then** yêu cầu đang xử lý không đổi trạng thái; yêu cầu mới theo danh mục mới |

---

### FR-06.4 — Lưu vết (nhật ký bất biến) và tra cứu
- **Mô tả**: Ghi nhật ký mọi phê duyệt và thay đổi quan trọng; không ai sửa hay xóa; tra cứu theo đối tượng.
- **Input – Xử lý – Output**: Input: thao tác thuộc danh sách "quan trọng". Xử lý: tạo đúng một bản ghi vết, chặn sửa/xóa. Output: lịch sử từng đối tượng, tra cứu theo người/thời gian.
- **Màn hình UI**: `[WIREFRAME: Lịch sử của đối tượng]` · `[WIREFRAME: Tra cứu nhật ký]`

**Luồng xử lý:**
- **Happy**: thao tác quan trọng → tạo bản ghi vết → người có quyền xem lịch sử.
- **Exception**: thao tác sửa/xóa bản ghi vết; xem vết ngoài quyền.
- **Edge/Boundary**: thao tác hệ thống (nhắc, nhập ban đầu); nhiều thao tác một lúc; vết của trường nhạy cảm; tệp đính kèm của bệnh viện.
- **Alternative**: xuất nhật ký ra tệp.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-06.28 | Nhật ký chỉ thêm, không sửa, không xóa, kể cả với quản trị; mỗi bản ghi: đối tượng, hành động, người và vai trò (hoặc "hệ thống"), thời điểm, giá trị trước/sau, căn cứ/lý do, nguồn (giao diện / hệ thống / nhập ban đầu) (NFR-G10.1, NFR-06.1) |
| BR-06.29 | Danh sách "phê duyệt và thay đổi quan trọng" (chốt tại đây, tổng hợp từ FR-01…05): mốc và sửa mốc, chuyển trạng thái hai trục dự án, BBNT, hợp đồng đã ký, hóa đơn/đòi nợ/thu tiền, đổi người phụ trách hoặc Trưởng nhóm, điều động, danh sách tồn và xác nhận nhận/trả lại, tỷ lệ/chu kỳ/ngoại lệ/dung sai, dòng đối soát và điều chỉnh, chốt BBĐS, phê duyệt từng cổng yêu cầu (IT bệnh viện, lãnh đạo bệnh viện, PGĐ Chuyên môn, Giám đốc), báo giá và chấp nhận, thay đổi tham số và danh mục cấu hình, đổi vai trò/quyền/tài khoản, gộp/ngừng danh mục, nhắc và leo thang |
| BR-06.30 | Mỗi thao tác trong danh sách tạo đúng một bản ghi vết, không thiếu, không trùng (NFR-G10.1) |
| BR-06.31 | Tra cứu: mỗi đối tượng có tab "Lịch sử"; tìm theo người, đối tượng, loại hành động, khoảng thời gian; quyền xem theo phạm vi quyền của đối tượng (vết tỷ lệ chia sẻ chỉ người có quyền xem tỷ lệ); IT nội bộ xem vết kỹ thuật, không xem giá trị tài chính |
| BR-06.32 | Thời gian lưu nhật ký `[CẦN LÀM RÕ VỚI KH]`; mặc định lưu trong suốt vòng đời hệ thống `[BA đề xuất]` |
| BR-06.33 | `[BA đề xuất]` Lượt xem tệp đính kèm của yêu cầu bệnh viện bởi nhân sự Công ty A được ghi vết (ai, khi nào, tệp nào) vì dữ liệu có thể nhạy cảm (G16) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-06.4.1 | Happy | BR-06.28, BR-06.30 | R4 | **Given** PGĐ Tài chính chốt BBĐS **When** thao tác hoàn tất **Then** có đúng một bản ghi vết (người, thời điểm, trạng thái kỳ trước/sau, căn cứ) |
| AC-06.4.2 | Exception | BR-06.28 | NFR-06.1 | **Given** bản ghi vết đã tạo **When** bất kỳ người dùng, kể cả quản trị, thử sửa hoặc xóa **Then** hệ thống từ chối và ghi nhận nỗ lực đó |
| AC-06.4.3 | Happy | BR-06.31 | R4 | **Given** dự án P có 12 bản ghi vết **When** Trưởng phòng có quyền mở tab "Lịch sử" **Then** thấy đủ 12 bản ghi theo thứ tự thời gian, lọc được theo người |
| AC-06.4.4 | Exception | BR-06.31 | NFR-G2.1 | **Given** nhân viên Kinh doanh **When** mở lịch sử của tỷ lệ chia sẻ **Then** hệ thống không hiển thị |
| AC-06.4.5 | Edge | BR-06.28 | R4 | **Given** hệ thống sinh nhắc quá hạn **When** gửi nhắc **Then** bản ghi vết có người là "hệ thống", nguồn "hệ thống" và nội dung quy tắc |
| AC-06.4.6 | Edge | BR-06.30 | NFR-G10.1 | **Given** thao tác "sửa mốc" thay đổi hai trường **When** lưu **Then** hệ thống tạo đúng 1 bản ghi vết chứa giá trị trước/sau của cả hai trường |
| AC-06.4.7 | Edge | BR-06.33 | G16 | **Given** nhân viên CSKH mở tệp đính kèm của yêu cầu bệnh viện **When** xem **Then** lượt xem ghi vết người, thời điểm, tệp `[BA đề xuất]` |
| AC-06.4.8 | Edge | BR-06.28 | R4 | **Given** dữ liệu nhập ban đầu **When** nạp một dự án **Then** bản ghi vết có nguồn "nhập ban đầu" |

---

### FR-06.5 — Ngày bắt đầu đo và nhập dữ liệu ban đầu
- **Mô tả**: Đặt ngày bắt đầu đo; nạp danh mục nền và dự án đang mở; không đoán trạng thái chưa rõ.
- **Input – Xử lý – Output**: Input: ngày bắt đầu đo, tệp danh mục, tệp dự án đang mở. Xử lý: kiểm tra, xem trước, nạp, gắn "Cần xác minh", giao việc xác minh. Output: dữ liệu nền sẵn sàng, danh sách việc cần xác minh.
- **Màn hình UI**: `[WIREFRAME: Nhập dữ liệu ban đầu]` · `[WIREFRAME: Danh sách dự án cần xác minh]`

**Luồng xử lý:**
- **Happy**: đặt ngày bắt đầu đo → nạp danh mục → nạp dự án đang mở → người phụ trách xác minh trạng thái chưa rõ.
- **Exception**: tệp sai định dạng; dòng thiếu trường; trùng dự án/khách hàng.
- **Edge/Boundary**: dự án 3/4 không xác định đã ký hợp đồng/xuất hóa đơn; dự án đã đóng; đổi ngày bắt đầu đo; nạp lại.
- **Alternative**: nhập tay từng dự án.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-06.34 | "Ngày bắt đầu đo" (P22) là tham số toàn hệ thống đặt trước khi nạp dự án; nhắc, leo thang và số liệu SLA chỉ tính cho mốc có ngày gốc từ ngày này trở đi; đổi ngày cần PGĐ phụ trách duyệt, có lý do, lưu vết (BR-05.18, BR-05.37) |
| BR-06.35 | MVP chỉ nạp danh mục nền và **dự án đang mở** (chưa Đóng); dự án đã đóng và dữ liệu lịch sử nạp sau theo yêu cầu (G26) |
| BR-06.36 | Mỗi dự án đang mở nạp: khách hàng, sản phẩm, kênh, đối tác hoặc hợp đồng, trạng thái hai trục, các mốc đã biết, người phụ trách hiện tại; trạng thái không xác định được ghi "Cần xác minh", không đoán (đề: ~3/4 dự án không xác định đã ký hợp đồng/xuất hóa đơn) |
| BR-06.37 | Dự án "Cần xác minh" sinh đầu việc xác minh cho người phụ trách hiện tại (hạn là tham số `[CẦN LÀM RÕ VỚI KH]`); xác minh xong chuyển về trạng thái thật; dự án "Cần xác minh" không tính vào số liệu tuân thủ và hiển thị riêng (BR-05.30, BR-05.37) |
| BR-06.38 | Nhập từ tệp mẫu: kiểm tra, xem trước, báo lỗi từng dòng, phát hiện trùng; chỉ nạp các dòng hợp lệ nếu người nhập xác nhận; bản ghi nạp ghi nguồn "nhập ban đầu" và người nhập (BR-06.28) |
| BR-06.39 | Số dự án "Cần xác minh" và đã xác minh hiển thị cho PGĐ/Trưởng phòng như chỉ số độ phủ dữ liệu |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-06.5.1 | Happy | BR-06.34 | G26 | **Given** ngày bắt đầu đo đặt 01/12 **When** hệ thống đánh giá quy tắc nhắc **Then** chỉ mốc có ngày gốc từ 01/12 mới sinh nhắc và vào số liệu |
| AC-06.5.2 | Happy | BR-06.35, BR-06.38 | G26 | **Given** tệp 40 dự án đang mở hợp lệ **When** người nhập xem trước rồi xác nhận **Then** hệ thống nạp 40 dự án với nguồn "nhập ban đầu" |
| AC-06.5.3 | Exception | BR-06.38 | R5 | **Given** tệp có 3 dòng thiếu khách hàng **When** xem trước **Then** hệ thống báo 3 dòng lỗi kèm số dòng và chỉ nạp phần hợp lệ nếu người nhập xác nhận |
| AC-06.5.4 | Edge | BR-06.36 | Đề bài | **Given** dự án đã xong triển khai nhưng người nhập không biết đã ký hợp đồng hay chưa **When** nạp **Then** trục doanh thu ghi "Cần xác minh" thay vì đoán |
| AC-06.5.5 | Happy | BR-06.37 | G26 | **Given** dự án "Cần xác minh" **When** Kinh doanh phụ trách xác minh "đã ký hợp đồng ngày 10/10, chưa xuất hóa đơn" **Then** dự án chuyển "Chờ xuất hóa đơn" và đầu việc xác minh tự hoàn thành, có vết |
| AC-06.5.6 | Edge | BR-06.37 | BR-05.37 | **Given** 12 dự án "Cần xác minh" **When** PGĐ Tài chính mở báo cáo tuân thủ **Then** 12 dự án bị loại khỏi tỷ lệ và hiển thị "12 dự án cần xác minh" |
| AC-06.5.7 | Edge | BR-06.35 | G26 | **Given** tệp chứa dự án đã Đóng **When** nạp **Then** hệ thống từ chối các dòng đó và nêu "MVP chỉ nạp dự án đang mở" |
| AC-06.5.8 | Edge | BR-06.34 | BR-05.18 | **Given** người có quyền đổi ngày bắt đầu đo sang ngày sau **When** PGĐ phụ trách duyệt kèm lý do **Then** ngày mới có hiệu lực, vết lưu giá trị cũ/mới và số liệu được tính lại theo ngày mới |

---

## 5. Business Rule chung của tính năng
Các BR áp dụng cho toàn tính năng: **BR-06.1** (kênh ở Dự án, không ở Khách hàng), **BR-06.11** (mặc định từ chối, kiểm tra ở máy chủ), **BR-06.13** (người đề xuất khác người duyệt), **BR-06.20** (đổi tham số bằng bản ghi mới có hiệu lực theo ngày), **BR-06.28** (nhật ký bất biến). Mã `BR-06.n` duy nhất trong toàn tính năng (mã `BR-06.19a` bổ sung riêng cho quy trình cấp quyền để giữ nguyên `BR-06.19` làm tham chiếu của phụ lục tham số).

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Khác biệt — FR-06 là nền cho tính toàn vẹn và tách nhiệm vụ của mọi phê duyệt; dữ liệu vết phải đáng tin tuyệt đối`.

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| NFR-06.1 | Truy vết / Toàn vẹn | 100% thao tác sửa hoặc xóa bản ghi nhật ký bị từ chối, kể cả tài khoản quản trị. Đo bằng kiểm thử: thử sửa/xóa bản ghi vết bằng từng vai trò (kể cả IT nội bộ) qua giao diện và truy cập trực tiếp, kết quả mong đợi 100% bị từ chối và có bản ghi nỗ lực. *(nguồn: R4; BA đề xuất làm rõ NFR-G10.1)* | Chặt hơn NFR-G10.1: NFR chung đòi ghi vết, ở đây cấm sửa/xóa vết |
| NFR-06.2 | Bảo mật / Tách nhiệm vụ | 0 trường hợp người đề xuất cũng là người duyệt đối với mọi thao tác loại "duyệt" (tham số, ngoại lệ, điều chỉnh đối soát, cấp quyền nhạy cảm). Đo bằng kiểm thử: với từng thao tác duyệt, dùng cùng một tài khoản đề xuất rồi duyệt, kết quả mong đợi 100% bị chặn ở máy chủ. *(nguồn: R4; G22; BA đề xuất)* | Bổ sung cho NFR-G2.1: NFR chung kiểm quyền theo vai trò/phạm vi, ở đây kiểm thêm quy tắc "hai người" |

## 6b. Phân quyền
> Action: V xem · C tạo · E sửa · A duyệt/xác nhận · X thực hiện. Ma trận đầy đủ do skill 1.5.

| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-06.1 Tạo/sửa Khách hàng | Nhân viên Kinh doanh, CSKH, Chuyển giao | C, E | Trong phạm vi quyền; không sửa khóa nhận diện sau khi có dự án `[BA đề xuất]` |
| FR-06.1 Gộp khách hàng trùng | Trưởng phòng Kinh doanh | X | Lưu vết `[BA đề xuất]` |
| FR-06.1 Đối tác và hợp đồng khung | Kế toán (nhập), PGĐ Tài chính (duyệt) | C, A | Khác người nhập |
| FR-06.1 Sản phẩm | Trưởng phòng Kinh doanh | C, E | `[BA đề xuất]` |
| FR-06.1 Phòng ban, nhân sự | Hành chính – Nhân sự | C, E | Chuyển việc trước khi khóa (BR-06.7) |
| FR-06.2 Gán/đổi vai trò, phạm vi | Trưởng phòng hoặc Hành chính – Nhân sự (đề xuất); IT nội bộ (thực hiện) | C, X | Vai trò nhạy cảm cần PGĐ/Giám đốc xác nhận `[BA đề xuất]` |
| FR-06.2 Quản lý người dùng bệnh viện | IT bệnh viện | C, E | Chỉ bệnh viện mình |
| FR-06.3 Đề xuất tham số nhóm Tài chính | Kế toán | C | Có căn cứ nếu ngoại lệ |
| FR-06.3 Duyệt tham số nhóm Tài chính | PGĐ Tài chính (Giám đốc khi vượt ngưỡng) | A | Khác người đề xuất |
| FR-06.3 Đề xuất, duyệt tham số nhóm Thường | Trưởng phòng liên quan (đề xuất); PGĐ phụ trách (duyệt) | C, A | `[CẦN LÀM RÕ VỚI KH]` |
| FR-06.3 Quản trị khung tham số | IT nội bộ | C, E | Không sửa giá trị nghiệp vụ |
| FR-06.4 Xem lịch sử đối tượng | Theo quyền xem đối tượng đó | V | Vết tài chính theo quyền xem tài chính |
| FR-06.4 Tra cứu nhật ký tổng | Giám đốc, PGĐ (phạm vi mảng), IT nội bộ (vết kỹ thuật) | V | — |
| FR-06.5 Nhập ban đầu | Người được giao (Kế toán/Kinh doanh/Chuyển giao theo dữ liệu) | C, X | Sau khi đặt ngày bắt đầu đo |
| FR-06.5 Đặt/đổi ngày bắt đầu đo | PGĐ phụ trách | A | Lưu vết, có lý do |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| Danh mục khách hàng nhập sai/trùng từ đầu | Dự án, yêu cầu gắn sai khách; báo cáo theo khách lệch | Chống trùng (BR-06.2), gộp có vết, nhập ban đầu có xem trước |
| Phân quyền thiết kế quá chặt hoặc quá lỏng | Người dùng không làm được việc, hoặc lộ dữ liệu | Ma trận quyền chạy thử với từng vai trò (skill 1.5, 3); mặc định từ chối nhưng review với Trưởng phòng |
| Quyền của PGĐ Kinh doanh chưa rõ (G13) | Thiếu quyền xem/duyệt ở nhiều FR | Hỏi KH; tạm thời xem phạm vi mảng, không xem tỷ lệ chia sẻ |
| Nhiều tham số chưa có giá trị (mục 3.2) | Một số FR chưa chạy đúng | Cảnh báo "thiếu tham số", hỏi KH trước khi phát hành |
| Cô lập dữ liệu giữa bệnh viện thất bại | Lộ dữ liệu y tế, mất uy tín | NFR-03.1, kiểm thử xuyên bệnh viện, ghi vết xem tệp (BR-06.33) |
| Người dùng bệnh viện đổi người liên tục | Danh sách thẩm quyền lỗi thời | IT bệnh viện tự quản lý (BR-06.17); danh mục người có thẩm quyền (BR-03.11) |
| Dữ liệu nạp ban đầu không đáng tin | Cảnh báo giả, mất niềm tin | "Cần xác minh", ngày bắt đầu đo, xem trước trước khi nạp |
| Phụ thuộc: mọi FR dùng FR-06 | Chậm FR-06 chặn toàn MVP | Phát hành đầu tiên cùng FR-05; chốt `KHACH_HANG`, `DOI_TAC`, `NHAN_SU`, `THAM_SO` trước (A.5) |
| Hai chiều với FR-02.4 (tham số tài chính) và FR-05.2 (QN) | Đổi khung tham số ảnh hưởng cả hai | Giữ khung ở FR-06.3, quy tắc nghiệp vụ ở FR gốc |
| Chưa có NFR hiệu năng, sao lưu/phục hồi, pháp lý dữ liệu cá nhân (G16) | Rủi ro triển khai | Hỏi KH ở các câu hỏi dưới |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-06.1 và FR-06.2: danh mục nền, danh sách vai trò, phạm vi dữ liệu, ma trận quyền sơ bộ; xác nhận G13, chủ sở hữu từng danh mục | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-06.3 và FR-06.4: khung tham số P01…P23, hai nhóm duyệt, nhật ký và danh sách thay đổi quan trọng | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-06.5: ngày bắt đầu đo, nhập thử 5–10 dự án đang mở thật, quy trình "Cần xác minh"; xác nhận G26 | Chưa bắt đầu |

## 9. Traceability (mini)
> Cột Model, Wireframe, Test để các bước sau điền (⏳).

{{TRACE}}

**NFR liên quan:**
| Mã NFR | Nguồn | NFR Test Scenario |
|---|---|---|
| NFR-06.1 | Mục 6 tính năng này (R4) | ⏳ |
| NFR-06.2 | Mục 6 tính năng này (R4, G22) | ⏳ |
| NFR-G2.1 | Backbone mục 3 (R4) | ⏳ |
| NFR-G7.1 | Backbone mục 3 (R5) | ⏳ |
| NFR-G8.1 | Backbone mục 3 (R1) | ⏳ |
| NFR-G10.1 | Backbone mục 3 (R4) | ⏳ |

## 10. Câu hỏi cần làm rõ với KH (tổng hợp)
| STT | Vị trí trong spec | Câu hỏi cần làm rõ | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | BR-06.12, BR-06.15 | PGĐ Kinh doanh có quyền gì trong hệ thống (xem dự án, xem tỷ lệ chia sẻ, duyệt gì)? Trưởng nhóm thay giữa chừng thì quyền xử lý ra sao? (G13) | Nice-to-know | Chưa hỏi |
| 2 | BR-06.2 | Khóa nhận diện khách hàng là gì (mã cơ sở khám chữa bệnh, mã số thuế, tên)? | Nice-to-know | Chưa hỏi |
| 3 | BR-06.9 | Ai quản lý từng danh mục (khách hàng, đối tác, sản phẩm, nhân sự)? | Nice-to-know | Chưa hỏi |
| 4 | BR-06.21 | Tham số thời hạn nội bộ (5/10/15 ngày…) do ai đề xuất và ai duyệt; mỗi phòng thuộc PGĐ nào? | Nice-to-know | Chưa hỏi |
| 5 | BR-06.18 | Phương thức đăng nhập mong muốn (mật khẩu, xác thực hai lớp, đăng nhập một lần)? | Nice-to-know | Chưa hỏi |
| 6 | BR-06.32 | Thời gian lưu nhật ký và quy định pháp lý về dữ liệu cá nhân/dữ liệu y tế (G16) | Nice-to-know | Chưa hỏi |
| 7 | BR-06.35, BR-06.36 | Dữ liệu dự án hiện có nằm ở đâu (bảng tính, giấy tờ), định dạng, bao nhiêu dự án đang mở? Có cần nạp dự án đã đóng không? (G26, G17) | Nice-to-know | Chưa hỏi |
| 8 | Mục 3.2 | Các tham số chưa có giá trị: P06, P07, P12–P19, P21 | Nice-to-know | Chưa hỏi |
| 9 | BR-06.9, BR-06.17, BR-06.19a, BR-06.21, BR-06.33 | Xác nhận các bổ sung `[BA đề xuất]` (chủ sở hữu danh mục, IT bệnh viện quản lý người dùng, cấp quyền hai người, duyệt tham số nhóm Thường, ghi vết xem tệp) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1, Phần A (A.5), FR-02.4; tổng hợp danh mục tham số P01…P23 và danh sách thay đổi quan trọng từ FR-01…05 | — | Draft |
