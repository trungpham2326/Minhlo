# Minhlo

## Giới thiệu

Minhlo là dự án nghiên cứu và thiết kế hệ thống ERP nội bộ cho Công ty A, tập trung vào quản lý chuỗi dự án, hợp đồng, hóa đơn, công nợ, yêu cầu khách hàng và bàn giao triển khai.

Dự án được xây dựng theo tài liệu SRS Backbone và mô hình nghiệp vụ, nhằm chuẩn hóa quy trình vận hành giữa các phòng ban: Kinh doanh, Chuyển giao, CSKH, Kế toán, Phần mềm, Hành chính – Nhân sự và Ban Giám đốc.

## Mục tiêu của hệ thống

- Theo dõi xuyên suốt chuỗi: Dự án → BBNT → Hợp đồng → Hóa đơn → Thu tiền (kênh B)
- Quản lý kênh A: dự án đối tác, BBNT, kỳ đối soát, BBĐS và chia sẻ doanh thu
- Tiếp nhận và duyệt yêu cầu khách hàng qua cổng, có nhánh rủi ro và tính phí
- Bàn giao triển khai → CSKH với danh sách việc tồn và theo dõi tiến độ xử lý
- Tạo cảnh báo, nhắc việc và "Việc của tôi" theo vai trò và phạm vi dữ liệu
- Hỗ trợ mô hình dữ liệu chung và tham số hóa nghiệp vụ để dễ mở rộng, cấu hình và bảo trì

## Nền tảng nghiệp vụ chính

Hệ thống được thiết kế theo các quyết định nền sau:

- Kênh gắn trực tiếp ở cấp Dự án, không gắn ở Khách hàng
- Tách riêng hai trục trạng thái: Triển khai & bàn giao, và Nghiệm thu & doanh thu
- Mỗi mốc nghiệp vụ có chủ trách nhiệm rõ ràng và nguồn ghi ngày rõ ràng
- Mọi con số nghiệp vụ quan trọng được cấu hình dưới dạng tham số, không hardcode

## Các luồng nghiệp vụ trọng tâm

### 1. Kênh B: chuỗi dự án và doanh thu
- Tạo dự án theo khách hàng, sản phẩm và kênh
- Ghi nhận BBNT và các mốc triển khai
- Hoàn tất hợp đồng, xuất hóa đơn và theo dõi công nợ
- Theo dõi quá hạn theo mốc 5/10/15 ngày

### 2. Kênh A: đối soát doanh thu với đối tác
- Thu thập dự án kênh A do đầu mối đối tác báo
- Ghi nhận BBNT và lập kỳ đối soát
- Chốt BBĐS và tính phần chia theo tỷ lệ hiệu lực
- Quản lý hóa đơn và thu tiền từ đối tác

### 3. Cổng yêu cầu khách hàng
- Người dùng bệnh viện gửi yêu cầu qua cổng
- IT bệnh viện duyệt hoặc từ chối
- Yêu cầu rủi ro cần xác nhận của lãnh đạo bệnh viện
- Yêu cầu có thể tính phí phải qua phê duyệt tính phí và báo giá
- Sau khi đạt đầy đủ điều kiện, yêu cầu được đưa vào giai đoạn xử lý

### 4. Bàn giao triển khai → CSKH
- Trưởng nhóm chuẩn bị danh sách việc tồn khi rút
- CSKH xác nhận nhận bàn giao hoặc trả lại kèm lý do
- Các việc tồn được chuyển thành đầu việc có người phụ trách và thời hạn

## Mô hình dữ liệu và quy trình

Dự án này định nghĩa dữ liệu nghiệp vụ chính như:

- Khách hàng, Đối tác, Sản phẩm, Dự án
- Hợp đồng, Hóa đơn, Thu tiền
- Yêu cầu, Phê duyệt, Tình trạng nghiệp vụ
- Nhân sự, Phòng ban, Điều động, Đầu việc
- Kỳ đối soát, BBĐS và tỷ lệ chia sẻ doanh thu

Bản mô hình dữ liệu khái niệm được lưu trong thư mục `Model/Modeling.md`.

## Cấu trúc repository

```text
Minhlo/
├── README.md
├── SRS/
│   └── 01-backbone-srs.md
├── Model/
│   └── Modeling.md
└── ...
```

## Tài liệu tham khảo chính

- `SRS/01-backbone-srs.md` — backbone SRS, bao gồm định nghĩa nghiệp vụ, actor, giả định, backlog, NFR và traceability
- `Model/Modeling.md` — sơ đồ quy trình, state diagram, ERD và mô hình dữ liệu khái niệm

## Trạng thái dự án

Tài liệu hiện tại đang ở mức nền tảng/thiết kế nghiệp vụ, tập trung vào:

- khung yêu cầu hệ thống
- mô tả luồng nghiệp vụ chính
- xác định feature backlog và ưu tiên MVP
- định nghĩa dữ liệu và đối tượng nghiệp vụ căn bản

## Lưu ý

Dự án có mục tiêu thực hiện trong giai đoạn MVP theo các tính năng ưu tiên cao như FR-01, FR-02, FR-03 và FR-04, với các tính năng mở rộng tiếp theo ở mức hướng đi.

---

Minhlo là nền tảng hỗ trợ quản trị nội bộ doanh nghiệp, giúp tăng minh bạch vận hành, giảm sai sót trong bàn giao, tăng kiểm soát doanh thu và cải thiện trải nghiệm xử lý yêu cầu khách hàng.
