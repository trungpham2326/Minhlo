# SRS Backbone — ERP Quản trị nội bộ Công ty A (`ERP-CongTyA`)

> **Phiên bản:** v1.1 (Draft) · **Ngày:** 23/10/2026 · **Nguồn đầu vào:** `Requirement.md` (đề bài PO — Công ty A, phần mềm y tế) · **Người soạn:** PO/BA
> Tài liệu này viết **một lần, dùng chung** cho cả dự án. Chi tiết từng tính năng nằm ở các file Feature Spec riêng (Nhỏ / Lớn), liên kết ở mục 4.
> Đây là **nguồn sự thật duy nhất** về: bối cảnh, actor, giả định, NFR chung, backlog và trạng thái traceability toàn dự án.

---

## 0. Cách chọn loại Feature Spec (đọc trước khi tạo spec mới)

**A. Dùng template NHỎ** (`02-feature-spec-nho.md`) — nếu phần lớn các ý sau đúng:
- Effort ≤ 2 tuần.
- Chỉ có 1 luồng xử lý chính, không cần tách thành các phần độc lập.
- Không phụ thuộc / không bị ảnh hưởng bởi tính năng khác chạy song song.
- Input – Xử lý – Output gói trong vài dòng, ít trường hợp.
- Khách hàng hình dung được trong một lần review, không cần demo giữa chừng.

**B. Dùng template LỚN** (`03-feature-spec-lon.md`) — nếu có **bất kỳ** tín hiệu nào:
- Effort ≥ 3–4 tuần.
- Tách được thành ≥ 2 phần việc độc lập (sub-feature).
- Phụ thuộc hai chiều với tính năng khác.
- Cần NFR riêng (hiệu năng, bảo mật đặc thù).
- Rủi ro cao đi sai hướng nếu chỉ cho khách xem ở cuối → cần review giữa chừng.

**C. Mập mờ (2–3 tuần, không chắc):** tự hỏi *"Nếu làm sai, sửa lại có mất > 3–4 người-ngày không?"* — Có → **Lớn** (an toàn hơn, có mốc review để dừng sớm); Không → **Nhỏ**.

---

## 1. Giới thiệu

**Mục đích tài liệu:** xác lập khung yêu cầu của hệ thống ERP nội bộ Công ty A để (1) đội phân tích/thiết kế/phát triển/kiểm thử cùng bám một nguồn; (2) Ban giám đốc đối chiếu hệ thống với 12 khó khăn vận hành (KK01–KK12); (3) làm đầu vào cho các Feature Spec, mô hình, wireframe và test case.

**Phạm vi dự án**

*Hệ thống làm:*
- Theo dõi xuyên suốt chuỗi **Dự án → BBNT → Hợp đồng → Hóa đơn → Thu tiền** (kênh B) và **Dự án → BBNT → Đối soát → Hóa đơn cho đối tác → Thu tiền** (kênh A), có mốc, người phụ trách và cảnh báo quá hạn.
- Quản lý nhiều đối tác bán hàng: tham số chia sẻ doanh thu, kỳ đối soát, biên bản đối soát (BBĐS).
- Cổng khách hàng nhận và duyệt yêu cầu (nhánh rủi ro, nhánh tính phí).
- Bàn giao Triển khai → CSKH kèm danh sách việc tồn.
- Bàn làm việc theo vai trò, trang "Việc của tôi", phân quyền theo vai trò và phạm vi dữ liệu, lưu vết phê duyệt.
- Hướng đi cho các mảng còn lại (hợp đồng định kỳ, điều động/công tác, thưởng, thiết bị, giao việc, tri thức, báo cáo, hành chính).

*Hệ thống KHÔNG làm (ranh giới đã chốt trong đề):*
- Không thay phần mềm kế toán pháp định; chỉ quản lý hóa đơn và công nợ ở mức quản trị. (R9)
- Không quản lý thầu và hợp đồng nội bộ của đối tác bán hàng; kênh A chỉ ghi nhận dự án triển khai và giấy tờ để điều động người. (R10)
- Không phạt tự động khi quá SLA; SLA chỉ để cảnh báo và báo cáo. (R8)

**Thuật ngữ / viết tắt**

| Từ | Giải thích |
|---|---|
| BBNT | Biên bản nghiệm thu — căn cứ để làm hợp đồng và ghi nhận doanh thu |
| BBĐS | Biên bản đối soát doanh thu với một đối tác bán hàng |
| Kênh A / Kênh B | Doanh thu qua đối tác bán hàng (X, Y, Z…) / ký trực tiếp với khách |
| Hợp đồng khung | Hợp đồng hợp tác giữa Công ty A và một đối tác, quy định tỷ lệ chia sẻ và cách đối soát |
| Chuyển giao | Tên nội bộ của phòng Triển khai |
| Rút | Ngày đội triển khai rời đơn vị |
| Nghỉ bù | Ngày nghỉ cộng thêm cho nhân viên đi công tác dài ngày |
| SLA | Thời hạn cam kết xử lý một đầu việc hoặc yêu cầu |
| Dự án | Một lần triển khai **một sản phẩm** tại **một khách hàng**; thuộc đúng **một kênh** (đơn vị gắn kênh, đối tác, hợp đồng) |
| Căn cứ triển khai | Giấy tờ tối thiểu để Chuyển giao bắt đầu triển khai kênh B (báo giá được chấp thuận, đơn đặt hàng hoặc thỏa thuận đã chốt) — xem G01 |
| Kỳ đối soát | Khoảng thời gian (tháng/quý, tùy đối tác) gom các dự án kênh A đã BBNT để đối soát |
| Chờ nghiệm thu | Trạng thái dự án đã xong kỹ thuật nhưng chưa có BBNT — xem G10 |
| Tham số | Giá trị cấu hình được (có mặc định và ngoại lệ có căn cứ), không code cứng |

---

## 2. Mô tả tổng quan hệ thống

**Bối cảnh sản phẩm:** ứng dụng web độc lập, chạy trên máy tính và điện thoại, một cơ sở dữ liệu chung cho mọi phòng (R1). Không tích hợp hệ thống bên ngoài trong phạm vi này; số liệu hóa đơn do Kế toán ghi vào hệ thống (G11). Việc tích hợp với phần mềm kế toán pháp định (nếu có) → `[CẦN LÀM RÕ VỚI KH]`.

