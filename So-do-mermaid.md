# Bộ sơ đồ — ERP Công ty A (Phần A)

> Toàn bộ sơ đồ viết bằng **Mermaid**, tách ra từ tài liệu Phần A. Mã hình khớp với tham chiếu trong file Word (Hình A.2, A.3.1…).
> Xem/xuất ảnh: dán từng khối vào https://mermaid.live, hoặc dùng draw.io (Arrange → Insert → Advanced → Mermaid), hoặc mở file này trong trình xem Markdown có hỗ trợ Mermaid (VS Code, GitHub, Obsidian).
> Mỗi cột trong sơ đồ tuần tự là một **làn (phòng ban)**. Thời hạn mặc định 5/10/15 ngày, 1–2 ngày, 2 tuần, 3 lần lấy từ quy định hiện hành; hạn chưa có trong đề ghi là tham số.

## Mục lục

- Hình A.2: Sơ đồ luồng hiện tại (AS-IS)
- Hình A.3.1: Dự án → doanh thu, tách kênh A và kênh B (FR-01, FR-02)
- Hình A.3.2: Yêu cầu khách hàng qua cổng, có nhánh rủi ro và nhánh tính phí (FR-03)
- Hình A.3.3: Bàn giao Triển khai → CSKH kèm danh sách việc tồn (FR-04)
- Hình A.3.4: Luồng tùy chọn — KK03 Hợp đồng định kỳ và gia hạn (FR-07, mức hướng đi)
- Hình A.4a: Dự án — trục Triển khai & bàn giao (FR-04, FR-01)
- Hình A.4b: Dự án — trục Nghiệm thu & doanh thu, rẽ nhánh kênh A/B (FR-01, FR-02)
- Hình A.4c: Yêu cầu (FR-03)
- Hình A.5: Mô hình dữ liệu khái niệm

---

## Hình A.2 — Sơ đồ luồng hiện tại (AS-IS)

```mermaid
flowchart LR
  subgraph DT["Đối tác bán hàng - kênh A"]
    D0["Đầu mối báo dự án trực tiếp cho Chuyển giao"]
    D1["Gửi lại biên bản"]
  end
  subgraph KD["Kinh doanh - kênh B"]
    K0["Chốt giá, hẹn lịch với Chuyển giao"]
    K1["Nhận BBNT"] --> K2["Làm và ký hợp đồng - quy định 10 ngày"]
    K2 --> K3["Gửi hồ sơ và hóa đơn cho khách"]
    K3 --> K4["Gọi xác nhận sau 1-2 ngày, xin lại hợp đồng gốc sau 2 tuần"]
  end
  subgraph CG["Chuyển giao"]
    C1["Nhận lịch triển khai"] --> C2["Triển khai tại đơn vị hoặc từ xa"]
    C2 --> C3["Thu BBNT"]
    C3 --> C4["Nộp BBNT cho Kinh doanh - quy định 5 ngày"]
  end
  subgraph KT["Kế toán và PGĐ Tài chính"]
    T1["Xuất hóa đơn"] --> T2["Đòi nợ 15 ngày một lần"]
    T2 --> T3["Thu tiền"]
    T4["Đối soát thủ công với đối tác"] --> T5["Xuất hóa đơn cho đối tác"]
  end
  K0 --> C1
  D0 --> C1
  C4 -->|"Kênh B"| K1
  K2 --> T1
  K4 --> T2
  C3 -->|"Kênh A"| D1
  D1 --> T4
  T5 --> T3

  X1["Đ1 Xong kỹ thuật nhưng chưa nghiệm thu hoặc đối tác chưa gửi biên bản - việc kẹt, không ai nhắc"]
  X2["Đ2 Mốc 5 ngày không có ai đo"]
  X3["Đ3 Mốc 10 ngày không ai đo, 3/4 dự án không xác định đã ký hợp đồng"]
  X4["Đ4 Không biết dự án nào đủ điều kiện xuất hóa đơn, 3/4 không xác định đã xuất hay chưa"]
  X5["Đ5 Mốc 15 ngày và 3 lần không ai đo"]
  X6["Đ6 Kênh A không qua Kinh doanh nên không ai thấy dự án tồn tại"]
  X7["Đ7 Đối soát thủ công, tỷ lệ ngoài hệ thống, phụ thuộc một người"]
  X8["Đ8 Ngày triển khai, vận hành, nghiệm thu ghi không thống nhất"]
  C3 -.- X1
  C4 -.- X2
  K2 -.- X3
  T1 -.- X4
  T2 -.- X5
  D0 -.- X6
  T4 -.- X7
  C2 -.- X8
  classDef brk fill:#fde8e8,stroke:#c0392b,color:#7b1d1d;
  class X1,X2,X3,X4,X5,X6,X7,X8 brk;
```

