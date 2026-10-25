# Phần B — Câu 1 (Story map MVP) và Câu 2 (User story Given/When/Then)

> **Dự án:** ERP Quản trị nội bộ Công ty A (`ERP-CongTyA`) · **Phiên bản:** v1.2 (Draft) · **Ngày:** 26/10/2026 · **Người soạn:** PO/BA
> **Căn cứ:** `Requirement.md` (Phần B, mục 1 và 2) · `01-backbone-srs.md` v1.1 · `FR-01` … `FR-06` (v1.0, 25/10/2026) · `Tinh-huong-phat-sinh-va-edge-case.md` · `So-do-mermaid.md`
> **Phạm vi file này:** chỉ **B.1** (bản đồ câu chuyện người dùng cho MVP, nêu rõ cái gì cố ý để sau) và **B.2** (10–15 user story cho khó khăn bắt buộc KK01, KK02, KK04, KK05, mỗi story có tiêu chí nghiệm thu Given / When / Then). B.3 (ưu tiên), B.4 (quy tắc chia sẻ doanh thu), B.5 (xác nhận giả thuyết) không nằm trong file này.
> **Ký hiệu:** `[BA đề xuất]` = bổ sung chưa có trong Backbone v1.1, chờ Công ty A xác nhận · `[CẦN LÀM RÕ VỚI KH]` = chưa có số liệu · `AC-XX.k.n` = tiêu chí nghiệm thu đã có ở Feature Spec (file này **không viết AC thứ hai**, chỉ chuyển thành Given/When/Then theo từng story và ghi truy vết).

---

## 0. Đã chỉnh gì so với tài liệu hiện có

Backbone mục 2b đã liệt kê US-01…US-15 ở dạng "As a… I want… so that…" và hẹn "Given/When/Then đầy đủ ở Phần B.2", nhưng B.2 chưa tồn tại; còn các Feature Spec chỉ ghi nguồn AC là `US-xx Ak` với số thứ tự `Ak` chưa thống nhất. File này giải quyết ba việc:

| # | Vấn đề tìm thấy | Cách chỉnh |
|---|---|---|
| 1 | Chưa có **story map** (B.1). Backbone mục 4b chỉ có phát hành và MoSCoW theo FR, không có bản đồ theo hành trình người dùng | Viết mới ở mục B.1: xương sống theo chuỗi giá trị, cắt lát trong MVP, danh sách "cố ý để sau" kèm cách MVP xử lý tạm |
| 2 | Chưa có **Given/When/Then theo story** (B.2). Spec chỉ có AC theo sub-feature | Viết mới ở mục B.2: 15 story, mỗi kịch bản `A1…An` ánh xạ về đúng `AC-XX.k.n` |
| 3 | **Số thứ tự `Ak` lệch** giữa các file (ví dụ `AC-01.4.1` và `AC-01.4.2` cùng ghi `US-05 A3`; toàn bộ AC của FR-04 chưa có `Ak`; hai AC cùng ghi `US-02 A1`) | Chốt đánh số `Ak` tại đây; bảng đồng bộ ở mục B.2.4 liệt kê từng ô "Nguồn" cần sửa trong FR-01, FR-02, FR-03, FR-04 và file edge case |

Ngoài ra, việc đối chiếu cho thấy 3 điểm cần PO quyết (không tự ý đưa vào MVP): xem mục B.1.6.

---

# B.1 — Bản đồ câu chuyện người dùng (User Story Map) cho MVP

## B.1.1 Nguyên tắc cắt MVP

1. **MVP phải trả lời được câu hỏi cốt lõi của đề:** "dự án này đã xong, đã có BBNT, đã ký hợp đồng (hoặc đã vào đối soát), đã xuất hóa đơn, đã thu tiền chưa, và kẹt ở ai?" Vì vậy KK01 (chuỗi doanh thu) đứng đầu, kèm cơ chế nhắc/leo thang (FR-05) làm cho mốc 5/10/15 ngày "chạy được" thay vì nằm trên giấy.
2. **MVP gồm đủ 4 khó khăn bắt buộc** (KK01, KK02, KK04, KK05) ở mức đi hết đường từ đầu đến cuối (walking skeleton), không làm sâu từng phần rồi bỏ dở phần khác. Thứ tự lát theo hạng ở Backbone mục 4b: FR-01 → FR-04 → FR-02 → FR-03.
3. **FR-05 (nhắc, "Việc của tôi", bàn làm việc) và FR-06 (danh mục, quyền, tham số, lưu vết) là nền**, không có khó khăn riêng nhưng 15 story đều chạy trên đó nên phát hành cùng nhau (Phát hành 1).
4. **Mọi con số là tham số** (R5): story nào cần số chưa có trong đề thì dùng quy tắc "thiếu tham số → không đoán, không tính quá hạn, cảnh báo người quản trị" (BR-06.23) thay vì chặn cả luồng.
5. **Kênh A ở mức ghi nhận** (R10): không quản lý thầu hay hợp đồng nội bộ của đối tác; đối tác không đăng nhập trong MVP (G24).
6. **SLA chỉ để nhắc và báo cáo, không phạt tự động** (R8) trong mọi story.

## B.1.2 Bản đồ: hoạt động → nhiệm vụ → story

Hàng ngang (xương sống) là chuỗi giá trị từ thiết lập đến bàn giao; mỗi dòng là một nhiệm vụ của một vai trò. Cột "Story MVP" là thứ làm trong Phát hành 1. **Enabler** là việc kỹ thuật cần có nhưng không đủ giá trị người dùng để thành story riêng; AC của chúng nằm ở Feature Spec.