**Đối tượng người dùng (Actor)** — mô tả theo đầu việc trong đề; quyền chi tiết theo từng chức năng ghi ở mục "Phân quyền" của từng spec, ma trận tổng hợp thuộc skill 1.5.

| Nhóm | Vai trò | Mô tả quyền hạn (theo đề) |
|---|---|---|
| Ban giám đốc | Giám đốc | Duyệt mọi quyết định chính sách khách hàng (giảm giá, miễn phí, ưu đãi) (R11); duyệt ngoại lệ chia sẻ doanh thu vượt ngưỡng `[CẦN LÀM RÕ VỚI KH]` (G22) |
| | PGĐ Kinh doanh | Phụ trách mảng Kinh doanh; đầu việc chi tiết trong hệ thống `[CẦN LÀM RÕ VỚI KH]` |
| | PGĐ Chuyên môn | Duyệt yêu cầu chức năng mới và yêu cầu tính phí |
| | PGĐ Tài chính | Đối soát doanh thu định kỳ với từng đối tác; duyệt tỷ lệ chia sẻ, chu kỳ đối soát và ngoại lệ dưới ngưỡng, chốt BBĐS (G22, Phần B.4) |
| Phòng ban | Trưởng phòng (các phòng) | Nhận cảnh báo khi quá mốc (hợp đồng, hợp đồng gốc, nợ…) |
| | Nhân viên Kinh doanh | Báo giá, làm hợp đồng, gửi hồ sơ và hóa đơn, xin lại hợp đồng gốc, nhắc gia hạn |
| | CSKH | Nhận yêu cầu, phân loại, chuyển Phần mềm/Chuyển giao, theo dõi tồn đọng, nhận bàn giao sau triển khai |
| | Trưởng phòng Chuyển giao / Trưởng nhóm triển khai / Nhân viên triển khai | Nhận lịch, điều động, triển khai, thu BBNT, bàn giao CSKH, tính công |
| | Phần mềm, Test, Hạ tầng kỹ thuật | Xử lý yêu cầu, kiểm thử, hạ tầng và thiết bị |
| | Kế toán / Trưởng phòng Kế toán | Xuất hóa đơn, công nợ, đòi nợ định kỳ, duyệt tạm ứng/quyết toán, tính lương; đề xuất tham số chia sẻ doanh thu và ngoại lệ (G22) |
| | Hành chính – Nhân sự | Chấm công, phép, hồ sơ nhân sự, tài sản |
| | IT nội bộ (kiêm nhiệm) | Quản trị kỹ thuật (tài khoản, khung tham số); **không** sửa giá trị tham số nghiệp vụ (G22, Phần B.4) |
| Bên ngoài | Người dùng bệnh viện | Nhập yêu cầu trên cổng |
| | IT bệnh viện | Duyệt yêu cầu trước khi sang Công ty A |
| | Lãnh đạo bệnh viện | Xác nhận yêu cầu rủi ro |
| | Đầu mối đối tác bán hàng | Báo dự án kênh A cho Chuyển giao, gửi BBNT, gửi số liệu đối soát |
| Hệ thống | Bộ nhắc/cảnh báo | Tạo việc, gửi cảnh báo theo mốc và SLA |

**Quyết định thiết kế nền (BA đề xuất, dùng cho mọi FR):**
- **D1.** Kênh gắn ở cấp **Dự án** (không gắn ở Khách hàng) → một bệnh viện thuộc cả hai kênh được biểu diễn tự nhiên (một khách hàng — nhiều dự án, mỗi dự án một kênh).
- **D2.** Trạng thái Dự án tách **hai trục độc lập**: (a) *Triển khai & bàn giao*, (b) *Nghiệm thu & doanh thu*. Lý do: "đã bàn giao" và "chưa thu tiền" có thể đúng cùng lúc — gộp một trục là nguồn gốc của KK01.
- **D3.** Mỗi mốc có **một chủ** (người chịu trách nhiệm) và **một nguồn ghi ngày**; hệ thống nhắc người chủ và leo thang lên Trưởng phòng, không chờ người trễ tự báo.
- **D4.** Mọi con số nghiệp vụ (5/10/15 ngày, tỷ lệ chia sẻ, chu kỳ đối soát, số tháng mốc, định mức thưởng) là **tham số** có mặc định và ngoại lệ có căn cứ (R5).

**Giả định & ràng buộc**

*Ràng buộc đã chốt trong đề:* R1–R11 (mục 5.1).

*Giả định BA đề xuất (đã được PO đồng ý dùng làm giả định dự án; Công ty A chưa xác nhận — mỗi dòng kèm câu hỏi hỏi lại):*