## Hình A.3.1 — Dự án → doanh thu, tách kênh A và kênh B (FR-01, FR-02)

```mermaid
sequenceDiagram
  autonumber
  participant BV as Bệnh viện
  participant DT as Đối tác kênh A
  participant KD as Kinh doanh
  participant CG as Chuyển giao
  participant HT as Hệ thống
  participant KT as Kế toán
  participant TC as PGĐ Tài chính
  Note over KD,DT: Kênh là thuộc tính của Dự án, tách ngay khi tạo Dự án (D1)
  alt Kênh B
    KD->>HT: Tạo Dự án kênh B (khách, sản phẩm, loại dự án, căn cứ triển khai)
    KD->>CG: Hẹn lịch triển khai qua hệ thống
  else Kênh A
    DT->>CG: Đầu mối báo dự án kèm giấy tờ điều động
    CG->>HT: Tạo Dự án kênh A (đối tác, sản phẩm)
  end
  CG->>HT: Điều động nhân sự, ghi mốc Bắt đầu triển khai và Vận hành
  CG->>BV: Đề nghị nghiệm thu
  Note over HT: Xong kỹ thuật mà chưa có BBNT thì ở trạng thái Chờ nghiệm thu, có người phụ trách và số ngày tồn
  HT-->>CG: Nhắc khi Chờ nghiệm thu quá hạn tham số, leo thang lên Trưởng phòng Chuyển giao
  alt Kênh B
    BV-->>CG: Ký BBNT
    CG->>HT: Nộp BBNT trong 5 ngày làm việc kể từ ngày ký
    HT->>KD: Giao việc hoàn tất hợp đồng, hạn 10 ngày kể từ khi nhận BBNT (trừ đấu thầu)
    KD->>HT: Cập nhật hợp đồng đã ký
    Note over HT: Quá hạn thì nhắc Kinh doanh và báo Trưởng phòng Kinh doanh
    KD->>KT: Đề nghị xuất hóa đơn khi đủ điều kiện
    KT->>HT: Ghi số hóa đơn, ngày, giá trị (xuất trên phần mềm kế toán pháp định)
    KD->>BV: Gửi hồ sơ và hóa đơn
    HT-->>KD: Nhắc gọi xác nhận sau 1-2 ngày, nhắc hợp đồng gốc sau 2 tuần
    loop Mỗi 15 ngày đến khi thu đủ
      KT->>BV: Đòi nợ
      KT->>HT: Ghi lần đòi nợ
    end
    Note over HT: Sau 3 lần chưa thu thì báo Trưởng phòng Kế toán
    BV-->>KT: Thanh toán
    KT->>HT: Ghi thu tiền, đóng trục doanh thu
  else Kênh A
    DT->>CG: Gửi BBNT (hạn gửi là tham số)
    CG->>HT: Ghi BBNT, dự án vào Kỳ đối soát theo ngày BBNT
    HT-->>CG: Nhắc khi đối tác chưa gửi BBNT quá hạn
    DT->>TC: Gửi số liệu doanh thu cuối kỳ
    TC->>HT: Đối chiếu danh sách dự án với số liệu đối tác, hệ thống tính phần chia theo tỷ lệ hiệu lực
    alt Số liệu khớp
      TC->>HT: Chốt BBĐS
    else Số liệu lệch
      TC->>DT: Trao đổi dòng lệch, ghi căn cứ điều chỉnh
      TC->>HT: Chốt BBĐS sau điều chỉnh
    end
    KT->>DT: Xuất hóa đơn cho đối tác phần Công ty A được hưởng
    DT-->>KT: Thanh toán
    KT->>HT: Ghi thu tiền
  end
```