| Hoạt động | Nhiệm vụ người dùng | Vai trò | Story MVP | Cố ý để sau (xem B.1.4) |
|---|---|---|---|---|
| **H0. Thiết lập nền** (FR-06) | Nhập khách hàng, đối tác + hợp đồng khung, sản phẩm, nhân sự | Kinh doanh, CSKH, Kế toán, HC-NS | Enabler (AC-06.1.x) | Hồ sơ nhân sự chi tiết, lương, thưởng |
| | Cấp tài khoản, vai trò, phạm vi dữ liệu; cô lập dữ liệu giữa các bệnh viện | IT nội bộ, IT bệnh viện | Enabler (AC-06.2.x); kiểm chứng ở **US-01 A3**, **US-10 A5** | Ủy quyền duyệt có thời hạn |
| | Đặt ngày bắt đầu đo, nạp dự án đang mở, đánh dấu "Cần xác minh" | PGĐ phụ trách, người được giao | **US-01 A4** | Nạp dự án đã đóng và dữ liệu lịch sử |
| | Cấu hình tham số hiệu lực theo ngày, duyệt hai người, lưu vết | Kế toán, PGĐ Tài chính, Giám đốc | **US-09** | Nhập số liệu từ tệp |
| **H1. Tạo và điều động dự án** | Tạo dự án kênh B kèm căn cứ triển khai | Kinh doanh | Enabler (AC-01.1.1, 01.1.2) | — |
| | Tạo dự án kênh A theo đầu mối đối tác báo, kèm giấy tờ điều động | Chuyển giao | **US-06** | Đối tác tự báo trên hệ thống |
| | Điều động tối thiểu (trưởng nhóm, nhân sự, vai trò, từ ngày) | Trưởng phòng Chuyển giao | Enabler (AC-04.4.1) | Kiểm tra trùng lịch, lệnh công tác, nghỉ bù (FR-08) |
| **H2. Triển khai và nghiệm thu** | Ghi mốc Bắt đầu, Vận hành; tạm dừng/tiếp tục | Trưởng nhóm triển khai | **US-02 A1, A5** | — |
| | Ghi BBNT, nộp cho Kinh doanh trong 5 ngày làm việc | Trưởng nhóm, nhân viên Chuyển giao | **US-02 A2–A4** | — |
| | Theo dõi dự án Chờ nghiệm thu, đổi trưởng nhóm, sửa mốc có vết | Trưởng nhóm, Trưởng phòng | **US-01, US-02 A6–A7** | Hạn nhắc Chờ nghiệm thu (chờ KH, G10) |
| | Theo dõi và ghi BBNT do đối tác gửi (kênh A) | Chuyển giao | **US-07** | — |
| **H3. Hợp đồng và đối soát** | Nhận BBNT, hoàn tất hợp đồng trong 10 ngày; bỏ qua bước này với đấu thầu | Kinh doanh | **US-03** | Nhiều hợp đồng/phụ lục trên một dự án |
| | Lập kỳ đối soát theo đối tác, đối chiếu số liệu, xử lý dòng lệch, chốt BBĐS | PGĐ Tài chính, Kế toán | **US-08** | Dung sai và quy ước làm tròn (mặc định không dung sai) |
| **H4. Hóa đơn và thu tiền** | Gửi hồ sơ và hóa đơn, gọi xác nhận 1–2 ngày, theo dõi hợp đồng gốc 2 tuần | Kinh doanh | **US-04** | — |
| | Ghi hóa đơn, đòi nợ 15 ngày/lần, báo sau 3 lần, ghi thu tiền | Kế toán | **US-05** | Bước leo thang sau lần 3 (chờ KH); tiền giữ lại bảo hành |
| | Ghi hóa đơn cho đối tác sau khi chốt BBĐS (kênh A) | Kế toán | **US-08 A6** | Tích hợp phần mềm kế toán pháp định (ngoài phạm vi, R9) |
| **H5. Yêu cầu khách hàng** (FR-03) | Gửi yêu cầu qua cổng; CSKH nhập hộ yêu cầu ngoài cổng | Người dùng bệnh viện, CSKH | **US-10** | Cảnh báo trùng tự động; luồng khẩn cấp |
| | IT bệnh viện duyệt hoặc từ chối | IT bệnh viện | **US-11** | Quy tắc tự đóng khi IT bệnh viện im lặng |
| | Đánh giá rủi ro, lãnh đạo bệnh viện xác nhận | CSKH, Phần mềm, Lãnh đạo BV | **US-12** | — |
| | Duyệt tính phí, báo giá, miễn/giảm | PGĐ Chuyên môn, Kinh doanh, Giám đốc | **US-13** | Phí yêu cầu thêm của kênh A thu và chia thế nào (G08) |
| **H6. Bàn giao và bảo hành** (FR-04) | Lập danh sách việc tồn (kể cả việc từ xa) hoặc xác nhận không còn tồn, ghi mốc Rút | Trưởng nhóm | **US-14** | Rút gấp khi chưa kịp lập danh sách (chưa đặc tả, xem B.1.6) |
| | Rà soát, nhận hoặc trả lại; theo dõi tồn đọng của đơn vị | CSKH | **US-15** | — |
| | Xác nhận đóng dự án khi đủ điều kiện | CSKH | Enabler (AC-04.4.7) `[BA đề xuất]` | Nhắc ký bảo trì khi hết bảo hành (FR-07) |
| **H7. Xuyên suốt** (FR-05) | "Việc của tôi", nhắc và leo thang, thông báo, bàn làm việc theo vai trò | Mọi người dùng | Nằm trong AC của các story (ví dụ US-01 A1, US-03 A1) | Đẩy thông báo lên điện thoại; báo cáo điều hành đầy đủ (FR-13) |

## B.1.3 Cắt lát phát hành trong MVP

Lát nối tiếp theo thứ tự ưu tiên ở Backbone mục 4b. Mỗi lát **chạy được độc lập** để review với khách hàng theo các mốc (Milestone) đã đặt ở mục 8 của từng Feature Spec.

| Lát | Nội dung | Story | FR | KK | Vì sao ở vị trí này |
|---|---|---|---|---|---|
| **Lát 0 — Nền** | Danh mục, quyền, tham số, nhật ký bất biến, đầu việc và nhắc, thông báo, ngày bắt đầu đo | (không story riêng; điều kiện chạy của 15 story) | FR-05.1–05.3, FR-06.1–06.4 | nền | Mọi story dùng; thiếu nền thì cảnh báo không có người nhận và quyền không cô lập được |
| **Lát 1 — Chuỗi doanh thu kênh B** | Hai trục trạng thái, 4 mốc, BBNT, hợp đồng, hóa đơn, công nợ, bàn làm việc Trưởng nhóm/Kinh doanh/PGĐ Tài chính | US-01, US-02, US-03, US-04, US-05 | FR-01, FR-05.4, FR-06.5 | KK01 | Hạng 1, trả lời câu hỏi cốt lõi; tuân thủ 5/10/15 ngày đo được ngay |
| **Lát 2 — Bàn giao** | Danh sách tồn, rút, CSKH nhận/trả lại, đầu việc tồn, điều kiện đóng dự án | US-14, US-15 | FR-04 | KK05 | WSJF cao nhất (4.50); cùng bản ghi Dự án với Lát 1 nên làm liền để không đổi mô hình hai lần |
| **Lát 3 — Kênh A và đối soát** | Dự án kênh A, BBNT đối tác, tham số chia sẻ, kỳ đối soát, BBĐS, hóa đơn cho đối tác | US-06, US-07, **US-09**, US-08 (làm US-09 trước US-08 vì tham số là đầu vào của phép tính) | FR-02 | KK02 | Phụ thuộc dữ liệu Dự án (Lát 1) và khung tham số (Lát 0); rủi ro làm lại cao nhất ở G03, G04 nên cần back-test BBĐS kỳ trước |
| **Lát 4 — Cổng yêu cầu** | Gửi, IT bệnh viện duyệt, cổng rủi ro, cổng tính phí, xử lý, đóng | US-10, US-11, US-12, US-13 | FR-03 | KK04 | WSJF thấp nhất trong 4 nhưng luồng đã được công ty chốt và việc làm miễn phí gây rò rỉ doanh thu (đã giải thích ở Backbone, PB-3) |

## B.1.4 Cố ý để sau (và MVP xử lý tạm thế nào)

| # | Hạng mục để sau | Vì sao để sau | MVP xử lý tạm | Phát hành |
|---|---|---|---|---|
| 1 | Đối tác đăng nhập, tự nhập dự án, BBNT, số liệu (G24) | Thêm nhóm người dùng bên ngoài làm MVP lớn hơn nhiều; đề giới hạn kênh A ở mức ghi nhận | Chuyển giao/Kế toán nhập thay, nguồn ghi "đối tác báo" (BR-02.19) | Giai đoạn sau, chưa xếp |
| 2 | Nhập số liệu đối tác từ tệp | Chưa biết định dạng số liệu các đối tác | Kế toán nhập tay | 2 |
| 3 | Kiểm tra trùng lịch, lệnh công tác, xe, tạm ứng, nghỉ bù (FR-08) | Không phải khó khăn bắt buộc | Chỉ ghi điều động tối thiểu (`DIEU_DONG`) để dùng sau | 3 |
| 4 | Xét thưởng (FR-09), hành chính (FR-14) | Won't đợt này theo WSJF | Không làm | 3 |
| 5 | Báo cáo điều hành đầy đủ (FR-13) | Hướng đi | Phễu doanh thu trên bàn PGĐ Tài chính và báo cáo tuân thủ mốc | 3 |
| 6 | Hợp đồng định kỳ, gia hạn, nhắc ký bảo trì (FR-07) | Khó khăn không bắt buộc | Kinh doanh theo dõi ngoài hệ thống | 2 |
| 7 | Giao việc nội bộ (FR-11), tri thức (FR-12), thiết bị LIS (FR-10) | Không bắt buộc | Dùng đầu việc tối thiểu của FR-05 | 2 |
| 8 | Phát hiện yêu cầu trùng tự động | Chưa rõ nhu cầu | CSKH liên kết hoặc gộp tay (AC-03.1.6) | Chờ KH |
| 9 | Đẩy thông báo lên điện thoại, tin nhắn | Cần quyết kênh thông báo | Thông báo trong ứng dụng + email chỉ có tiêu đề, hạn, đường dẫn | Chờ KH |
| 10 | Nạp dự án đã đóng, dữ liệu lịch sử (G26) | Dữ liệu nguồn chưa rõ ở đâu | Chỉ nạp dự án đang mở; trạng thái không rõ ghi "Cần xác minh" | Theo yêu cầu |
| 11 | Luồng khẩn cấp ngoài cổng khi bệnh viện ngừng vận hành (TH-C03) | Là quyết định của Ban giám đốc, không phải của BA | Giữ nguyên cổng, mức "gấp" chỉ rút ngắn bằng cảnh báo ưu tiên | Chờ BGĐ |
| 12 | Khóa đồng thời, RPO/RTO, tiền giữ lại bảo hành, ngoại tệ, thuế suất khác nhau | Đề không nêu | Ghi `[CẦN LÀM RÕ VỚI KH]` | Chờ KH |
| 13 | Thay phần mềm kế toán pháp định | Ngoài ranh giới (R9) | Kế toán xuất hóa đơn ở phần mềm kế toán rồi ghi số vào hệ thống | Không làm |
| 14 | Phạt tự động khi quá SLA | Ngoài ranh giới (R8) | Chỉ nhắc, leo thang, báo cáo | Không làm |