| Mã | Giả định | Câu hỏi hỏi lại Công ty A |
|---|---|---|
| G01 | Dự án thông thường: hợp đồng chính thức hoàn tất **sau** BBNT; trước triển khai chỉ cần "căn cứ triển khai". Dự án đấu thầu có hợp đồng trước (phân biệt bằng loại dự án) | Hợp đồng ký trước hay sau BBNT? "Căn cứ triển khai" tối thiểu gồm giấy tờ nào? |
| G02 | Kênh A: đầu mối đối tác gửi BBNT cho Chuyển giao; Chuyển giao chịu trách nhiệm theo dõi và đòi biên bản; hạn gửi là tham số theo đối tác `[CẦN LÀM RÕ VỚI KH]`; quá hạn → nhắc người phụ trách và Trưởng phòng Chuyển giao | Hợp đồng khung có quy định hạn gửi biên bản không? |
| G03 | Đối soát theo kỳ (tháng/quý theo đối tác). Danh sách dự án kênh A đã BBNT trong kỳ do Chuyển giao ghi nhận; đối tác gửi số doanh thu; hệ thống so khớp, tính phần chia theo tỷ lệ hiệu lực, lập BBĐS. Đối tác thu của bệnh viện, Công ty A xuất hóa đơn cho đối tác phần mình được hưởng | Cơ sở tính là doanh thu thực thu hay phát sinh? Dòng tiền chiều nào? |
| G04 | Tỷ lệ chia sẻ gắn với đối tác (và nhóm sản phẩm nếu cần), mặc định theo hợp đồng khung, có ngày bắt đầu/kết thúc hiệu lực, áp theo **ngày BBNT của dự án**. Đổi tỷ lệ giữa kỳ: dự án có BBNT trước ngày hiệu lực dùng tỷ lệ cũ, sau đó dùng tỷ lệ mới; chỉ thêm bản ghi mới, không sửa bản ghi cũ. Ngoại lệ theo dự án cần căn cứ bằng văn bản và phê duyệt | Tỷ lệ áp theo đối tác, sản phẩm hay loại hợp đồng? Đổi giữa kỳ tính theo ngày nào? |
| G05 | Mọi yêu cầu quy về một bản ghi có trường *Nguồn* (cổng, điện thoại, tin nhắn, email, nhân viên triển khai). Yêu cầu ngoài cổng do người nhận nhập hộ và vẫn cần xác nhận của đầu mối IT bệnh viện trên hệ thống trước khi chuyển xử lý | IT bệnh viện có thể xác nhận qua email/đường liên kết không? |
| G06 | CSKH phân loại rủi ro sơ bộ theo danh mục (mặc định: sửa/xóa dữ liệu, đổi quy trình; cấu hình được); Phần mềm xác nhận hoặc nâng mức. Yêu cầu rủi ro cần lãnh đạo bệnh viện xác nhận (ghi người, thời điểm, nội dung); chưa xác nhận thì không xử lý | Danh mục rủi ro đầy đủ gồm những gì? |
| G07 | Nhánh tính phí: Phần mềm đánh dấu "có thể tính phí" → PGĐ Chuyên môn duyệt → Kinh doanh báo giá → bệnh viện chấp nhận (có bằng chứng) mới làm. Miễn/giảm/ưu đãi do Giám đốc duyệt | Ngưỡng giá trị nào cần Giám đốc duyệt? |
| G08 | Cổng phục vụ cả bệnh viện kênh A và B (sản phẩm do Công ty A làm). Kênh chỉ quyết định hợp đồng/doanh thu, không quyết định ai xử lý yêu cầu kỹ thuật. Yêu cầu tính phí của bệnh viện kênh A không báo giá trực tiếp mà thông báo cho đầu mối đối tác | Phí yêu cầu thêm ở kênh A thu và chia thế nào? |
| G09 | Bốn mốc chuẩn: Bắt đầu triển khai, Vận hành, Nghiệm thu (ngày ký BBNT), Rút. Mốc 5 ngày tính từ ngày ký BBNT; mốc 10 ngày tính từ ngày Kinh doanh nhận BBNT; doanh thu ghi nhận theo ngày BBNT; thưởng tính theo số ngày triển khai thực tế (bắt đầu → rút) so với định mức. Mỗi mốc có một người nhập và một người xác nhận | Xác nhận định nghĩa 4 mốc; thưởng tính theo ngày nào? |
| G10 | Thêm trạng thái *Chờ nghiệm thu* (xong kỹ thuật, chưa có BBNT) có người phụ trách (trưởng nhóm), lý do chặn và số ngày tồn | Ai chịu trách nhiệm khi bệnh viện/đối tác trì hoãn nghiệm thu lâu? |
| G11 | Kênh B: Kinh doanh đề nghị xuất hóa đơn kèm điều kiện (BBNT, hợp đồng, đợt hoặc kỳ thuê); Kế toán xuất hóa đơn trên phần mềm kế toán pháp định rồi ghi số hóa đơn, ngày, giá trị vào hệ thống. Kênh A: hóa đơn cho đối tác xuất từ kết quả đối soát | Ai là người đề nghị xuất hóa đơn, theo điều kiện nào? |
| G12 | Trưởng nhóm lập danh sách việc tồn khi chuẩn bị rút (có thể rỗng nhưng phải xác nhận "không còn tồn"); CSKH xác nhận nhận hoặc trả lại kèm lý do và hệ thống báo Trưởng phòng Chuyển giao; việc từ xa ghi như đầu việc thường với hình thức "từ xa", có người phụ trách và hạn | Có trường hợp nào được rút mà vẫn còn tồn không? |
| G19 | Cổng rủi ro và cổng tính phí là hai cổng độc lập; yêu cầu vừa rủi ro vừa tính phí phải qua **cả hai** trước khi sang "Đã duyệt" (chạy song song) | Có thứ tự bắt buộc giữa hai cổng không? |
| G20 | "Số tháng mốc" (đề, mục Kiến trúc) hiểu là số tháng kể từ mốc (BBNT) đến hết bảo hành/đến lúc nhắc gia hạn | "Số tháng mốc" chính xác là mốc nào, tính từ ngày nào? |
| G21 | Ngày Rút và ngày BBNT **độc lập về thứ tự** (có thể Rút trước hoặc sau BBNT); hai trục trạng thái của D2 xử lý riêng | Thực tế thường rút trước hay sau nghiệm thu? |
| G22 | Tham số chia sẻ doanh thu: **Kế toán** đề xuất → **PGĐ Tài chính** duyệt; ngoại lệ dưới ngưỡng do PGĐ Tài chính duyệt, vượt ngưỡng do Giám đốc duyệt; người đề xuất khác người duyệt; IT nội bộ không sửa giá trị nghiệp vụ; Chuyển giao/Kinh doanh không xem tỷ lệ | Ai nhập, ai duyệt? Ngưỡng nào cần Giám đốc? PGĐ Kinh doanh có được xem tỷ lệ không? |
| G23 | Kỳ đã chốt BBĐS **không sửa**; thay đổi hồi tố được xử lý bằng dòng điều chỉnh có căn cứ vào kỳ kế tiếp hoặc BBĐS bổ sung, PGĐ Tài chính duyệt | Thực tế xử lý điều chỉnh sau chốt thế nào? |
| G24 | Trong MVP đối tác **không đăng nhập** hệ thống; đầu mối đối tác vẫn báo dự án, gửi BBNT và số liệu qua Chuyển giao/Kế toán như hiện nay | Đối tác có muốn/được phép tự nhập trong giai đoạn sau không? |
| G25 | Dung sai chênh lệch số liệu đối soát mặc định là **không dung sai**; mọi dòng lệch phải được giải thích hoặc điều chỉnh có căn cứ | Có chấp nhận dung sai nào không, mức bao nhiêu? |

---

## 2b. Hướng phân tích (Analysis Approach)