## Hình A.3.2 — Yêu cầu khách hàng qua cổng, có nhánh rủi ro và nhánh tính phí (FR-03)

```mermaid
sequenceDiagram
  autonumber
  participant ND as Người dùng bệnh viện
  participant IT as IT bệnh viện
  participant LD as Lãnh đạo bệnh viện
  participant HT as Hệ thống
  participant CS as CSKH
  participant PM as Phần mềm
  participant CM as PGĐ Chuyên môn
  participant KD as Kinh doanh
  participant GD as Giám đốc
  ND->>HT: Nhập yêu cầu trên cổng (nguồn là cổng)
  Note over CS,HT: Yêu cầu qua điện thoại, tin nhắn, email, nhân viên triển khai do CSKH nhập hộ với nguồn tương ứng (G05)
  HT->>IT: Chờ duyệt
  alt IT bệnh viện từ chối
    IT-->>HT: Từ chối kèm lý do
    HT-->>ND: Thông báo và đóng yêu cầu
  else IT bệnh viện duyệt
    IT-->>HT: Duyệt
    HT->>CS: Yêu cầu đã sang Công ty A, chờ đánh giá
    CS->>HT: Phân loại sơ bộ (phổ thông, rủi ro, có thể tính phí)
    PM->>HT: Xác nhận hoặc nâng mức rủi ro, đánh dấu có thể tính phí
    par Cổng rủi ro nếu có
      HT->>LD: Gửi yêu cầu để xác nhận
      alt Lãnh đạo xác nhận
        LD-->>HT: Xác nhận (ghi người, thời điểm, nội dung)
      else Lãnh đạo từ chối
        LD-->>HT: Từ chối, yêu cầu đóng
      end
    and Cổng tính phí nếu có
      HT->>CM: Chờ duyệt tính phí
      CM-->>HT: Duyệt tính phí
      HT->>KD: Giao việc lập báo giá
      KD->>ND: Gửi báo giá cho bệnh viện
      ND-->>HT: Bệnh viện chấp nhận báo giá (có bằng chứng)
      opt Có đề xuất miễn, giảm hoặc ưu đãi
        GD->>HT: Duyệt hoặc không duyệt trước khi thực hiện
      end
    end
    Note over HT: Chỉ khi mọi cổng bắt buộc đã qua thì yêu cầu mới sang Đã duyệt (G19)
    HT->>PM: Giao xử lý
    PM->>HT: Xử lý xong
    HT->>CS: Cập nhật trạng thái
    CS->>ND: Thông báo kết quả cho khách
    ND-->>HT: Xác nhận, đóng yêu cầu
  end
```

## Hình A.3.3 — Bàn giao Triển khai → CSKH kèm danh sách việc tồn (FR-04)

```mermaid
sequenceDiagram
  autonumber
  participant TN as Trưởng nhóm triển khai
  participant HT as Hệ thống
  participant CS as CSKH
  participant TP as Trưởng phòng Chuyển giao
  TN->>HT: Chuẩn bị rút, lập danh sách việc tồn gồm cả việc từ xa, hoặc xác nhận không còn tồn
  Note over HT: Dự án chỉ sang Đã rút - chờ bàn giao khi có danh sách hoặc xác nhận không còn tồn (G12)
  HT->>CS: Giao việc rà soát bàn giao
  CS->>HT: Rà soát danh sách tồn
  alt Danh sách đủ
    CS->>HT: Xác nhận nhận bàn giao
    HT->>HT: Dự án sang Đã bàn giao, mỗi việc tồn thành đầu việc của CSKH có người phụ trách và hạn
  else Danh sách chưa đủ
    CS->>HT: Trả lại kèm lý do
    HT->>TN: Yêu cầu bổ sung danh sách
    HT-->>TP: Báo khi trả lại hoặc chờ quá hạn tham số
  end
  loop Đến khi hết việc tồn
    CS->>HT: Cập nhật tiến độ xử lý việc tồn
  end
  Note over HT: Việc triển khai từ xa tạo đầu việc cùng cấu trúc (hình thức từ xa, người phụ trách, hạn) nên hiện trong Việc của tôi
```