## B.1.5 Kịch bản đi hết đường (tiêu chí "MVP đủ dùng")

Bốn kịch bản demo; khi cả bốn chạy trơn trên dữ liệu thật thì MVP đạt. Đây cũng là bốn luồng Ban giám đốc dễ yêu cầu chỉ trên sơ đồ khi bảo vệ.

| # | Kịch bản | Đi qua story | Tình huống phát sinh có thể hỏi |
|---|---|---|---|
| E2E-1 | Kênh B: tạo dự án → Vận hành → BBNT → nộp 5 ngày → hợp đồng 10 ngày → hóa đơn → gửi hồ sơ → công nợ → thu đủ | US-01, 02, 03, 04, 05 | Bệnh viện trì hoãn nghiệm thu; hợp đồng đã ký trước (TH-B01, B04) |
| E2E-2 | Kênh A: tạo dự án → BBNT đối tác → đổi tỷ lệ giữa kỳ → kỳ đối soát hai tỷ lệ → chốt BBĐS → hóa đơn cho đối tác | US-06, 07, 09, 08 | Đối tác Y đổi tỷ lệ giữa kỳ; hồi tố vào kỳ đã chốt (TH-A01, A03) |
| E2E-3 | Yêu cầu vừa rủi ro vừa tính phí: IT BV duyệt → lãnh đạo BV xác nhận ∥ PGĐ Chuyên môn duyệt, báo giá → Phần mềm xử lý → đóng | US-10, 11, 12, 13 | Bệnh viện yêu cầu xóa dữ liệu gấp (TH-C01) |
| E2E-4 | Rút đội: danh sách tồn có việc từ xa → CSKH trả lại → bổ sung → nhận → đầu việc tồn → xác nhận đóng | US-14, 15 | Rút gấp chưa kịp lập danh sách (TH-D02) |

## B.1.6 Ba điểm cần PO quyết trước khi chốt MVP

Đây là chỗ đối chiếu với `Tinh-huong-phat-sinh-va-edge-case.md` (mục 7, nhóm "nên có trước khi bảo vệ") cho thấy **Feature Spec chưa phủ**, nên story map chưa thể coi là kín:

| # | Khoảng trống | Ảnh hưởng tới story | Đề xuất |
|---|---|---|---|
| 1 | **Rút gấp** chưa kịp lập danh sách tồn (TH-D02). BR-04.1 chặn tuyệt đối, có thể làm kẹt thực tế | US-14 A3 | Thêm ngoại lệ do Trưởng phòng Chuyển giao duyệt, danh sách bổ sung trong hạn (tham số). Nếu không thêm, nói rõ khi bảo vệ là cố ý chặn cứng để bảo vệ KK05 |
| 2 | **Ủy quyền duyệt có thời hạn** khi PGĐ Tài chính vắng (TH-E02) chưa có trong FR-02/FR-06 | US-08, US-09 | Thêm vào FR-06.2 hoặc ghi "để sau" ở B.1.4 |
| 3 | **Chấm dứt dự án** sau khi đã triển khai (TH-B11): BR-04.23 chỉ cho Hủy ở "Mới tạo"/"Đã điều động" | US-01, US-14 | Thêm trạng thái Chấm dứt (Giám đốc duyệt) hoặc đưa vào "để sau" |

---

# B.2 — User story có tiêu chí nghiệm thu Given / When / Then

## B.2.1 Quy ước

- **15 story**, đúng giới hạn 10–15 của đề; mỗi khó khăn bắt buộc có ≥ 2 story (bảng B.2.2).
- Mỗi story: câu "Là… tôi muốn… để…", khó khăn nguồn, FR, Rx, lát phát hành, giả định liên quan, rồi các kịch bản **A1…An** gồm *Happy* (đường chính), *Exception* (bị chặn hoặc lỗi), *Edge* (biên, trường hợp đặc biệt).
- Cột **Truy vết** ghi `AC-XX.k.n` đã có ở Feature Spec. Dòng gắn `[BA đề xuất]` là hành vi chưa nằm trong Backbone v1.1 (kèm mã tình huống `TH-..`), chờ xác nhận; nếu Công ty A không đồng ý thì bỏ dòng đó mà không ảnh hưởng các kịch bản còn lại.
- Số liệu trong ví dụ lấy từ chính các AC của Feature Spec để Given/When/Then kiểm thử được ngay.
- Các hạn (5, 10, 15 ngày…) là tham số (NFR-G7.1); giá trị nêu trong kịch bản là **mặc định**.

## B.2.2 Bảng tổng hợp 15 story

| Mã | Tên ngắn | KK | Vai trò | FR | Rx | Lát | Số kịch bản | Giả định |
|---|---|---|---|---|---|---|---|---|
| US-01 | Xem chuỗi dự án, biết đang kẹt ở ai | KK01 | Trưởng phòng/PGĐ | FR-01.1, FR-05.4, FR-06.5 | R25 | 1 | 4 | G09, G10, G26 |
| US-02 | Ghi 4 mốc và BBNT trên một dự án | KK01 | Trưởng nhóm triển khai | FR-01.1, FR-01.2 | R19, R25 | 1 | 7 | G09, G10, G21 |
| US-03 | Hoàn tất hợp đồng sau khi nhận BBNT | KK01 | Nhân viên Kinh doanh | FR-01.2 | R20, R25 | 1 | 5 | **G01** |
| US-04 | Theo dõi hồ sơ, hóa đơn đã gửi | KK01 | Nhân viên Kinh doanh | FR-01.3 | R21 | 1 | 3 | G11 |
| US-05 | Ghi hóa đơn, đòi nợ, thu tiền | KK01 | Kế toán | FR-01.3, FR-01.4 | R9, R22 | 1 | 7 | G11 |
| US-06 | Ghi nhận dự án kênh A đối tác báo | KK02 | Nhân viên Chuyển giao | FR-02.1 | R10, R16 | 3 | 5 | G24 |
| US-07 | Theo dõi BBNT đối tác chưa gửi | KK02 | Nhân viên Chuyển giao | FR-02.1, FR-02.3 | R26 | 3 | 4 | G02 |
| US-08 | Lập kỳ đối soát, ra BBĐS | KK02 | PGĐ Tài chính | FR-02.2, FR-02.3 | R26 | 3 | 6 | **G03, G04**, G23, G25 |
| US-09 | Cấu hình tỷ lệ, chu kỳ, ngoại lệ | KK02 | Kế toán, PGĐ Tài chính | FR-02.4, FR-06.3 | R5, R26 | 3 | 7 | **G04**, G22 |
| US-10 | Gửi yêu cầu trên cổng, theo dõi trạng thái | KK04 | Người dùng bệnh viện | FR-03.1 | R12, R28 | 4 | 5 | G05, G08 |
| US-11 | IT bệnh viện duyệt hoặc từ chối | KK04 | IT bệnh viện | FR-03.2 | R13 | 4 | 4 | G05 |
| US-12 | Đánh giá rủi ro, lãnh đạo BV xác nhận | KK04 | CSKH, Phần mềm | FR-03.3 | R14, R28 | 4 | 7 | G05, G06, G19 |
| US-13 | Duyệt tính phí, báo giá, miễn/giảm | KK04 | PGĐ Chuyên môn | FR-03.4 | R11, R28 | 4 | 6 | G07, G08, G19 |
| US-14 | Lập danh sách việc tồn khi chuẩn bị rút | KK05 | Trưởng nhóm triển khai | FR-04.1, FR-04.3 | R29 | 2 | 7 | G09, G12, G21 |
| US-15 | Xác nhận nhận bàn giao hoặc trả lại | KK05 | CSKH | FR-04.2, FR-04.3 | R29 | 2 | 7 | G12 |