**Approach đã chốt:** `Hybrid` — FR-01 đến FR-04 theo **UC-driven** (UC làm gốc, sinh AC/BR) đồng thời có `US-XX` ánh xạ về FR để đáp ứng Phần B của đề; FR-05 trở đi theo **Requirement thuần**.

**Điểm approach (Bước 1b):** `3/4` — các câu trả lời "Có": #1 (đã có actor + kịch bản từng bước: cổng 4 bước, bảng kênh A/B), #2 (≥3 actor chuyển giao qua lại: Chuyển giao → Kinh doanh → Kế toán → CSKH; Người dùng BV → IT BV → Công ty A → Lãnh đạo BV), #3 (B2B với đối tác bán hàng). #4 "Không" (đề không bắt buộc Use Case Diagram). Quy tắc: ≥2 → Use Case-first. *(Nếu coi #3 là "Không" vì không tích hợp hệ thống thì điểm 2, vẫn Use Case-first.)*

**Xác nhận KH khi biên/rủi ro cao:** `Đã xác nhận` — lý do phải hỏi: dự kiến nhiều FR Lớn (rủi ro cao) và đề bắt buộc User Story ở Phần B. PO xác nhận Hybrid trong phiên làm việc; Công ty A chưa xác nhận.

**Case & tín hiệu vá điểm mù:** `Case A` (có AS-IS thủ công, rời rạc). Tín hiệu: `Case A + Gap Process` có xuất hiện nhưng điểm 3/4 không thấp nên không làm đổi kết luận; ghi nhận để kiểm tra lại ở checkpoint cuối Bước 3 (đã re-check: Gap lộ thêm hand-off nhiều bên — củng cố UC-driven, không đảo ngược).

**Lý do ngắn:** chuỗi nghiệp vụ trải 4–5 phòng và có bên ngoài (đối tác, bệnh viện); điểm đứt nằm ở các điểm bàn giao nên UC (luồng chính/phụ/ngoại lệ) mô tả sát hơn US. US vẫn cần cho Phần B nên chỉ dùng làm lớp ánh xạ — **AC chỉ có một bộ, sinh từ UC**.

**Migration approach:** *(chưa có thay đổi)*

**Bảng User Story** *(Given/When/Then đầy đủ ở Phần B.2; Feature Spec ánh xạ mỗi kịch bản sang `AC-XX.k.n`)*

| Mã | User Story (As a… I want… so that…) | Rx nguồn | FR dẫn ra |
|---|---|---|---|
| US-01 | Là Trưởng phòng/PGĐ, tôi muốn xem một dự án đang ở bước nào của chuỗi triển khai → BBNT → hợp đồng → hóa đơn → thu tiền và đang kẹt ở ai, để biết nhắc đúng người | R25 | FR-01 |
| US-02 | Là Trưởng nhóm triển khai, tôi muốn ghi 4 mốc ngày và BBNT trên cùng một dự án, để mọi phòng thấy cùng một dữ liệu | R19, R25 | FR-01 |
| US-03 | Là Nhân viên Kinh doanh, tôi muốn được giao việc hoàn tất hợp đồng ngay khi nhận BBNT và được nhắc theo hạn, để không để quá 10 ngày mà không biết | R20, R25 | FR-01 |
| US-04 | Là Nhân viên Kinh doanh, tôi muốn theo dõi hồ sơ/hóa đơn đã gửi (gọi xác nhận 1–2 ngày, hợp đồng gốc 2 tuần), để không bỏ sót | R21 | FR-01 |
| US-05 | Là Kế toán, tôi muốn nhận đề nghị xuất hóa đơn đủ điều kiện, ghi số hóa đơn và nhắc đòi nợ 15 ngày/lần, báo Trưởng phòng sau 3 lần, để công nợ không treo | R9, R22 | FR-01 |
| US-06 | Là nhân viên Chuyển giao, tôi muốn ghi nhận dự án kênh A do đầu mối đối tác báo kèm giấy tờ điều động, để dự án kênh A không nằm ngoài hệ thống | R10, R16 | FR-02 |
| US-07 | Là nhân viên Chuyển giao, tôi muốn theo dõi BBNT đối tác chưa gửi và được nhắc, để việc kẹt không bị bỏ quên | R26 | FR-02 |
| US-08 | Là PGĐ Tài chính, tôi muốn lập kỳ đối soát theo đối tác, đối chiếu danh sách dự án với số liệu đối tác và ra BBĐS, để đối soát kiểm chứng được | R26 | FR-02 |
| US-09 | Là người quản trị tham số, tôi muốn cấu hình tỷ lệ chia sẻ, chu kỳ đối soát theo đối tác có hiệu lực theo ngày và ngoại lệ có căn cứ, để không phụ thuộc một người | R5, R26 | FR-02, FR-06 |
| US-10 | Là người dùng bệnh viện, tôi muốn gửi yêu cầu trên cổng và theo dõi trạng thái, để yêu cầu không thất lạc | R12, R28 | FR-03 |
| US-11 | Là IT bệnh viện, tôi muốn duyệt hoặc từ chối yêu cầu trước khi sang Công ty A, để chỉ yêu cầu đã duyệt mới được xử lý | R13 | FR-03 |
| US-12 | Là CSKH/Phần mềm, tôi muốn đánh giá rủi ro yêu cầu và gửi lãnh đạo bệnh viện xác nhận trước khi xử lý, để tránh tranh cãi "ai đã đồng ý" | R14, R28 | FR-03 |
| US-13 | Là PGĐ Chuyên môn, tôi muốn duyệt yêu cầu tính phí để Kinh doanh báo giá (miễn/giảm do Giám đốc), để không làm miễn phí việc lẽ ra thu tiền | R11, R28 | FR-03 |
| US-14 | Là Trưởng nhóm triển khai, tôi muốn lập danh sách việc tồn (cả việc từ xa) khi chuẩn bị rút, để bàn giao có danh sách | R29 | FR-04 |
| US-15 | Là CSKH, tôi muốn xác nhận nhận bàn giao hoặc trả lại kèm lý do, để biết mình nhận những tồn đọng nào | R29 | FR-04 |

---

## 3. Yêu cầu phi chức năng chung (NFR)

*(Áp dụng cho toàn hệ thống. NFR chặt hơn/riêng cho một feature Lớn ghi ở mục 6 của Feature Spec Lớn.)*

