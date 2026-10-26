# Dự án ERP nội bộ

## Giới thiệu

Dự án ERP nội bộ là dự án nghiên cứu và thiết kế hệ thống ERP nội bộ cho Công ty A, tập trung vào quản lý chuỗi dự án, hợp đồng, hóa đơn, công nợ, yêu cầu khách hàng, bàn giao và cảnh báo công việc.

Dự án được xây dựng theo tài liệu SRS Backbone và mô hình nghiệp vụ của Công ty A, nhằm chuẩn hóa quy trình vận hành giữa các phòng ban: Kinh doanh, Chuyển giao, CSKH, Kế toán, Phần mềm, IT bệnh viện và Ban lãnh đạo.

## Mục tiêu của hệ thống

- Theo dõi xuyên suốt chuỗi: Dự án → BBNT → Hợp đồng → Hóa đơn → Thu tiền (kênh B)
- Quản lý kênh A: dự án đối tác, BBNT, kỳ đối soát, BBĐS và chia sẻ doanh thu
- Tiếp nhận và duyệt yêu cầu khách hàng qua cổng, có nhánh rủi ro và tính phí
- Bàn giao triển khai → CSKH với danh sách việc tồn và theo dõi tiến độ xử lý
- Tạo cảnh báo, nhắc việc và "Việc của tôi" theo vai trò và phạm vi dữ liệu
- Hỗ trợ mô hình dữ liệu chung, cấu hình tham số nghiệp vụ và lưu vết thay đổi

## Nền tảng nghiệp vụ chính

- Kênh gắn trực tiếp ở cấp Dự án, không gắn ở Khách hàng
- Tách riêng hai trục trạng thái: Triển khai & bàn giao, và Nghiệm thu & doanh thu
- Mỗi mốc nghiệp vụ có chủ trách nhiệm rõ ràng và nguồn ghi ngày rõ ràng
- Mọi con số nghiệp vụ quan trọng được cấu hình dưới dạng tham số, không hardcode

## 6 Feature chính (FR) và vấn đề mỗi FR giải quyết

### FR-01 — Kênh B: chuỗi dự án → BBNT → hợp đồng → hóa đơn → thu tiền
Mục tiêu: quản lý toàn bộ vòng đời dự án kênh B từ khi tạo dự án, ghi BBNT, ký hợp đồng, xuất hóa đơn, thu tiền đến cảnh báo quá hạn và "đang kẹt ở ai".

Vấn đề chính:
- Không có nguồn dữ liệu duy nhất để biết dự án đã nghiệm thu, ký hợp đồng, xuất hóa đơn, thu tiền chưa.
- Không có quy trình quản lý mốc thời gian và cảnh báo khi trễ.
- Không có đánh dấu rõ người phụ trách từng bước.

Sub-feature:
- FR-01.1: Dự án kênh B, hai trục trạng thái, 4 mốc
- FR-01.2: Nghiệm thu BBNT và hợp đồng
- FR-01.3: Hóa đơn và hồ sơ gửi khách
- FR-01.4: Công nợ và thu tiền

### FR-02 — Kênh A: dự án đối tác, BBNT đối tác, kỳ đối soát, BBĐS
Mục tiêu: quản lý dự án kênh A, tiếp nhận BBNT từ đối tác, đối soát doanh thu theo kỳ, tính phần chia Công ty A và chốt BBĐS.

Vấn đề chính:
- Việc đối soát doanh thu thủ công dễ sai, thiếu kiểm soát và khó truy vết.
- Tỷ lệ chia sẻ và kỳ đối soát cần theo ngày hiệu lực và phụ thuộc vào đối tác, sản phẩm.
- Cần tránh sửa trực tiếp kỳ đã chốt và phải có điều chỉnh sau chốt rõ ràng.

Sub-feature:
- FR-02.1: Dự án kênh A và BBNT đối tác
- FR-02.2: Kỳ đối soát và đối chiếu
- FR-02.3: Chốt BBĐS, điều chỉnh sau chốt, hóa đơn cho đối tác
- FR-02.4: Tham số chia sẻ: tỷ lệ, chu kỳ, ngoại lệ

### FR-03 — Cổng yêu cầu khách hàng (rủi ro + tính phí)
Mục tiêu: quản lý toàn bộ luồng yêu cầu khách hàng từ khi gửi qua cổng, đánh giá rủi ro, duyệt IT bệnh viện, tính phí, báo giá và xử lý sau khi được duyệt.

Vấn đề chính:
- Yêu cầu từ bệnh viện đến nhiều kênh và nguồn khác nhau nhưng cần đi qua một quy trình chuẩn.
- Rủi ro và tính phí phải chạy song song, không được bỏ qua.
- Cần ngăn việc giao Phần mềm khi cổng chưa hoàn tất.

Sub-feature:
- FR-03.1: Gửi yêu cầu qua cổng và CSKH nhập hộ
- FR-03.2: IT bệnh viện duyệt
- FR-03.3: Đánh giá và cổng rủi ro
- FR-03.4: Cổng tính phí, báo giá, miễn/giảm
- FR-03.5: Xử lý, cập nhật khách và đóng

### FR-04 — Bàn giao triển khai → CSKH
Mục tiêu: đảm bảo không có việc tồn sau khi đội triển khai rút khỏi dự án, và việc bàn giao được CSKH xác nhận một cách rõ ràng.