Phân bổ: KK01 = 5 story · KK02 = 4 · KK04 = 4 · KK05 = 2 → tổng 15.

---

## B.2.3 Các user story

### KK01 — Chuỗi Triển khai → BBNT → Hợp đồng → Hóa đơn → Thu tiền

#### US-01 — Xem chuỗi dự án và biết đang kẹt ở ai

**Là** Trưởng phòng hoặc PGĐ, **tôi muốn** xem một dự án đang ở bước nào của chuỗi triển khai → BBNT → hợp đồng → hóa đơn → thu tiền và đang kẹt ở ai, **để** nhắc đúng người thay vì hỏi từng phòng.

*KK01 · FR-01.1 (UC-01.4), FR-05.4, FR-06.5 · R25 · Lát 1 · Must*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Dự án kênh B ở trục doanh thu "Chờ hợp đồng", người phụ trách là Kinh doanh K, đã ở bước này 6 ngày | Trưởng phòng Kinh doanh mở chi tiết dự án hoặc bàn làm việc | Thấy hai trục trạng thái, các mốc đã ghi, người phụ trách bước hiện tại là K và "6 ngày ở bước Chờ hợp đồng" | AC-01.1.6, AC-05.4.3 |
| A2 | Edge | Dự án thuộc kênh A | Mở chi tiết dự án | Trục doanh thu rẽ nhánh "Chờ đối soát"; không hiển thị bước hợp đồng và hóa đơn của kênh B | AC-01.1.7 |
| A3 | Exception | Người dùng chỉ có quyền trong phạm vi phòng mình | Tìm hoặc mở trực tiếp (đường dẫn) một dự án ngoài phạm vi | Hệ thống không hiển thị và từ chối cả khi gọi trực tiếp, không trả dữ liệu | AC-01.1.8, AC-06.2.2 |
| A4 | Edge | Dự án đang mở được nạp từ dữ liệu cũ mà người nhập không xác định được đã ký hợp đồng hay chưa | PGĐ Tài chính mở phễu doanh thu | Trục doanh thu ghi "Cần xác minh" (không đoán); dự án hiển thị riêng, loại khỏi tỷ lệ tuân thủ và có dòng "N dự án cần xác minh" | AC-06.5.4, AC-05.4.6 |

#### US-02 — Ghi 4 mốc và BBNT trên cùng một dự án

**Là** Trưởng nhóm triển khai, **tôi muốn** ghi bốn mốc ngày và BBNT trên cùng một dự án, **để** mọi phòng cùng thấy một dữ liệu và hạn 5 ngày nộp BBNT được tính từ đúng ngày ký.

*KK01 · FR-01.1, FR-01.2 · R19, R25 · Lát 1 · Must · Bốn mốc: Bắt đầu, Vận hành, Nghiệm thu (= ngày ký BBNT), Rút (G09)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Dự án trục triển khai "Đang triển khai" | Trưởng nhóm ghi mốc Vận hành | Trục doanh thu chuyển "Chưa phát sinh" → "Chờ nghiệm thu", người phụ trách là Trưởng nhóm, số ngày tồn bắt đầu đếm | AC-01.1.3 |
| A2 | Happy | Trục doanh thu "Chờ nghiệm thu" | Trưởng nhóm ghi BBNT kèm ngày ký 03/12 và tệp | Trục doanh thu "Đã nghiệm thu"; mốc Nghiệm thu = 03/12; hệ thống tạo việc "Nộp BBNT cho Kinh doanh" hạn 5 ngày làm việc kể từ ngày ký (10/12) | AC-01.2.1, AC-05.5.1 |
| A3 | Exception | BBNT chưa có ngày ký (hoặc chưa có tệp) | Lưu | Hệ thống chặn và yêu cầu nhập trường còn thiếu; không tạo bản ghi BBNT | AC-01.2.2 |
| A4 | Exception | Đã quá 5 ngày làm việc kể từ ngày ký mà BBNT chưa nộp cho Kinh doanh | Đến hạn | Hệ thống nhắc nhân viên Chuyển giao phụ trách, báo Trưởng phòng Chuyển giao và ghi vết; không phạt tự động | AC-01.2.3 |
| A5 | Exception | Dự án đang "Tạm dừng" | Trưởng nhóm ghi mốc Vận hành | Hệ thống không cho ghi và nêu lý do | AC-01.1.4 |
| A6 | Edge | Dự án đổi trưởng nhóm giữa chừng | Trưởng phòng Chuyển giao đổi người | Hệ thống ghi sự kiện (thời điểm, người cũ, người mới); trạng thái không đổi; ngày làm việc ghi theo từng người; trưởng nhóm cũ không còn quyền ghi mốc nhưng xem được lịch sử phần mình | AC-01.1.5, AC-06.2.7 |
| A7 | Edge | Một mốc đã ghi | Người có quyền sửa ngày mốc | Hệ thống lưu giá trị trước/sau, người sửa, thời điểm (đúng 1 bản ghi vết) và tính lại số ngày tồn | AC-01.1.9, AC-06.4.6 |

#### US-03 — Hoàn tất hợp đồng sau khi nhận BBNT

**Là** Nhân viên Kinh doanh, **tôi muốn** được giao việc hoàn tất hợp đồng ngay khi nhận BBNT và được nhắc theo hạn, **để** không để quá 10 ngày mà không biết.

*KK01 · FR-01.2 · R20, R25 · Lát 1 · Must · Phụ thuộc G01 (hợp đồng ký **sau** BBNT) — nếu sai, đảo thứ tự "Chờ hợp đồng" ở A.4b*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | BBNT của dự án kênh B không đấu thầu đã được nộp | Kinh doanh xác nhận đã nhận | Trục doanh thu chuyển "Chờ hợp đồng"; hệ thống tạo việc "Hoàn tất hợp đồng" cho nhân viên phụ trách, hạn 10 ngày kể từ ngày nhận, hiện trong "Việc của tôi" | AC-01.2.4, AC-05.1.1 |
| A2 | Exception | Việc hoàn tất hợp đồng đã quá 10 ngày | Đến hạn | Hệ thống nhắc nhân viên, báo Trưởng phòng Kinh doanh, ghi vết; không phạt tự động | AC-01.2.5, AC-05.2.1 |
| A3 | Edge | Dự án đấu thầu (hợp đồng có trước) | BBNT được nhận | Hệ thống không tạo việc hoàn tất hợp đồng, chuyển thẳng "Chờ xuất hóa đơn" | AC-01.2.6 |
| A4 | Edge | Dự án **không** đấu thầu nhưng hợp đồng đã ký trước đó | Kinh doanh nhận BBNT | Hệ thống bỏ qua "Chờ hợp đồng", chuyển "Chờ xuất hóa đơn" `[BA đề xuất — TH-B04]` | AC-01.2.7 |
| A5 | Edge | Hạn 10 ngày rơi vào ngày nghỉ | Hệ thống tính hạn | Áp quy ước ngày lịch hoặc ngày làm việc theo tham số `[CẦN LÀM RÕ VỚI KH]` | AC-01.2.9, AC-05.2.6 |

#### US-04 — Theo dõi hồ sơ và hóa đơn đã gửi khách

**Là** Nhân viên Kinh doanh, **tôi muốn** theo dõi hồ sơ và hóa đơn đã gửi (gọi xác nhận sau 1–2 ngày, hợp đồng gốc sau 2 tuần), **để** không bỏ sót.