**Kết quả gate:** `Có tín hiệu (Client nêu số / domain nhạy cảm)` — (b) domain nhạy cảm: dữ liệu tài chính (hóa đơn, công nợ, doanh thu chia sẻ), dữ liệu lương/thưởng/hồ sơ nhân sự, doanh nghiệp phần mềm y tế (cổng khách hàng có thể chứa dữ liệu bệnh viện). Đề **không** nêu ngưỡng hiệu năng/uptime nào (các mốc 5/10/15 ngày là SLA nghiệp vụ, không phải NFR) nên các nhóm thiếu số liệu ghi `[CẦN LÀM RÕ VỚI KH]`, **chưa gán mã**.

| # | Nhóm NFR | Câu hỏi kích hoạt | Mã | Giá trị / ngưỡng đo được / phương pháp đo |
|---|---|---|---|---|
| 1 | Hiệu suất | Có yêu cầu thời gian phản hồi/tải? | — | `[CẦN LÀM RÕ VỚI KH]` — đề không nêu; cần số người dùng đồng thời và thời gian phản hồi mong muốn |
| 2 | Bảo mật | Có dữ liệu nhạy cảm / phân quyền? | **NFR-G2.1** | Mỗi người dùng chỉ xem/sửa dữ liệu trong **vai trò và phạm vi dữ liệu** được cấp; không có truy cập vượt phạm vi. Đo: kiểm thử theo ma trận quyền (mỗi vai trò × mỗi chức năng × phạm vi), kết quả mong đợi 0 truy cập vượt quyền. *(nguồn: R4)* |
| 3 | Độ tin cậy + Sao lưu/Phục hồi | Yêu cầu không mất dữ liệu / RPO-RTO? | — | `[CẦN LÀM RÕ VỚI KH]` — nghi ngờ cần (dữ liệu tài chính, một CSDL chung) nhưng đề chưa nêu RPO/RTO |
| 4 | Tính sẵn sàng | Cam kết uptime / SLA hệ thống? | — | `[CẦN LÀM RÕ VỚI KH]` |
| 5 | Khả dụng + Đa ngôn ngữ | Yêu cầu UX/ngôn ngữ/định dạng? | — | `[CẦN LÀM RÕ VỚI KH]` — ngôn ngữ giao diện, định dạng ngày/tiền tệ chưa nêu |
| 6 | Khả năng mở rộng | Dữ liệu/người dùng tăng mạnh? | — | `[CẦN LÀM RÕ VỚI KH]` — đề chỉ nêu "hàng trăm bệnh viện", chưa có số người dùng nội bộ, số dự án/năm |
| 7 | Khả năng bảo trì | Có yêu cầu về cấu hình thay vì code? | **NFR-G7.1** | Mọi tham số nghiệp vụ (tỷ lệ chia sẻ, chu kỳ đối soát, số tháng mốc, định mức thưởng, các mốc 5/10/15 ngày) thay đổi được qua giao diện cấu hình, **không cần phát hành bản mới**; có giá trị mặc định và ngoại lệ có ghi căn cứ. Đo: với mỗi tham số, đổi giá trị trên môi trường kiểm thử và xác nhận hành vi mới áp dụng đúng. *(nguồn: R5)* |
| 8 | Tương thích + Khả chuyển | Trình duyệt/thiết bị? | **NFR-G8.1** | Mọi chức năng của bàn làm việc và "Việc của tôi" dùng được trên máy tính và điện thoại qua trình duyệt web. Đo: kiểm thử trên danh sách trình duyệt/thiết bị `[CẦN LÀM RÕ VỚI KH]`. *(nguồn: R1)* |
| 9 | Tuân thủ pháp lý | Dữ liệu cá nhân / hóa đơn điện tử / chuẩn ngành? | — | `[CẦN LÀM RÕ VỚI KH]` — hệ thống lưu hồ sơ nhân sự (dữ liệu cá nhân); cổng khách hàng có lưu dữ liệu bệnh nhân không? Hóa đơn pháp định nằm ngoài hệ thống (R9) |
| 10 | Khả năng truy vết | Cần log ai làm gì, khi nào? | **NFR-G10.1** | **Mọi** phê duyệt và thay đổi quan trọng được lưu vết gồm: người thực hiện, thời điểm, giá trị trước/sau, căn cứ (nếu có). Đo: mỗi thao tác thuộc danh sách phê duyệt/thay đổi quan trọng tạo đúng 1 bản ghi vết, kiểm tra bằng test và rà soát nhật ký; danh sách "thay đổi quan trọng" chốt ở spec từng FR. *(nguồn: R4)* |

---

## 4. Danh sách tính năng (Feature Backlog) + Trạng thái Traceability

> Bảng trace tổng của dự án (thay cho RTM riêng). ✅ xong · ⏳ chưa/đang · — không áp dụng.
> **Ai đóng cột nào:** skill 1 đặt trạng thái ban đầu · skill 1.5 đóng **Model** · skill 2 đóng **Wireframe** · skill 3 đóng **Test** và **NFR**.
> **Loại Nhỏ/Lớn** phân theo *tín hiệu* (≥2 phần độc lập, phụ thuộc hai chiều, rủi ro cao đi sai hướng, cần NFR riêng) chứ không theo số tuần, vì chưa có năng lực đội phát triển để ước lượng — cột Effort để `Chưa ước lượng`.
> **Cột NFR:** NFR chung `NFR-G2.1`, `NFR-G7.1`, `NFR-G8.1`, `NFR-G10.1` áp dụng cho toàn hệ thống nên mọi FR đều `⏳`.
> Loại có dấu `*` là dự kiến, xác nhận lại khi viết spec. Các FR từ FR-07 trở đi ở mức **hướng đi** (đề cho phép xử lý mức hướng đi); FR-07 được chọn làm luồng TO-BE tùy chọn ở Phần A.3.4.