## Hình A.3.4 — Luồng tùy chọn — KK03 Hợp đồng định kỳ và gia hạn (FR-07, mức hướng đi)

```mermaid
sequenceDiagram
  autonumber
  participant KD as Kinh doanh
  participant HT as Hệ thống
  participant KT as Kế toán
  participant BV as Bệnh viện
  KD->>HT: Tạo hợp đồng định kỳ (thuê, bảo trì, thuê máy chủ) với kỳ thanh toán và ngày hiệu lực
  HT->>HT: Sinh lịch kỳ thanh toán và mốc nhắc gia hạn (tham số số tháng mốc)
  loop Mỗi kỳ thanh toán
    HT->>KT: Nhắc xuất hóa đơn kỳ trước hạn tham số
    KT->>HT: Ghi số hóa đơn kỳ và theo dõi công nợ như FR-01
  end
  HT->>KD: Nhắc gia hạn khi đến mốc
  KD->>BV: Đàm phán gia hạn
  alt Gia hạn
    KD->>HT: Tạo phụ lục gia hạn, sinh lịch kỳ mới
  else Không gia hạn
    KD->>HT: Ghi lý do và đóng hợp đồng
  end
  Note over HT: Dự án đã bàn giao hết bảo hành thì nhắc Kinh doanh ký hợp đồng bảo trì
```

## Hình A.4a — Dự án — trục Triển khai & bàn giao (FR-04, FR-01)

```mermaid
stateDiagram-v2
  state "Mới tạo" as MoiTao
  state "Đã điều động" as DaDieuDong
  state "Đang triển khai" as DangTK
  state "Tạm dừng" as TamDung
  state "Đã rút - chờ bàn giao" as DaRut
  state "Đã bàn giao - bảo hành" as DaBanGiao
  state "Đóng" as Dong
  state "Hủy" as Huy
  [*] --> MoiTao
  MoiTao --> DaDieuDong: Cử trưởng nhóm và nhân sự
  DaDieuDong --> DangTK: Ghi mốc Bắt đầu triển khai
  DangTK --> TamDung: Có lý do
  TamDung --> DangTK: Tiếp tục
  DangTK --> DaRut: Ghi mốc Rút, đã có danh sách tồn
  DaRut --> DaBanGiao: CSKH xác nhận nhận
  DaBanGiao --> Dong: Hết bảo hành và trục doanh thu hoàn tất
  MoiTao --> Huy: Có lý do
  DaDieuDong --> Huy: Có lý do
  Dong --> [*]
  Huy --> [*]
```

## Hình A.4b — Dự án — trục Nghiệm thu & doanh thu, rẽ nhánh kênh A/B (FR-01, FR-02)

```mermaid
stateDiagram-v2
  state "Chưa phát sinh" as ChuaPS
  state "Chờ nghiệm thu" as ChoNT
  state "Đã nghiệm thu" as DaNT
  state "Chờ hợp đồng" as ChoHD
  state "Chờ xuất hóa đơn" as ChoHDon
  state "Đã xuất hóa đơn" as DaHDon
  state "Đã gửi hồ sơ" as DaGui
  state "Đang công nợ" as CongNo
  state "Chờ đối soát" as ChoDS
  state "Đã đối soát" as DaDS
  state "Đã xuất hóa đơn cho đối tác" as DaHDonDT
  state "Đã thu" as DaThu
  [*] --> ChuaPS
  ChuaPS --> ChoNT: Xong kỹ thuật, ghi mốc Vận hành
  ChoNT --> DaNT: Ghi nhận BBNT
  DaNT --> ChoHD: Kênh B không đấu thầu
  DaNT --> ChoHDon: Kênh B đấu thầu, hợp đồng có trước
  ChoHD --> ChoHDon: Kinh doanh ký hợp đồng
  ChoHDon --> DaHDon: Kế toán ghi hóa đơn
  DaHDon --> DaGui: Kinh doanh gửi hồ sơ và hóa đơn
  DaGui --> CongNo: Khách xác nhận đã nhận
  CongNo --> DaThu: Thu đủ
  DaNT --> ChoDS: Kênh A
  ChoDS --> DaDS: Chốt BBĐS
  DaDS --> DaHDonDT: Xuất hóa đơn cho đối tác
  DaHDonDT --> DaThu: Thu đủ
  DaThu --> [*]
```