*KK01 · FR-01.3 · R21 · Lát 1 · Must*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Hóa đơn đã được ghi | Kinh doanh ghi "Đã gửi hồ sơ" | Hệ thống tạo việc gọi xác nhận trong 1–2 ngày; khi ghi khách đã nhận, trục doanh thu chuyển "Đang công nợ" | AC-01.3.3 |
| A2 | Exception | Sau 2 tuần chưa nhận lại hợp đồng gốc | Đến hạn | Hệ thống báo Trưởng phòng Kinh doanh; cờ "hợp đồng gốc đã về" chưa đánh dấu và **không chặn** công nợ | AC-01.3.4 |
| A3 | Exception | Quá hạn gọi xác nhận mà chưa ghi kết quả | Đến hạn | Hệ thống nhắc nhân viên Kinh doanh phụ trách | AC-01.3.5 |

#### US-05 — Ghi hóa đơn, đòi nợ và thu tiền

**Là** Kế toán, **tôi muốn** nhận đề nghị xuất hóa đơn đủ điều kiện, ghi số hóa đơn, và được nhắc đòi nợ 15 ngày/lần, báo Trưởng phòng sau 3 lần, **để** công nợ không treo.

*KK01 · FR-01.3, FR-01.4 · R9, R22 · Lát 1 · Must · Hệ thống không phát hành hóa đơn pháp định (R9)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Dự án kênh B đã có BBNT và hợp đồng đã ký (hoặc đấu thầu) | Kế toán nhận đề nghị và ghi số hóa đơn, ngày, giá trị | Trục doanh thu chuyển "Đã xuất hóa đơn"; hệ thống không phát hành hóa đơn pháp định | AC-01.3.1 |
| A2 | Exception | Dự án chưa có BBNT | Kinh doanh gửi đề nghị xuất hóa đơn | Hệ thống từ chối và nêu điều kiện còn thiếu | AC-01.3.2 |
| A3 | Happy | Hóa đơn ở "Đang công nợ" | Đến nhịp 15 ngày | Hệ thống tạo việc đòi nợ cho Kế toán | AC-01.4.1 |
| A4 | Exception | Đã đòi nợ 3 lần mà chưa thu | Đến nhịp kế tiếp | Hệ thống báo Trưởng phòng Kế toán và ghi vết; vẫn tạo việc đòi nợ | AC-01.4.2, AC-05.2.2 |
| A5 | Happy | Dự án có một hóa đơn | Kế toán ghi thu đủ | Trục doanh thu chuyển "Đã thu" | AC-01.4.3 |
| A6 | Edge | Dự án có hai hóa đơn theo đợt, mới thu đủ một | Xem trạng thái | Trục doanh thu vẫn "Đang công nợ" đến khi thu đủ cả hai | AC-01.4.4 |
| A7 | Edge | Một hóa đơn đã ghi bị hủy hoặc điều chỉnh | Kế toán ghi hóa đơn thay thế | Hóa đơn cũ ở trạng thái "đã hủy/đã điều chỉnh", liên kết hóa đơn mới; công nợ tính theo hóa đơn hiệu lực `[BA đề xuất — TH-B06]` | AC-01.3.6 |

---

### KK02 — Nhiều đối tác bán hàng (kênh A) và đối soát

#### US-06 — Ghi nhận dự án kênh A do đối tác báo

**Là** nhân viên Chuyển giao, **tôi muốn** ghi nhận dự án kênh A do đầu mối đối tác báo kèm giấy tờ điều động, **để** dự án kênh A không nằm ngoài hệ thống.

*KK02 · FR-02.1 · R10, R16 · Lát 3 · Must · Kênh A không có hợp đồng hay báo giá với từng bệnh viện (R10)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Đầu mối đối tác X báo một dự án cho Chuyển giao | Nhân viên Chuyển giao tạo Dự án kênh A với đối tác X, sản phẩm và giấy tờ điều động | Dự án ở "Mới tạo"; hệ thống không yêu cầu hợp đồng hay báo giá | AC-02.1.1 |
| A2 | Exception | Chọn kênh A nhưng chưa chọn đối tác | Lưu | Hệ thống chặn | AC-02.1.2 |
| A3 | Edge | Bệnh viện M đã có dự án LIS kênh B | Tạo dự án HIS kênh A với đối tác X | Hai dự án cùng thuộc khách hàng M; báo cáo theo M thấy cả hai, đối soát của X chỉ thấy dự án HIS | AC-02.1.3, AC-06.1.1 |
| A4 | Exception | Hợp đồng khung của đối tác đã hết hiệu lực | Chuyển giao tạo dự án mới với đối tác đó | Hệ thống chặn; dự án đã tạo trước đó vẫn đối soát bình thường `[BA đề xuất — TH-A16]` | AC-02.1.7, AC-06.1.5 |
| A5 | Edge | Đối tác đề nghị tự nhập BBNT trên hệ thống | Nhân viên Chuyển giao ghi thay | Bản ghi có nguồn "đối tác báo" và người nhập; việc đối tác đăng nhập thuộc giai đoạn sau (G24) | AC-02.1.8 |

#### US-07 — Theo dõi BBNT đối tác chưa gửi

**Là** nhân viên Chuyển giao, **tôi muốn** theo dõi BBNT đối tác chưa gửi và được nhắc, **để** việc kẹt không bị bỏ quên.

*KK02 · FR-02.1, FR-02.3 · R26 · Lát 3 · Must · Hạn gửi BBNT là tham số theo đối tác, chưa có số (G02)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Exception | Dự án kênh A ở "Chờ nghiệm thu"; đối tác có hạn gửi BBNT đã cấu hình | Quá hạn | Hệ thống nhắc nhân viên phụ trách và Trưởng phòng Chuyển giao, hiện trong "Việc của tôi" | AC-02.1.4 |
| A2 | Happy | Đối tác gửi BBNT | Nhân viên Chuyển giao ghi BBNT | Dự án chuyển "Đã nghiệm thu" rồi "Chờ đối soát" và thuộc kỳ theo ngày BBNT | AC-02.1.5 |
| A3 | Edge | Đối tác chưa có hạn gửi BBNT được cấu hình | Dự án ở "Chờ nghiệm thu" | Hệ thống không tính quá hạn, hiển thị cảnh báo "thiếu tham số" cho người quản trị tham số | AC-02.1.6, AC-06.3.5 |
| A4 | Edge | Đối tác gửi BBNT trễ khi kỳ chứa ngày BBNT đã chốt | Kế toán ghi BBNT | Hệ thống tạo dòng điều chỉnh vào kỳ kế tiếp, không sửa kỳ đã chốt (G23) | AC-02.3.6 |

#### US-09 — Cấu hình tỷ lệ chia sẻ, chu kỳ đối soát và ngoại lệ

**Là** người quản trị tham số (Kế toán đề xuất, PGĐ Tài chính duyệt), **tôi muốn** cấu hình tỷ lệ chia sẻ và chu kỳ đối soát theo đối tác, có hiệu lực theo ngày và ngoại lệ có căn cứ, **để** không phụ thuộc một người.