Vấn đề chính:
- Nếu không có danh sách việc tồn và xác nhận nhận bàn giao, dự án dễ bị treo.
- Việc từ xa và việc tồn cần được quản lý như đầu việc có người phụ trách và hạn xử lý.
- Cần có vòng đời trạng thái rõ ràng của trục triển khai và bảo hành.

Sub-feature:
- FR-04.1: Danh sách việc tồn và ghi mốc Rút
- FR-04.2: Rà soát và xác nhận nhận bàn giao
- FR-04.3: Việc từ xa và theo dõi tồn sau bàn giao
- FR-04.4: Vòng đời trục Triển khai & bàn giao

### FR-05 — Bàn làm việc, "Việc của tôi" và cảnh báo SLA
Mục tiêu: biến các mốc, quy định và SLA thành đầu việc tự động, nhắc người phụ trách, cảnh báo quá hạn và truyền thông báo đúng vai trò.

Vấn đề chính:
- Dữ liệu nghiệp vụ có nhiều mốc và thời hạn nhưng không được tự động theo dõi.
- Người phụ trách thường không biết việc nào đang quá hạn hoặc sắp đến hạn.
- Cần một màn hình tổng hợp việc và báo cáo tuân thủ mốc để lãnh đạo theo dõi.

Sub-feature:
- FR-05.1: Đầu việc và trang "Việc của tôi"
- FR-05.2: Bộ nhắc và leo thang theo mốc/SLA
- FR-05.3: Thông báo
- FR-05.4: Bàn làm việc theo vai trò
- FR-05.5: Số liệu SLA, báo cáo nội bộ và phễu doanh thu

### FR-06 — Danh mục nền, phân quyền và tham số
Mục tiêu: cung cấp dữ liệu nền dùng chung, cơ chế quyền theo vai trò và cấu hình tham số để các FR khác chạy đúng và đáng tin.

Vấn đề chính:
- Nhiều FR cùng dùng khách hàng, đối tác, sản phẩm, nhân sự, nhưng thiếu chuẩn hóa dữ liệu.
- Cần có cơ chế phân quyền rõ ràng và theo dõi mọi thay đổi/duyệt bằng nhật ký bất biến.
- Mọi thời hạn và tham số nghiệp vụ cần có hiệu lực theo ngày thay vì hardcode.

Sub-feature:
- FR-06.1: Danh mục nền: Khách hàng, Đối tác và hợp đồng khung, Sản phẩm, Phòng ban và Nhân sự
- FR-06.2: Tài khoản, vai trò, phạm vi dữ liệu và phân quyền
- FR-06.3: Khung cấu hình tham số
- FR-06.4: Lưu vết (nhật ký bất biến) và tra cứu
- FR-06.5: Ngày bắt đầu đo và nhập dữ liệu ban đầu

## Mối liên hệ giữa các FR

- FR-01 và FR-02 tập trung vào nghiệp vụ doanh thu theo kênh B và kênh A.
- FR-03 xử lý đầu vào từ bệnh viện thông qua cổng yêu cầu.
- FR-04 đảm bảo dự án được bàn giao đúng cách sau triển khai.
- FR-05 là lớp hạ tầng hỗ trợ nhắc nợ, "Việc của tôi", cảnh báo và SLAs cho mọi FR.
- FR-06 là nền tảng dữ liệu, quyền, tham số và vết, làm cho các FR khác vận hành đồng bộ và đáng tin.

## Cấu trúc repository

```text
Dự án ERP nội bộ/
├── README.md
├── SRS/
│   ├── 01-backbone-srs.md
│   ├── FR-01-chuoi-du-an-doanh-thu.md
│   ├── FR-02-kenh-a-doi-soat.md
│   ├── FR-03-cong-yeu-cau-khach-hang.md
│   ├── FR-04-ban-giao-cskh.md
│   ├── FR-05-ban-lam-viec-viec-cua-toi.md
│   ├── FR-06-danh-muc-nen-tham-so.md
│   └── ...
├── Model/
│   └── Modeling.md
└── ...
```

## Tài liệu tham khảo chính

- `SRS/01-backbone-srs.md` — backbone SRS, bao gồm định nghĩa nghiệp vụ, actor, giả định, backlog, NFR và traceability
- `Model/Modeling.md` — sơ đồ quy trình, state diagram, ERD và mô hình dữ liệu khái niệm
- `SRS/FR-01...FR-06` — các feature nghiệp vụ chính của hệ thống

## Trạng thái dự án

Tài liệu hiện tại đang ở mức nền tảng/thiết kế nghiệp vụ, tập trung vào:

- khung yêu cầu hệ thống
- mô tả luồng nghiệp vụ chính
- xác định feature backlog và ưu tiên MVP
- định nghĩa dữ liệu và đối tượng nghiệp vụ căn bản

## Lưu ý

Dự án có mục tiêu thực hiện trong giai đoạn MVP theo các tính năng ưu tiên cao như FR-01, FR-02, FR-03 và FR-04, với các tính năng mở rộng tiếp theo ở mức hướng dẫn và cấu hình.

---

Dự án ERP nội bộ là nền tảng hỗ trợ quản trị nội bộ doanh nghiệp, giúp tăng minh bạch vận hành, giảm sai sót trong bàn giao, tăng kiểm soát doanh thu và cải thiện trải nghiệm người dùng trong các quy trình nghiệp vụ trọng tâm.