## Hình A.4c — Yêu cầu (FR-03)

```mermaid
stateDiagram-v2
  state "Mới" as Moi
  state "Chờ IT bệnh viện duyệt" as ChoIT
  state "Chờ đánh giá" as ChoDG
  state "Chờ lãnh đạo bệnh viện xác nhận" as ChoLD
  state "Chờ PGĐ Chuyên môn duyệt tính phí" as ChoCM
  state "Chờ Giám đốc duyệt miễn giảm" as ChoGD
  state "Chờ báo giá" as ChoBG
  state "Chờ bệnh viện chấp nhận báo giá" as ChoCN
  state "Đã duyệt" as DaDuyet
  state "Đang xử lý" as DangXL
  state "Đã xử lý" as DaXL
  state "Đóng" as Dong
  [*] --> Moi
  Moi --> ChoIT: Gửi
  ChoIT --> Dong: IT bệnh viện từ chối
  ChoIT --> ChoDG: IT bệnh viện duyệt
  ChoDG --> DaDuyet: Phổ thông
  ChoDG --> ChoLD: Có rủi ro
  ChoDG --> ChoCM: Có thể tính phí
  ChoLD --> DaDuyet: Lãnh đạo xác nhận và các cổng khác đã xong
  ChoLD --> Dong: Lãnh đạo từ chối
  ChoCM --> ChoBG: Duyệt tính phí
  ChoCM --> ChoGD: Đề xuất miễn hoặc giảm
  ChoGD --> ChoBG: Giám đốc duyệt giảm
  ChoGD --> DaDuyet: Giám đốc duyệt miễn và các cổng khác đã xong
  ChoGD --> Dong: Giám đốc không duyệt
  ChoBG --> ChoCN: Kinh doanh gửi báo giá
  ChoCN --> DaDuyet: Bệnh viện chấp nhận và các cổng khác đã xong
  ChoCN --> Dong: Bệnh viện từ chối
  DaDuyet --> DangXL: Phần mềm nhận
  DangXL --> DaXL: Xử lý xong
  DaXL --> Dong: CSKH cập nhật, khách xác nhận
  Dong --> [*]
```

## Hình A.5 — Mô hình dữ liệu khái niệm