*KK02 · FR-02.4, FR-06.3 · R5, R26 · Lát 3 (làm trước US-08) · Must · Người đề xuất ≠ người duyệt; IT nội bộ không sửa giá trị nghiệp vụ (G22)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Kế toán đề xuất tỷ lệ mới cho đối tác Y × sản phẩm HIS, hiệu lực từ ngày N, kèm căn cứ | PGĐ Tài chính duyệt | Hệ thống tạo bản ghi mới; bản cũ giữ nguyên và hết hạn ngày N−1; tỷ lệ mới áp cho dự án có BBNT từ ngày N | AC-02.4.1, AC-06.3.1 |
| A2 | Exception | Người đề xuất cũng là người duyệt | Bấm duyệt | Hệ thống chặn (kiểm tra ở máy chủ) | AC-02.4.2, AC-06.2.3 |
| A3 | Exception | Ngoại lệ cho một dự án không đính căn cứ | Gửi duyệt | Hệ thống chặn | AC-02.4.3 |
| A4 | Edge | Người dùng là IT nội bộ | Mở màn tham số nghiệp vụ | Chỉ thấy khung tham số, không có quyền sửa giá trị | AC-02.4.4, AC-06.2.4 |
| A5 | Edge | Tỷ lệ mới hiệu lực từ ngày N; một dự án có BBNT đúng ngày N | Hệ thống tìm tỷ lệ | Dự án dùng tỷ lệ mới | AC-02.4.5 |
| A6 | Edge | Ngoại lệ có giá trị vượt ngưỡng cấu hình | PGĐ Tài chính duyệt | Hệ thống chuyển Giám đốc duyệt trước khi có hiệu lực (ngưỡng `[CẦN LÀM RÕ VỚI KH]`) | AC-02.4.10 |
| A7 | Edge | Đối tác đổi chu kỳ từ tháng sang quý | PGĐ Tài chính duyệt | Chu kỳ mới có hiệu lực từ kỳ kế tiếp; kỳ đang chạy giữ chu kỳ cũ | AC-02.4.9 |

#### US-08 — Lập kỳ đối soát và ra BBĐS

**Là** PGĐ Tài chính, **tôi muốn** lập kỳ đối soát theo đối tác, đối chiếu danh sách dự án với số liệu đối tác và ra BBĐS, **để** đối soát kiểm chứng được và không phụ thuộc một người.

*KK02 · FR-02.2, FR-02.3 · R26 · Lát 3 · Must · Rủi ro cao nhất nếu sai: G03 (cơ sở tính, chiều dòng tiền), G04 (khóa tỷ lệ, ngày áp)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Đối tác Y có chu kỳ quý và có dự án kênh A "Chờ đối soát" trong quý; số liệu đối tác đã được nhập | Kỳ kết thúc | Hệ thống lập Kỳ đối soát liệt kê dự án, tính phần chia theo tỷ lệ hiệu lực tại **ngày BBNT**, đánh dấu từng dòng khớp hoặc lệch | AC-02.2.1 |
| A2 | Exception | Kỳ còn dòng lệch (thiếu bên hệ thống, thiếu bên đối tác, lệch giá trị) | PGĐ Tài chính chốt BBĐS | Chỉ chốt được khi mọi dòng lệch đã được giải thích hoặc điều chỉnh có căn cứ; mặc định không dung sai | AC-02.3.2, AC-02.2.4 |
| A3 | Exception | Thiếu tỷ lệ cho đối tác × sản phẩm tại ngày BBNT và đối tác không có tỷ lệ mặc định | Hệ thống tính | Không tự đoán; chặn chốt kỳ và chỉ rõ tỷ lệ còn thiếu | AC-02.2.2 |
| A4 | Edge | BBĐS đã chốt | Cần sửa số liệu | Hệ thống không cho sửa trực tiếp, chỉ cho tạo điều chỉnh có căn cứ | AC-02.3.3 |
| A5 | Edge | Tỷ lệ đổi từ 30% sang 35% hiệu lực 15/08; dự án P1 BBNT 10/08 giá trị 100, P2 BBNT 20/08 giá trị 200 | Hệ thống tính | P1 dùng 30% (phần chia 30), P2 dùng 35% (phần chia 70); kỳ có hai tỷ lệ, tổng phần chia 100 | AC-02.2.5 |
| A6 | Happy | BBĐS đã chốt | Kế toán ghi số hóa đơn, ngày, giá trị cho đối tác | Dự án chuyển "Đã xuất hóa đơn cho đối tác". **Ngược lại**, nếu BBĐS chưa chốt thì hệ thống từ chối và nêu "chưa chốt BBĐS" | AC-02.3.4, AC-02.3.5 |

---

### KK04 — Yêu cầu khách hàng qua cổng (nhánh rủi ro, nhánh tính phí)

#### US-10 — Gửi yêu cầu trên cổng và theo dõi trạng thái

**Là** người dùng bệnh viện, **tôi muốn** gửi yêu cầu trên cổng và theo dõi trạng thái, **để** yêu cầu không thất lạc.

*KK04 · FR-03.1, FR-06.2 · R12, R28 · Lát 4 · Must · Cổng phục vụ cả kênh A và B (G08)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Người dùng bệnh viện đã đăng nhập cổng | Nhập nội dung, đính kèm và gửi | Yêu cầu ở "Chờ IT bệnh viện duyệt" với nguồn "cổng"; người gửi thấy trạng thái | AC-03.1.1 |
| A2 | Exception | Thiếu nội dung bắt buộc | Gửi | Hệ thống báo lỗi và không tạo yêu cầu | AC-03.1.2 |
| A3 | Edge | Yêu cầu chưa được IT bệnh viện duyệt | CSKH Công ty A tìm trong hàng đợi xử lý | Không thấy yêu cầu đó | AC-03.1.3 |
| A4 | Happy | Bệnh viện báo yêu cầu qua điện thoại | CSKH nhập hộ với nguồn "điện thoại" | Yêu cầu có nguồn và người nhập, vào "Chờ IT bệnh viện duyệt" (G05) | AC-03.1.4 |
| A5 | Exception | Tài khoản của bệnh viện A | Sửa đường dẫn để mở yêu cầu hoặc tệp của bệnh viện B | Truy cập thất bại, không lộ sự tồn tại của dữ liệu (0 dữ liệu bệnh viện khác ở màn hình, tìm kiếm, thông báo) | AC-06.2.6, AC-05.3.5 (NFR-03.1) |

#### US-11 — IT bệnh viện duyệt hoặc từ chối

**Là** IT bệnh viện, **tôi muốn** duyệt hoặc từ chối yêu cầu trước khi sang Công ty A, **để** chỉ yêu cầu đã duyệt mới được xử lý.

*KK04 · FR-03.2 · R13 · Lát 4 · Must*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Yêu cầu "Chờ IT bệnh viện duyệt" | IT bệnh viện duyệt | Yêu cầu sang "Chờ đánh giá" và CSKH thấy | AC-03.2.1 |
| A2 | Exception | IT bệnh viện chọn từ chối | Không nhập lý do | Hệ thống chặn; có lý do thì đóng yêu cầu và thông báo người gửi | AC-03.2.2 |
| A3 | Edge | Yêu cầu qua điện thoại do CSKH nhập hộ | Chưa có xác nhận của IT bệnh viện | Yêu cầu không được chuyển sang xử lý | AC-03.2.3 |
| A4 | Exception | IT bệnh viện không phản hồi quá hạn tham số | Đến hạn | Hệ thống nhắc IT bệnh viện và hiển thị tình trạng cho CSKH; không tự đóng khi chưa có quy tắc `[CẦN LÀM RÕ VỚI KH]` | AC-03.2.4 |

#### US-12 — Đánh giá rủi ro và lãnh đạo bệnh viện xác nhận

**Là** CSKH hoặc Phần mềm, **tôi muốn** đánh giá rủi ro yêu cầu và gửi lãnh đạo bệnh viện xác nhận trước khi xử lý, **để** tránh tranh cãi "ai đã đồng ý".