| Mã | Tên tính năng | Approach | US | Loại | Effort | Trạng thái | Spec | Model | Wireframe | NFR | Test | Link Feature Spec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| FR-01 | Chuỗi Dự án → BBNT → Hợp đồng → Hóa đơn → Thu tiền (kênh B), mốc và cảnh báo — KK01 | UC (+US ánh xạ) | US-01…05 | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-01-chuoi-du-an-doanh-thu.md |
| FR-02 | Kênh A: ghi nhận dự án đối tác, BBNT đối tác, kỳ đối soát, BBĐS, tham số chia sẻ — KK02 | UC (+US ánh xạ) | US-06…09 | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-02-kenh-a-doi-soat.md |
| FR-03 | Cổng yêu cầu khách hàng (nhánh rủi ro, nhánh tính phí) — KK04 | UC (+US ánh xạ) | US-10…13 | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-03-cong-yeu-cau-khach-hang.md |
| FR-04 | Bàn giao Triển khai → CSKH kèm danh sách việc tồn, việc từ xa — KK05 | UC (+US ánh xạ) | US-14…15 | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-04-ban-giao-cskh.md |
| FR-05 | Bàn làm việc, "Việc của tôi" và cảnh báo SLA (xuyên suốt) — hỗ trợ KK01/09/11 | Requirement thuần | — | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-05-ban-lam-viec-viec-cua-toi.md |
| FR-06 | Danh mục nền (Khách hàng, Đối tác, Sản phẩm, Nhân sự), phân quyền và cấu hình tham số | Requirement thuần | US-09 (một phần) | Lớn | Chưa ước lượng | Draft | ⏳ | ⏳ | ⏳ | ⏳ | ⏳ | ./FR-06-danh-muc-nen-tham-so.md |
| FR-07 | Hợp đồng định kỳ và gia hạn (thuê, bảo trì, thuê máy chủ) — KK03 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-07-hop-dong-dinh-ky.md |
| FR-08 | Điều động, công tác, nghỉ bù — KK06 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-08-dieu-dong-cong-tac.md |
| FR-09 | Xét thưởng triển khai — KK07 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-09-xet-thuong.md |
| FR-10 | Thiết bị kết nối (LIS) — KK08 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-10-thiet-bi-ket-noi.md |
| FR-11 | Giao việc nội bộ — KK09 | Requirement thuần | — | Nhỏ* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-11-giao-viec-noi-bo.md |
| FR-12 | Tri thức xử lý lỗi và đặc thù từng viện — KK10 | Requirement thuần | — | Nhỏ* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-12-tri-thuc.md |
| FR-13 | Báo cáo điều hành — KK11 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-13-bao-cao-dieu-hanh.md |
| FR-14 | Hành chính (chấm công, phép, ngoài giờ, suất ăn, tài sản) — KK12 | Requirement thuần | — | Lớn* | Chưa ước lượng | Hướng đi | ⏳ | — | ⏳ | ⏳ | ⏳ | ./FR-14-hanh-chinh.md |

**Trạng thái tính năng:** `Draft` → `Đã gửi KH` → `Đã confirm` → `Đang phát triển` → `Hoàn thành`

### 4b. Ưu tiên và phân kỳ phát hành (từ Phần B.3)

> Phương pháp WSJF (điểm tương đối do PO chấm, kiểm chứng ở Phần B.5) + lớp phủ MoSCoW. Thứ hạng đã điều chỉnh theo phụ thuộc và quyết định có chủ ý (chi tiết ở B.3). FR-05 và FR-06 là nền của Phát hành 1, không có khó khăn riêng.

| Hạng | FR | Khó khăn | WSJF | Phát hành | MoSCoW |
|---|---|---|---|---|---|
| — | FR-05, FR-06 | (nền) | — | 1 (MVP) | Must |
| 1 | FR-01 | KK01 | 3.00 | 1 (MVP) | Must |
| 2 | FR-04 | KK05 | 4.50 | 1 (MVP) | Must |
| 3 | FR-02 | KK02 | 2.60 | 1 (MVP) | Must |
| 4 | FR-03 | KK04 | 2.00 | 1 (MVP) | Must |
| 5 | FR-07 | KK03 | 3.00 | 2 | Should |
| 6 | FR-11 | KK09 | 3.00 | 2 | Should |
| 7 | FR-12 | KK10 | 2.50 | 2 | Could |
| 8 | FR-10 | KK08 | 2.33 | 2 | Could |
| 9 | FR-08 | KK06 | 2.00 | 3 (MVP chỉ thu dữ liệu điều động tối thiểu) | Could |
| 10 | FR-13 | KK11 | 2.00 | 3 (MVP có phễu trên bàn làm việc PGĐ Tài chính) | Could |
| 11 | FR-09 | KK07 | 1.75 | 3 | Won't (đợt này) |
| 12 | FR-14 | KK12 | 1.00 | 3 | Won't (đợt này) |

> **Quy ước mã (không đổi):** `Rx` nguyên tử yêu cầu · `FR-XX` tính năng (khớp tên file spec), sub `FR-XX.1`… · `US-XX` user story · `UC-XX.n` use case · `BR-XX.n` quy tắc nghiệp vụ · `AC-XX.n` (Nhỏ) / `AC-XX.k.n` (Lớn) tiêu chí nghiệm thu · `NFR-G<#>.n` NFR chung · `NFR-XX.n` NFR riêng FR · `[WIREFRAME: <tên màn>]` · `[MODEL: <loại>]` · `[CẦN LÀM RÕ VỚI KH]`.

---

## 5. Phụ lục

### 5.1 Nguyên tử yêu cầu (Rx)