```mermaid
erDiagram
  KHACH_HANG ||--o{ DU_AN : "có nhiều"
  KHACH_HANG ||--o{ HOP_DONG : "ký (kênh B)"
  KHACH_HANG ||--o{ YEU_CAU : "gửi"
  SAN_PHAM ||--o{ DU_AN : "thuộc"
  DOI_TAC ||--o{ HOP_DONG_KHUNG : "ký"
  DOI_TAC |o--o{ DU_AN : "bán (kênh A)"
  HOP_DONG |o--o{ DU_AN : "căn cứ (kênh B)"
  HOP_DONG_KHUNG ||--o{ TY_LE_CHIA_SE : "quy định"
  SAN_PHAM |o--o{ TY_LE_CHIA_SE : "áp cho"
  DU_AN ||--o{ BBNT : "có"
  DU_AN ||--o{ DIEU_DONG : "phân công"
  NHAN_SU ||--o{ DIEU_DONG : "tham gia"
  PHONG_BAN ||--o{ NHAN_SU : "gồm"
  DU_AN ||--o{ DAU_VIEC : "có"
  YEU_CAU |o--o{ DAU_VIEC : "sinh ra"
  NHAN_SU ||--o{ DAU_VIEC : "phụ trách"
  DU_AN |o--o{ YEU_CAU : "liên quan"
  YEU_CAU ||--o{ PHE_DUYET : "qua cổng"
  DU_AN |o--o{ HOA_DON : "kênh B"
  BBDS |o--o{ HOA_DON : "kênh A"
  HOA_DON ||--o{ THU_TIEN : "thu"
  DOI_TAC ||--o{ KY_DOI_SOAT : "đối soát"
  KY_DOI_SOAT ||--o{ DONG_DOI_SOAT : "gồm"
  DU_AN ||--o{ DONG_DOI_SOAT : "nằm trong"
  TY_LE_CHIA_SE ||--o{ DONG_DOI_SOAT : "được áp"
  KY_DOI_SOAT ||--o| BBDS : "chốt"
  DU_AN ||--o{ THIET_BI : "quản lý"

  KHACH_HANG {
    string id PK
    string ten
    string loai
  }
  DOI_TAC {
    string id PK
    string ten
  }
  SAN_PHAM {
    string id PK
    string ten
  }
  HOP_DONG_KHUNG {
    string id PK
    string doi_tac_id FK
    date hieu_luc_tu
    date hieu_luc_den
    string chu_ky_doi_soat
  }
  TY_LE_CHIA_SE {
    string id PK
    string hop_dong_khung_id FK
    string san_pham_id FK
    decimal ty_le
    date hieu_luc_tu
    date hieu_luc_den
    string ngoai_le_can_cu
  }
  HOP_DONG {
    string id PK
    string khach_hang_id FK
    string loai
    string trang_thai
    date ngay_ky
    bool ban_goc_da_ve
  }
  DU_AN {
    string id PK
    string khach_hang_id FK
    string san_pham_id FK
    string kenh
    string doi_tac_id FK
    string hop_dong_id FK
    string loai_du_an
    string trang_thai_trien_khai
    string trang_thai_doanh_thu
    date ngay_bat_dau
    date ngay_van_hanh
    date ngay_nghiem_thu
    date ngay_rut
  }
  BBNT {
    string id PK
    string du_an_id FK
    date ngay_ky
    date ngay_nhan
    string nguon
  }
  DIEU_DONG {
    string id PK
    string du_an_id FK
    string nhan_su_id FK
    string vai_tro
    date tu_ngay
    date den_ngay
  }
  NHAN_SU {
    string id PK
    string ho_ten
    string phong_ban_id FK
  }
  PHONG_BAN {
    string id PK
    string ten
  }
  DAU_VIEC {
    string id PK
    string du_an_id FK
    string yeu_cau_id FK
    string loai
    string hinh_thuc
    string nguoi_phu_trach FK
    date han
    string trang_thai
  }
  YEU_CAU {
    string id PK
    string khach_hang_id FK
    string du_an_id FK
    string nguon
    string muc_rui_ro
    bool co_tinh_phi
    string trang_thai
  }
  PHE_DUYET {
    string id PK
    string yeu_cau_id FK
    string cong
    string nguoi_duyet
    datetime thoi_diem
    string ket_qua
  }
  HOA_DON {
    string id PK
    string loai
    string du_an_id FK
    string bbds_id FK
    string so_hoa_don
    date ngay_xuat
    decimal gia_tri
  }
  THU_TIEN {
    string id PK
    string hoa_don_id FK
    date ngay_thu
    decimal so_tien
  }
  KY_DOI_SOAT {
    string id PK
    string doi_tac_id FK
    date tu_ngay
    date den_ngay
    string trang_thai
  }
  DONG_DOI_SOAT {
    string id PK
    string ky_doi_soat_id FK
    string du_an_id FK
    string ty_le_id FK
    decimal gia_tri_doi_tac_bao
    decimal gia_tri_he_thong
    decimal phan_chia
  }
  BBDS {
    string id PK
    string ky_doi_soat_id FK
    date ngay_chot
    string nguoi_xac_nhan
  }
  THIET_BI {
    string id PK
    string du_an_id FK
    string ten_may
    string trang_thai_ket_noi
  }
  THAM_SO {
    string ma PK
    string pham_vi
    string gia_tri_mac_dinh
    string gia_tri_ngoai_le
    string can_cu
    date hieu_luc_tu
  }
```