*KK04 · FR-03.3 · R14, R28 · Lát 4 · Must · "Gấp" không bỏ cổng, chỉ rút ngắn bằng cảnh báo ưu tiên (BR-03.9)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Yêu cầu "xóa dữ liệu" đã qua IT bệnh viện | CSKH phân loại rủi ro và Phần mềm xác nhận | Yêu cầu sang "Chờ lãnh đạo bệnh viện xác nhận" và **không** được giao Phần mềm | AC-03.3.1 |
| A2 | Happy | Yêu cầu đang chờ lãnh đạo | Lãnh đạo bệnh viện xác nhận | Hệ thống lưu người, thời điểm, nội dung xác nhận; nếu mọi cổng khác đã xong thì sang "Đã duyệt" | AC-03.3.2 |
| A3 | Exception | Yêu cầu đang chờ lãnh đạo | Lãnh đạo từ chối | Yêu cầu "Đóng - bị từ chối" và thông báo người gửi | AC-03.3.3 |
| A4 | Edge | Yêu cầu vừa rủi ro vừa có thể tính phí | Lãnh đạo xác nhận nhưng PGĐ Chuyên môn chưa duyệt tính phí | Yêu cầu chưa sang "Đã duyệt" (hai cổng độc lập, chạy song song, G19) | AC-03.3.4 |
| A5 | Exception | Bệnh viện nói qua điện thoại "cứ làm, sẽ gửi giấy sau" cho yêu cầu rủi ro | CSKH ghi nhận cuộc gọi | Hệ thống không giao Phần mềm cho đến khi có xác nhận của người có thẩm quyền ghi trên hệ thống | AC-03.3.7 |
| A6 | Edge | Yêu cầu phân loại phổ thông đang "Đang xử lý" nhưng thực chất rủi ro | Phần mềm thu hồi kèm lý do | Yêu cầu về "Chờ đánh giá", dừng xử lý và ghi vết `[BA đề xuất — TH-C08]` | AC-03.3.6 |
| A7 | Edge | Lãnh đạo bệnh viện đã nghỉ hoặc đổi người | Hệ thống chọn người xác nhận | Chỉ người trong danh mục có thẩm quyền của bệnh viện mới xác nhận được `[BA đề xuất — TH-C06]` | AC-03.3.5 |

#### US-13 — Duyệt yêu cầu tính phí, báo giá, miễn/giảm

**Là** PGĐ Chuyên môn, **tôi muốn** duyệt yêu cầu tính phí để Kinh doanh báo giá (miễn/giảm do Giám đốc), **để** không làm miễn phí việc lẽ ra thu tiền.

*KK04 · FR-03.4 · R11, R28 · Lát 4 · Must*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Yêu cầu được đánh dấu "có thể tính phí" | PGĐ Chuyên môn duyệt tính phí | Hệ thống tạo việc lập báo giá cho Kinh doanh; sau khi báo giá gửi và bệnh viện chấp nhận (có bằng chứng), cổng tính phí hoàn tất | AC-03.4.1 |
| A2 | Exception | Có đề xuất miễn hoặc giảm | PGĐ Chuyên môn chuyển Giám đốc | Việc chỉ thực hiện sau khi Giám đốc duyệt; chưa duyệt thì không giao Phần mềm | AC-03.4.2 |
| A3 | Exception | Yêu cầu chưa qua cổng tính phí | Có người cố giao Phần mềm làm | Hệ thống chặn và nêu cổng còn lại | AC-03.4.3, AC-03.5.2 |
| A4 | Edge | Báo giá quá hạn hiệu lực | Đến hạn | Yêu cầu "Đóng"; hủy sau khi đã chấp nhận thì Kinh doanh phải ghi lý do | AC-03.4.4 |
| A5 | Edge | Bệnh viện thuộc kênh A gửi yêu cầu có tính phí | PGĐ Chuyên môn duyệt tính phí | Hệ thống tạo thông báo cho đầu mối đối tác thay vì giao Kinh doanh báo giá trực tiếp (G08) | AC-03.4.5 |
| A6 | Edge | Yêu cầu loại "lỗi" của dự án đang trong bảo hành | Phân loại | Yêu cầu không vào cổng tính phí `[BA đề xuất — TH-C10]` | AC-03.4.6 |

---

### KK05 — Bàn giao Triển khai → CSKH

#### US-14 — Lập danh sách việc tồn khi chuẩn bị rút

**Là** Trưởng nhóm triển khai, **tôi muốn** lập danh sách việc tồn (cả việc từ xa) khi chuẩn bị rút, **để** bàn giao có danh sách.

*KK05 · FR-04.1, FR-04.3 · R29 · Lát 2 · Must · Danh sách được phép rỗng nhưng phải xác nhận "không còn tồn" (G12); Rút trước hay sau BBNT đều hợp lệ (G21)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Dự án "Đang triển khai"; Trưởng nhóm đã lập danh sách gồm 2 việc tồn đủ trường | Ghi mốc Rút | Trục triển khai chuyển "Đã rút - chờ bàn giao"; mốc Rút lưu người ghi và thời điểm; hệ thống tạo việc "Rà soát bàn giao" cho CSKH | AC-04.1.1 |
| A2 | Happy | Không còn việc tồn | Trưởng nhóm bấm "Không còn tồn" rồi ghi mốc Rút | Hệ thống chấp nhận, ghi xác nhận kèm người và thời điểm, vẫn giao việc rà soát cho CSKH | AC-04.1.2 |
| A3 | Exception | Chưa có việc tồn nào và chưa xác nhận "không còn tồn" | Ghi mốc Rút | Hệ thống chặn và nêu "thiếu danh sách việc tồn" (xem B.1.6 điểm 1 về rút gấp) | AC-04.1.3 |
| A4 | Exception | Một việc tồn thiếu hạn hoặc người phụ trách dự kiến | Lưu | Hệ thống chặn và nêu trường thiếu | AC-04.1.4 |
| A5 | Edge | Dự án có 2 việc từ xa còn mở | Trưởng nhóm mở "Chuẩn bị rút" | 2 việc đó đã nằm sẵn trong danh sách tồn và không có nút gỡ | AC-04.1.7 |
| A6 | Happy | Dự án "Đang triển khai" (hoặc đã bàn giao, còn trong bảo hành) | Tạo việc hình thức "từ xa" với người phụ trách và hạn | Việc hiện trong "Việc của tôi" của người phụ trách và xem được theo dự án; việc tồn phát hiện sau bàn giao mang nhãn "tồn từ triển khai" | AC-04.3.1 (mở rộng cho giai đoạn bảo hành, xem B.2.4) |
| A7 | Edge | Dự án chưa có BBNT (trục doanh thu "Chờ nghiệm thu") | Ghi mốc Rút | Trục triển khai chuyển bình thường; trục doanh thu giữ nguyên, hạn "Chờ nghiệm thu" vẫn chạy | AC-04.1.9 |

#### US-15 — Xác nhận nhận bàn giao hoặc trả lại kèm lý do

**Là** CSKH, **tôi muốn** xác nhận nhận bàn giao hoặc trả lại kèm lý do, **để** biết mình nhận những tồn đọng nào.

*KK05 · FR-04.2, FR-04.3 · R29 · Lát 2 · Must · Chỉ CSKH được xác nhận nhận; không có đường tắt cho Trưởng nhóm hay Trưởng phòng Chuyển giao (BR-04.8, BR-04.12)*

| # | Loại | Given | When | Then | Truy vết |
|---|---|---|---|---|---|
| A1 | Happy | Dự án "Đã rút - chờ bàn giao" với danh sách 3 việc tồn | CSKH xác nhận nhận | Dự án chuyển "Đã bàn giao - bảo hành"; 3 đầu việc có người phụ trách, hạn, hình thức hiện trong "Việc của tôi" | AC-04.2.1 |
| A2 | Happy | CSKH rà soát thấy danh sách chưa đủ | Trả lại kèm lý do "thiếu việc từ xa module X" | Dự án giữ "Đã rút - chờ bàn giao"; Trưởng nhóm nhận yêu cầu bổ sung; Trưởng phòng Chuyển giao được báo; lý do ghi vào lịch sử | AC-04.2.4, AC-04.2.9 |
| A3 | Exception | CSKH chọn trả lại | Không nhập lý do | Hệ thống chặn | AC-04.2.3 |
| A4 | Happy | Danh sách "không còn tồn" | CSKH xác nhận nhận | Dự án chuyển "Đã bàn giao - bảo hành"; không sinh đầu việc tồn nào | AC-04.2.2 |
| A5 | Exception | Người dùng là Trưởng nhóm đã lập danh sách | Bấm xác nhận nhận | Hệ thống chặn và nêu "chỉ CSKH được xác nhận nhận" | AC-04.2.6 |
| A6 | Exception | Quá hạn rà soát mà CSKH chưa xử lý | Đến hạn | Hệ thống nhắc CSKH phụ trách, báo Trưởng phòng CSKH, hiển thị cho Trưởng phòng Chuyển giao, ghi vết (leo thang Trưởng phòng CSKH là `[BA đề xuất]`; hạn rà soát `[CẦN LÀM RÕ VỚI KH]`) | AC-04.2.5, AC-05.2.5 |
| A7 | Edge | Đơn vị có 4 đầu việc tồn mở, 1 quá hạn | CSKH mở bàn làm việc | Hiển thị số tồn 4, số quá hạn 1 và tuổi của việc lâu nhất | AC-04.3.6 |