| Mã | Nội dung | Nguồn trong đề |
|---|---|---|
| R1 | Một ứng dụng web chạy trên máy tính và điện thoại, một CSDL chung cho mọi phòng | Kiến trúc |
| R2 | Khách hàng là trung tâm; mọi nghiệp vụ bắt đầu từ một yêu cầu/sự kiện có nguồn gốc rõ ràng | Kiến trúc |
| R3 | Nghiệp vụ chạy theo workflow trạng thái; không xử lý ngoài luồng với nghiệp vụ chính | Kiến trúc |
| R4 | Phân quyền theo vai trò và phạm vi dữ liệu; mọi phê duyệt và thay đổi quan trọng đều lưu vết | Kiến trúc |
| R5 | Ưu tiên cấu hình: tỷ lệ chia sẻ, chu kỳ đối soát, số tháng mốc, định mức thưởng là tham số, có mặc định và ngoại lệ có căn cứ | Kiến trúc |
| R6 | Bàn làm việc theo đối tượng/vai trò | Trải nghiệm |
| R7 | Trang "Việc của tôi": việc đến hạn, quá hạn, yêu cầu cần xử lý, lịch công tác, thông báo | Trải nghiệm |
| R8 | SLA chỉ dùng báo cáo nội bộ và cảnh báo, không phạt tự động | Trải nghiệm |
| R9 | Không thay phần mềm kế toán pháp định; quản lý hóa đơn/công nợ ở mức quản trị | Ranh giới |
| R10 | Kênh A chỉ ghi nhận dự án triển khai và giấy tờ điều động; không quản lý thầu/hợp đồng nội bộ đối tác | Ranh giới |
| R11 | Chính sách khách hàng (giảm giá, miễn phí, ưu đãi) phải qua Giám đốc duyệt | Ranh giới |
| R12 | Cổng bước 1: người dùng bệnh viện nhập yêu cầu | Cổng khách hàng |
| R13 | Cổng bước 2: IT bệnh viện duyệt; chỉ yêu cầu đã duyệt mới sang Công ty A | Cổng khách hàng |
| R14 | Cổng bước 3: yêu cầu rủi ro gửi lãnh đạo bệnh viện xác nhận trước khi xử lý | Cổng khách hàng |
| R15 | Cổng bước 4: yêu cầu phổ thông duyệt thẳng cho Phần mềm; CSKH cập nhật cho khách | Cổng khách hàng |
| R16 | Hai kênh A/B chạy song song, khác nhau về ai bán, hợp đồng, ai báo việc, tiền về | Bối cảnh |
| R17 | Một bệnh viện có thể thuộc cả hai kênh hoặc hai đối tác khác nhau cho hai sản phẩm | Bối cảnh |
| R18 | Vòng đời: Khách tiềm năng → Demo/khảo sát → Báo giá → Hợp đồng → Dự án → Điều động → Triển khai → Nghiệm thu → Bảo hành → Bảo trì → Gia hạn | Đầu việc |
| R19 | Chuyển giao nộp BBNT cho Kinh doanh tối đa 5 ngày làm việc kể từ ngày ký | Quy định nội bộ |
| R20 | Kinh doanh ký hợp đồng tối đa 10 ngày kể từ khi nhận BBNT (trừ đấu thầu và kênh A); quá hạn báo Trưởng phòng | Quy định nội bộ |
| R21 | Sau khi gửi hồ sơ, 1–2 ngày gọi xác nhận; 2 tuần chưa nhận hợp đồng gốc thì báo Trưởng phòng | Quy định nội bộ |
| R22 | Kế toán đòi nợ 15 ngày/lần; sau 3 lần chưa thu được báo Trưởng phòng Kế toán | Quy định nội bộ |
| R23 | Nghỉ bù: Chuyển giao chỉ tính đợt công tác liên tục từ 10 ngày; phòng khác cộng dồn đủ 10 ngày được 1 ngày | Quy định nội bộ |
| R24 | Thưởng triển khai tính theo quy mô giường, số bộ phận, loại dự án, số ngày vượt/chậm so với định mức | Quy định nội bộ |
| R25–R36 | Khó khăn KK01–KK12 (R25 = KK01 … R36 = KK12) | Khó khăn vận hành |
| R37 | Phần A: chẩn đoán, AS-IS, TO-BE (≥4 luồng), máy trạng thái, mô hình dữ liệu, bàn làm việc 3 vai trò, 5–8 chỉ số, câu hỏi mở | Yêu cầu bài làm |
| R38 | Phần B: story map MVP, 10–15 user story có Given/When/Then, ưu tiên backlog, quy tắc chia sẻ doanh thu kênh A, cách xác nhận giả thuyết | Yêu cầu bài làm |
| R39 | Xử lý sâu KK01, KK02, KK04, KK05; các mã còn lại mức hướng đi | Yêu cầu bài làm |
| R40 | Tài liệu chính ≤ 20 trang; phải giải thích được mọi quyết định khi bảo vệ (kèm tình huống phát sinh) | Yêu cầu bài làm |

### 5.2 Gap Analysis AS-IS → TO-BE (Case A)

**Bốn khó khăn bắt buộc**

| KK | AS-IS (theo đề) | TO-BE (hướng đề xuất) | Loại Gap | Gốc rễ chính |
|---|---|---|---|---|
| KK01 | Chuỗi Triển khai → BBNT → HĐ → Hóa đơn → Thu tiền đứt ở mỗi lần đổi phòng; mốc 5/10/15 ngày chỉ có trên giấy | Một Dự án là xương sống; mỗi mốc là một trạng thái có chủ và hạn; hết hạn cảnh báo người phụ trách và Trưởng phòng (không phạt tự động) | Process, Data, System | Trách nhiệm (không ai sở hữu mốc) + Công cụ (không đo được) |
| KK02 | Kênh A không qua Kinh doanh, không có HĐ từng khách; đối soát thủ công, mỗi đối tác một tỷ lệ, phụ thuộc một người | Dự án kênh A cùng cấu trúc với kênh B, rẽ nhánh sang kỳ đối soát; tỷ lệ, chu kỳ, ngoại lệ là tham số có hiệu lực theo ngày và lưu vết | Process, Data, Policy, People | Công cụ (thiếu tham số, thiếu nguồn đối chiếu) + Trách nhiệm (một người) |
| KK04 | Yêu cầu đến qua nhiều kênh, thất lạc; rủi ro và tính phí chưa có luồng duyệt; tranh cãi "ai đã đồng ý" | Một bản ghi Yêu cầu có nguồn gốc qua cổng; nhánh rủi ro (lãnh đạo BV xác nhận) và nhánh tính phí (PGĐ duyệt, báo giá) | Process, System, Policy | Quy trình (không có luồng chuẩn) + Công cụ |
| KK05 | Rút đội mà việc dở không có danh sách bàn giao; việc từ xa quản lý kém | Dự án chỉ sang "Đã bàn giao" khi có danh sách tồn và CSKH xác nhận nhận | Process, People, Data | Quy trình (thiếu điều kiện bàn giao) + Trách nhiệm (ai lập, ai nhận) |

**Các khó khăn còn lại — hướng đi**