---

## B.2.4 Đồng bộ lại các file khác (danh sách sửa ô "Nguồn")

Sau khi chốt số thứ tự `Ak` ở trên, cần cập nhật cột **Nguồn** của các AC sau để ba tài liệu khớp nhau. Chỉ các ô khác với hiện tại được liệt kê; các AC không nêu ở đây giữ nguyên.

| File | AC | Nguồn hiện tại | Nguồn sau khi chỉnh |
|---|---|---|---|
| FR-01 | AC-01.2.1 | US-02 A1 / UC-01.1 | US-02 A2 / UC-01.1 |
| FR-01 | AC-01.2.2 | US-02 A2 / UC-01.1 E1 | US-02 A3 / UC-01.1 E1 |
| FR-01 | AC-01.2.3 | UC-01.2 E2 | US-02 A4 / UC-01.2 E2 |
| FR-01 | AC-01.1.4 | UC-01.1 E2 | US-02 A5 / UC-01.1 E2 |
| FR-01 | AC-01.1.5 | US-02 A3 / UC-01.1 | US-02 A6 / UC-01.1 |
| FR-01 | AC-01.1.9 | UC-01.1 | US-02 A7 / UC-01.1 |
| FR-01 | AC-01.2.7 | UC-01.2 | US-03 A4 / UC-01.2 |
| FR-01 | AC-01.2.9 | UC-01.2 | US-03 A5 / UC-01.2 |
| FR-01 | AC-01.4.2 | US-05 A3 / UC-01.3 E4 | US-05 A4 / UC-01.3 E4 |
| FR-01 | AC-01.4.1 | US-05 A3 / UC-01.3 | US-05 A3 / UC-01.3 (giữ) |
| FR-01 | AC-01.4.3, 01.4.4 | UC-01.3 | US-05 A5, US-05 A6 |
| FR-01 | AC-01.3.6 | UC-01.3 | US-05 A7 / UC-01.3 |
| FR-02 | AC-02.1.7, 02.1.8 | UC-02.1 E3 / UC-02.1 | US-06 A4, US-06 A5 |
| FR-02 | AC-02.3.6 | UC-02.3 | US-07 A4 / UC-02.3 |
| FR-02 | AC-02.2.5 | UC-02.2 | US-08 A5 / UC-02.2 |
| FR-02 | AC-02.3.4, 02.3.5 | UC-02.2 | US-08 A6 / UC-02.2 |
| FR-02 | AC-02.4.5, 02.4.9, 02.4.10 | UC-02.3 | US-09 A5, A7, A6 |
| FR-03 | AC-03.1.4 | UC-03.1 | US-10 A4 / UC-03.1 |
| FR-03 | AC-03.2.4 | UC-03.1 E3 | US-11 A4 / UC-03.1 E3 |
| FR-03 | AC-03.3.7, 03.3.6, 03.3.5 | UC-03.2 E4, E3, E2 | US-12 A5, A6, A7 |
| FR-03 | AC-03.4.4, 03.4.5, 03.4.6 | UC-03.3 | US-13 A4, A5, A6 |
| FR-04 | AC-04.1.1 / .2 / .3 / .4 / .7 / .9 | US-14 hoặc UC-04.1 | US-14 A1 / A2 / A3 / A4 / A5 / A7 |
| FR-04 | AC-04.3.1 | US-14 / UC-04.3 | US-14 A6 / UC-04.3; **thêm** điều kiện "đã bàn giao, còn trong bảo hành" vào Given hoặc tách AC mới (BR-04.15 đã cho phép) |
| FR-04 | AC-04.2.1 / .4 / .3 / .2 / .6 / .5 | US-15 hoặc UC-04.2 | US-15 A1 / A2 / A3 / A4 / A5 / A6 |
| FR-04 | AC-04.3.6 | UC-04.3 | US-15 A7 / UC-04.3 |
| Edge case | TH-A04 | US-09 A1 | US-09 A5 |
| Edge case | TH-D05 | US-14 A3 | US-14 A6 |
| Edge case | TH-D04 | US-15 A2 | US-15 A2 (giữ) |

**AC cố ý không gắn vào story** (giữ ở Feature Spec như enabler hoặc biên kỹ thuật, vẫn nằm trong phạm vi nghiệm thu MVP): AC-01.1.1, 01.1.2 (tạo dự án kênh B), AC-01.2.8 (nghiệm thu theo giai đoạn `[BA đề xuất]`), AC-01.3.7 (hóa đơn tạm ứng `[BA đề xuất]`), AC-01.4.5, 01.4.6 (thu thiếu/thừa, leo thang sau lần 3), AC-02.2.3, 02.2.6, 02.2.7, 02.3.1, 02.3.7, 02.3.8, 02.4.6–02.4.8, AC-03.1.5, 03.1.6, 03.4.x còn lại, AC-03.5.x, AC-04.1.6, 04.1.8, 04.1.10, AC-04.2.7–04.2.8, AC-04.3.2–04.3.5, 04.3.7, AC-04.4.x (điều động, tạm dừng, hủy, đóng), toàn bộ AC của FR-05 và FR-06 không được dẫn ở trên.

## B.2.5 Tự kiểm tra theo yêu cầu của đề

| Yêu cầu trong `Requirement.md` | Đáp ứng | Ghi chú |
|---|---|---|
| B.1 Story map cho phạm vi phát hành đầu tiên (MVP) | ✅ Mục B.1.2–B.1.3 | Xương sống theo hoạt động, cắt 5 lát (Lát 0–4) |
| B.1 "Nêu rõ cái gì cố ý để sau" | ✅ Mục B.1.4 (14 hạng mục, kèm cách MVP xử lý tạm và phát hành) | Ba điểm chưa phủ ở B.1.6 cần PO quyết |
| B.2 Viết đầy đủ 10–15 user story | ✅ 15 story | US-01…US-15 |
| B.2 "Cho các khó khăn bắt buộc" | ✅ KK01 ×5, KK02 ×4, KK04 ×4, KK05 ×2 | KK05 chỉ có 2 story nhưng gồm 14 kịch bản; không tách thêm để giữ tổng ≤ 15 |
| B.2 Mỗi story có tiêu chí nghiệm thu Given / When / Then | ✅ Tổng 84 kịch bản, mỗi kịch bản ghi `AC-XX.k.n` | Gồm cả Happy, Exception, Edge |
| Không viết AC hai lần | ✅ | Chỉ một bộ AC (ở Feature Spec); Phần B ánh xạ, đúng cam kết Backbone mục 2b |

**Các điểm còn phụ thuộc xác nhận từ Công ty A** (không làm đổi cấu trúc story, chỉ đổi giá trị trong Given/Then): G01 (US-03), G03 và G04 (US-08, US-09), G05 và G06 (US-10…12), G12 (US-14, US-15); toàn bộ dòng `[BA đề xuất]` và `[CẦN LÀM RÕ VỚI KH]` đã gắn nhãn tại chỗ.

## B.2.6 Change Request Log

| Version | Ngày | Người sửa | Nội dung | Ảnh hưởng | Trạng thái |
|---|---|---|---|---|---|
| v1.2 | 26/10/2026 | PO/BA | Viết mới B.1 (story map MVP) và B.2 (15 story Given/When/Then); chốt đánh số `Ak`; lập bảng đồng bộ ô "Nguồn" của AC | FR-01, FR-02, FR-03, FR-04 (ô Nguồn của AC); Backbone mục 2b; file edge case (TH-A04, TH-D05) | Draft |