| KK | Hướng đi TO-BE | FR |
|---|---|---|
| KK03 | Hợp đồng định kỳ có lịch kỳ thanh toán (tháng/quý/6 tháng/năm); hệ thống sinh nhắc xuất hóa đơn kỳ và nhắc gia hạn theo "số tháng mốc"; hết bảo hành nhắc ký bảo trì | FR-07 |
| KK06 | Điều động theo dự án × người × ngày, kiểm tra trùng lịch; lệnh công tác, xe, tạm ứng, quyết toán gắn dự án; nghỉ bù tự tính theo tham số | FR-08 |
| KK07 | Tính công theo bản ghi điều động thực tế (4 mốc + vai trò trưởng nhóm); thưởng theo công thức tham số (giường, bộ phận, loại dự án, ngày vượt/chậm) | FR-09 |
| KK08 | Thực thể Thiết bị gắn dự án LIS (từng máy), trạng thái chạy/đã thu tiền; chỉ **cảnh báo/ghi nhận** việc ngắt kết nối (G15) | FR-10 |
| KK09 | Đầu việc có người giao, người nhận, hạn, kết quả; nằm trong "Việc của tôi" | FR-11 |
| KK10 | Kho tri thức gắn lỗi/khách hàng/yêu cầu; đóng yêu cầu thì gợi ý lưu cách xử lý | FR-12 |
| KK11 | Dashboard điều hành tổng hợp tự động từ dữ liệu FR-01…05 (doanh thu, công nợ, tiến độ, tồn đọng, hiệu quả phòng) | FR-13 |
| KK12 | Chấm công, phép, ngoài giờ, suất ăn, tài sản dùng chung nhân sự; nghỉ bù công tác liên thông với phép năm | FR-14 |

### 5.3 Câu hỏi còn mở (Open Questions)

**Đã đóng bằng giả định (Blocking → giả định tạm):** G01–G12, G19–G25 (mục 2). Rủi ro làm lại cao nhất nếu sai: **G01, G03, G04** (nền của máy trạng thái và quy tắc chia sẻ doanh thu).

**Nice-to-know (giữ `[CẦN LÀM RÕ VỚI KH]`):**

| Mã | Câu hỏi |
|---|---|
| G13 | Quyền và phạm vi dữ liệu: Trưởng nhóm thay giữa chừng; vai trò của PGĐ Kinh doanh trong hệ thống |
| G14 | Công thức nghỉ bù của Chuyển giao (đợt ≥10 ngày liên tục được bao nhiêu ngày); định mức thưởng; SLA xử lý yêu cầu |
| G15 | KK08: hệ thống chỉ cảnh báo/ghi nhận hay thực sự ngắt kết nối máy xét nghiệm? |
| G16 | Quy mô (người dùng, dự án/năm, số bệnh viện, số đối tác) và dữ liệu nhạy cảm (lương/thưởng, dữ liệu bệnh viện trên cổng) — cần để chốt NFR nhóm 1, 3, 4, 6, 9 |
| G17 | Số liệu nền (baseline) cho chỉ số: số dự án/năm, tỷ lệ đã có đủ hợp đồng/hóa đơn (đề chỉ cho ~3/4 không xác định được) |
| G18 | Lỗi nhỏ trong đề: "KK-11" ghi khác "KK11"; "Kế toán" và "Kế toán/Tài chính"; phòng Nghiên cứu không có đầu việc |
| G26 | Dữ liệu lịch sử: MVP chỉ nhập **dự án đang mở** và danh mục nền (khách hàng, đối tác, sản phẩm, nhân sự). Dữ liệu dự án đã đóng và nguồn dữ liệu hiện có (bảng tính, giấy tờ) ở đâu? |

### 5.4 Kiểm tra trace hai chiều (Bước 7)

**Xuôi (Rx → FR):**

| Rx | FR / NFR |
|---|---|
| R1 | NFR-G8.1, FR-05 |
| R2 | FR-01, FR-03, FR-06 |
| R3 | FR-01, FR-02, FR-03, FR-04 |
| R4 | NFR-G2.1, NFR-G10.1, FR-06 |
| R5 | NFR-G7.1, FR-06 |
| R6, R7, R8 | FR-05 |
| R9 | FR-01 (ranh giới: Ngoài phạm vi kế toán pháp định) |
| R10 | FR-02 |
| R11 | FR-03 |
| R12–R15 | FR-03 |
| R16 | FR-01, FR-02 |
| R17 | FR-06 (mô hình dữ liệu), FR-02 |
| R18 | FR-01 |
| R19, R20, R21, R22 | FR-01 |
| R23 | FR-08, FR-14 |
| R24 | FR-09 |
| R25–R36 | KK01→FR-01; KK02→FR-02; KK03→FR-07; KK04→FR-03; KK05→FR-04; KK06→FR-08; KK07→FR-09; KK08→FR-10; KK09→FR-11; KK10→FR-12; KK11→FR-13; KK12→FR-14 |
| R37–R40 | Out of scope (ràng buộc sản phẩm nộp, không phải chức năng hệ thống) |

**Ngược (FR → Rx):** FR-01 ← R2, R3, R9, R16, R18–R22, R25; FR-02 ← R3, R10, R16, R17, R26; FR-03 ← R2, R3, R11–R15, R28; FR-04 ← R3, R29; FR-05 ← R1, R6–R8; FR-06 ← R2, R4, R5, R17; FR-07 ← R27; FR-08 ← R23, R30; FR-09 ← R24, R31; FR-10 ← R32; FR-11 ← R33; FR-12 ← R34; FR-13 ← R35; FR-14 ← R23, R36. Mọi FR có ≥1 Rx; mọi US ánh xạ ≥1 Rx và ≥1 FR (bảng mục 2b). Mọi NFR đã gán mã có nguồn (R1, R4, R5).

---

## 6. Change Request Log

> Ghi lại mọi thay đổi/bổ sung của SRS sau khi đã gửi/chốt. Mỗi thay đổi một dòng.

| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng FR | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 22/10/2026 | PO/BA | Bản draft đầu: Backbone, Approach Hybrid, giả định G01–G12 và G19–G21, NFR gate, backlog FR-01…FR-14 | — | Draft |
| v1.1 | 23/10/2026 | PO/BA | Đồng bộ với Phần B: thêm giả định G22–G25, Nice-to-know G26; thêm mục 4b (hạng, phát hành, MoSCoW); cập nhật quyền Giám đốc, PGĐ Tài chính, Kế toán, IT nội bộ; User Story có Given/When/Then ở Phần B.2 | FR-02, FR-06 (quyền tham số), toàn backlog (phát hành) | Draft |
