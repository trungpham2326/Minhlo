# Feature Spec: Bàn làm việc, "Việc của tôi" và cảnh báo SLA (xuyên suốt)

**Mã**: FR-05  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-05)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: Requirement thuần  
**User Story liên quan**: — (không có US riêng; là lớp nền cho US-01…US-15)  
**Khó khăn nguồn**: hỗ trợ KK01, KK09, KK11 (xuyên suốt) · **Rx nguồn**: R1, R6, R7, R8 · **Giả định liên quan**: G10, G13, G14, G16, G26 · **Quyết định nền**: D3, D4

> **Quy ước:** `Rx` là mã của Backbone mục 5.1. `[BA đề xuất]` là bổ sung chưa nằm trong Backbone v1.1, chờ xác nhận. Mã `QN-xx` là quy tắc nhắc (mục 3.3); mỗi QN dẫn về BR của FR gốc, FR-05 không đặt lại nghiệp vụ của FR khác mà chỉ vận hành cơ chế nhắc, leo thang và hiển thị.

---

## 1. Mục đích
Biến các mốc và quy định nội bộ (5/10/15 ngày, 1–2 ngày, 2 tuần, 3 lần) từ "tồn tại trên giấy" thành thứ hệ thống tự theo dõi: mọi bước có chủ sinh ra một đầu việc có người phụ trách và hạn, nằm trong "Việc của tôi"; đến hạn thì nhắc người chủ, quá hạn thì leo thang lên Trưởng phòng mà không chờ người trễ tự báo (D3). Mỗi người mở hệ thống ra thấy ngay bàn làm việc của vai trò mình thay vì đi qua danh sách module (R6). SLA chỉ để đo, cảnh báo và báo cáo nội bộ, không phạt tự động (R8).

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Sự kiện nghiệp vụ từ FR-01…04 (mốc, chuyển trạng thái, đề nghị, phê duyệt); tham số thời hạn và lịch ngày làm việc (FR-06); quan hệ leo thang theo cơ cấu (FR-06); thao tác của người dùng trên đầu việc (ghi kết quả, đổi người); điều động tối thiểu (FR-04.4) |
| Xử lý | Sinh đầu việc theo bước có chủ; tính hạn theo tham số; đánh giá quy tắc nhắc hằng ngày và theo sự kiện; leo thang theo mức; chống nhắc trùng; đóng nhắc khi bước hoàn tất; tạo thông báo; tính số liệu SLA; dựng bàn làm việc theo vai trò và phạm vi quyền |
| Output | Trang "Việc của tôi"; thông báo trong ứng dụng và email; bàn làm việc theo vai trò; cảnh báo leo thang; số liệu tuân thủ mốc và phễu doanh thu cho báo cáo nội bộ; nhật ký nhắc và leo thang |

Sơ đồ: `[MODEL: ERD]` `DAU_VIEC`, `NHAN_SU`, `THAM_SO` (Hình A.5) · `[MODEL: Sequence]` các nhắc "Nhắc…, leo thang…" trong Hình A.3.1, A.3.2, A.3.3

## 3. Yêu cầu chi tiết (Requirement thuần)

### 3.1 Khái niệm nền
| Khái niệm | Định nghĩa |
|---|---|
| Đầu việc (`DAU_VIEC`) | Một việc có người phụ trách, hạn, trạng thái (Mở, Đang làm, Hoàn thành, Hủy), hình thức (tại đơn vị / từ xa / văn phòng), kết quả, và nguồn sinh (hệ thống theo mốc, hoặc do người dùng giao). Gắn với Dự án hoặc Yêu cầu |
| Quy tắc nhắc | Cấu hình gồm: sự kiện gốc, ngày gốc, hạn (tham số), loại ngày (làm việc/lịch), người nhận chính, người leo thang, nhịp lặp, điều kiện dừng |
| Nhắc | Một bản ghi thông báo gửi người nhận chính khi sắp đến hạn hoặc đến hạn |
| Leo thang | Nhắc gửi Trưởng phòng (hoặc vai trò chỉ định) khi quá hạn; không thay thế việc của người chủ mốc |
| Bàn làm việc | Trang đầu của một vai trò, gom các khối thông tin và thao tác theo vai trò (R6) |
| Khối (widget) | Một ô nội dung trên bàn làm việc, có nguồn dữ liệu và phạm vi quyền |

### 3.2 Mô hình hoạt động chung của bộ nhắc
1. Một bước nghiệp vụ có chủ bắt đầu (ví dụ Kinh doanh xác nhận đã nhận BBNT). → 2. Hệ thống sinh đầu việc cho người phụ trách, tính hạn theo tham số và ngày gốc. → 3. Khi sắp đến hạn (tham số "sắp đến hạn"), hệ thống nhắc người phụ trách. → 4. Khi quá hạn, hệ thống nhắc và leo thang Trưởng phòng theo quy tắc, ghi nhận. → 5. Khi bước nghiệp vụ hoàn tất, đầu việc tự hoàn thành và nhắc tự đóng; không có đường nào để người dùng đóng nhắc mà không hoàn tất bước.

### 3.3 Danh mục quy tắc nhắc (QN) cho MVP
| Mã | Sự kiện / bước | Người nhận chính | Leo thang | Hạn / nhịp (mặc định) | Nguồn BR |
|---|---|---|---|---|---|
| QN-01 | Dự án kênh B ở "Chờ nghiệm thu" quá hạn | Trưởng nhóm triển khai | Trưởng phòng Chuyển giao | `[CẦN LÀM RÕ VỚI KH]` (G10) | BR-01.7 |
| QN-02 | Nộp BBNT cho Kinh doanh | Nhân viên Chuyển giao phụ trách | Trưởng phòng Chuyển giao | 5 ngày làm việc kể từ ngày ký | BR-01.11 |
| QN-03 | Hoàn tất hợp đồng | Nhân viên Kinh doanh phụ trách | Trưởng phòng Kinh doanh | 10 ngày kể từ ngày nhận BBNT (loại ngày chờ làm rõ) | BR-01.15 |
| QN-04 | Gọi xác nhận khách đã nhận hồ sơ | Nhân viên Kinh doanh | — (chỉ nhắc nhân viên) | 1–2 ngày sau khi gửi | BR-01.19 |
| QN-05 | Hợp đồng gốc chưa về | — | Trưởng phòng Kinh doanh | 2 tuần sau khi gửi hồ sơ | BR-01.20 |
| QN-06 | Đòi nợ | Kế toán | Trưởng phòng Kế toán sau 3 lần chưa thu | Nhịp 15 ngày/lần | BR-01.24 |
| QN-07 | BBNT kênh A chưa gửi quá hạn; thiếu tham số hạn | Nhân viên Chuyển giao phụ trách; cảnh báo "thiếu tham số" cho người quản trị tham số | Trưởng phòng Chuyển giao | Tham số theo đối tác, không đặt số | BR-02.13 |
| QN-08 | Quá hạn chốt BBĐS mà đối tác chưa gửi số liệu | PGĐ Tài chính | — | Tham số `[CẦN LÀM RÕ VỚI KH]` | BR-02.22 |
| QN-09 | IT bệnh viện chưa phản hồi yêu cầu | IT bệnh viện (CSKH thấy tình trạng) | — | Tham số `[CẦN LÀM RÕ VỚI KH]` | BR-03.7 |
| QN-10 | Khách chưa xác nhận sau khi xử lý | CSKH | — | Tham số `[CẦN LÀM RÕ VỚI KH]` | BR-03.19 |
| QN-11 | Yêu cầu quá SLA nhận hoặc xử lý | Người phụ trách (Phần mềm/CSKH) | Trưởng phòng; ghi báo cáo nội bộ | SLA `[CẦN LÀM RÕ VỚI KH]` (G14) | BR-03.20 |
| QN-12 | Rà soát bàn giao quá hạn | CSKH phụ trách | Trưởng phòng CSKH | Tham số `[CẦN LÀM RÕ VỚI KH]` | BR-04.11 |
| QN-13 | Trả lại danh sách bàn giao | Trưởng nhóm (bổ sung) | Trưởng phòng Chuyển giao (báo mỗi lần trả lại) | Theo sự kiện | BR-04.9 |
| QN-14 | Đầu việc tồn/việc từ xa quá hạn | Người phụ trách | Trưởng phòng của người đó | Hạn của từng việc | BR-04.17 |
| QN-15 | Dự án đủ điều kiện đóng | CSKH phụ trách | — | Theo sự kiện | BR-04.24 |

Quy tắc mới của FR sau này (hợp đồng định kỳ, gia hạn, giao việc nội bộ…) thêm bằng cấu hình QN, không đổi mã nguồn (NFR-G7.1).

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-05.1 | R7 | Đầu việc và trang "Việc của tôi" | Chưa ước lượng | Draft | |
| FR-05.2 | R8, R19–R22 (qua FR-01) | Bộ nhắc và leo thang theo mốc/SLA | Chưa ước lượng | Draft | |
| FR-05.3 | R7 | Thông báo | Chưa ước lượng | Draft | |
| FR-05.4 | R1, R6 | Bàn làm việc theo vai trò | Chưa ước lượng | Draft | |
| FR-05.5 | R8 | Số liệu SLA, báo cáo nội bộ và phễu doanh thu | Chưa ước lượng | Draft | |

Vai trò, phân quyền, tham số, lưu vết, danh mục nhân sự và quan hệ leo thang ở FR-06. Báo cáo điều hành đầy đủ thuộc FR-13 (Phát hành 3); MVP chỉ có phễu trên bàn làm việc PGĐ Tài chính. Giao việc nội bộ tự do (người giao, hạn, kết quả) thuộc FR-11.

---

### FR-05.1 — Đầu việc và trang "Việc của tôi"
- **Mô tả**: Mọi bước có chủ sinh đầu việc; mỗi người có trang tổng hợp việc của mình.
- **Input – Xử lý – Output**: Input: sự kiện nghiệp vụ, thao tác người dùng. Xử lý: sinh đầu việc, tính hạn, phân nhóm, tự hoàn thành khi bước xong. Output: trang "Việc của tôi" gồm 5 khối.
- **Màn hình UI**: `[WIREFRAME: Việc của tôi]` · `[WIREFRAME: Chi tiết đầu việc]`

**Luồng xử lý:**
- **Happy**: bước nghiệp vụ bắt đầu → đầu việc xuất hiện trong "Việc của tôi" → xử lý → đầu việc tự hoàn thành.
- **Exception**: người xem không có quyền với dự án của đầu việc; cố đóng tay đầu việc hệ thống sinh.
- **Edge/Boundary**: đổi người phụ trách; người phụ trách nghỉ; nhiều đầu việc cùng hạn; đầu việc của dự án Tạm dừng; điện thoại.
- **Alternative**: đầu việc do người dùng giao (mức tối thiểu, mở rộng ở FR-11).

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-05.1 | Mọi bước có chủ trong FR-01…04 sinh một đầu việc có người phụ trách, hạn và nguồn sinh; không tồn tại đầu việc không có người phụ trách (D3) |
| BR-05.2 | "Việc của tôi" có 5 khối (R7): việc đến hạn, việc quá hạn, yêu cầu cần xử lý, lịch công tác, thông báo; chỉ hiển thị dữ liệu trong phạm vi quyền của người dùng (NFR-G2.1) |
| BR-05.3 | Sắp xếp mặc định: quá hạn (lâu nhất trước) → đến hạn → còn lại; lọc theo dự án, bệnh viện, loại việc, hình thức; hiển thị số lượng từng khối |
| BR-05.4 | Đầu việc do hệ thống sinh tự hoàn thành khi bước nghiệp vụ tương ứng hoàn tất; người dùng không tự đánh dấu hoàn thành (để dữ liệu mốc không tách khỏi trạng thái thật); từ đầu việc mở thẳng tới dự án/yêu cầu để thao tác |
| BR-05.5 | Đổi người phụ trách do Trưởng phòng của người đó thực hiện, lưu vết; đầu việc chuyển sang người mới; nhân sự nghỉ thì chuyển hàng loạt theo FR-06.1 |
| BR-05.6 | Khối "Lịch công tác" trong MVP chỉ đọc, lấy từ điều động tối thiểu (FR-04.4); lệnh công tác, xe, nghỉ bù thuộc FR-08 |
| BR-05.7 | Mọi khối dùng được trên máy tính và điện thoại (NFR-G8.1); thao tác thường dùng (ghi kết quả, mở dự án) làm được trên điện thoại |
| BR-05.8 | Đầu việc của dự án đang "Tạm dừng" hiển thị nhãn "Tạm dừng" và không bị tính quá hạn mới cho mốc triển khai; có tính vào hạn hay không `[CẦN LÀM RÕ VỚI KH]` (BR-04.22) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-05.1.1 | Happy | BR-05.1, BR-05.2 | R7 | **Given** Kinh doanh K xác nhận đã nhận BBNT (FR-01.2) **When** hệ thống sinh đầu việc "Hoàn tất hợp đồng" **Then** đầu việc có người phụ trách là K, hạn 10 ngày kể từ ngày nhận và xuất hiện trong khối "việc đến hạn" của K |
| AC-05.1.2 | Happy | BR-05.3 | R7 | **Given** K có 1 việc quá hạn 3 ngày, 1 việc quá hạn 1 ngày và 2 việc đến hạn hôm nay **When** K mở "Việc của tôi" **Then** thứ tự là quá hạn 3 ngày, quá hạn 1 ngày, rồi 2 việc đến hạn |
| AC-05.1.3 | Exception | BR-05.2 | NFR-G2.1 | **Given** người dùng chỉ có quyền trong phạm vi phòng mình **When** mở "Việc của tôi" **Then** không thấy đầu việc của dự án ngoài phạm vi |
| AC-05.1.4 | Exception | BR-05.4 | R3 | **Given** đầu việc "Nộp BBNT cho Kinh doanh" do hệ thống sinh **When** nhân viên thử đánh dấu hoàn thành mà chưa nộp BBNT trên dự án **Then** hệ thống không có thao tác đóng tay; đầu việc chỉ hoàn thành khi BBNT được nộp |
| AC-05.1.5 | Edge | BR-05.5 | R7 | **Given** nhân viên Kinh doanh nghỉ việc và còn 3 đầu việc mở **When** Trưởng phòng Kinh doanh đổi sang nhân viên khác **Then** 3 đầu việc xuất hiện ở người mới, biến mất ở người cũ và sự kiện đổi được ghi vết |
| AC-05.1.6 | Edge | BR-05.6 | R7 | **Given** Trưởng nhóm được điều động ở dự án P từ ngày 01/11 **When** mở khối "Lịch công tác" **Then** thấy dòng điều động của P chỉ đọc; không có lệnh công tác hay nghỉ bù trong MVP |
| AC-05.1.7 | Edge | BR-05.8 | BR-04.22 | **Given** dự án đang "Tạm dừng" **When** đến hạn của đầu việc mốc triển khai **Then** đầu việc hiển thị nhãn "Tạm dừng" và hệ thống không leo thang mới `[CẦN LÀM RÕ VỚI KH]` |
| AC-05.1.8 | Edge | BR-05.7 | NFR-G8.1 | **Given** người dùng mở "Việc của tôi" trên điện thoại **When** bấm một đầu việc **Then** mở được dự án liên quan và ghi được kết quả mà không phải chuyển sang máy tính |

---

### FR-05.2 — Bộ nhắc và leo thang theo mốc/SLA
- **Mô tả**: Đánh giá quy tắc nhắc (QN-01…QN-15), nhắc người chủ mốc, leo thang Trưởng phòng khi quá hạn.
- **Input – Xử lý – Output**: Input: sự kiện, ngày gốc, tham số, lịch ngày làm việc, quan hệ leo thang. Xử lý: tính hạn, đánh giá hằng ngày và theo sự kiện, chống trùng, dừng khi xong. Output: nhắc, leo thang, nhật ký.
- **Màn hình UI**: `[WIREFRAME: Cấu hình quy tắc nhắc]` · `[WIREFRAME: Cảnh báo leo thang của phòng]`

**Luồng xử lý:**
- **Happy**: bước bắt đầu → sắp đến hạn → nhắc người phụ trách → quá hạn → leo thang Trưởng phòng → bước xong thì dừng.
- **Exception**: thiếu tham số hạn; người nhận không xác định; người nhận nghỉ.
- **Edge/Boundary**: hạn rơi vào ngày nghỉ; nhịp đòi nợ 15 ngày; đổi tham số khi đang có việc mở; đổi người giữa chừng; dữ liệu trước "ngày bắt đầu đo".
- **Alternative**: quy tắc chỉ nhắc, không leo thang (QN-04); quy tắc chỉ leo thang (QN-05).

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-05.9 | Quy tắc nhắc là cấu hình (QN-xx): sự kiện gốc, ngày gốc, hạn (tham số), loại ngày, người nhận chính, người leo thang, nhịp, điều kiện dừng; đổi được không cần phát hành bản mới (R5, NFR-G7.1) |
| BR-05.10 | Nhắc người chủ mốc trước; khi quá hạn leo thang Trưởng phòng theo quan hệ cơ cấu (FR-06.1); không chờ người trễ tự báo (D3); mỗi mốc leo thang một lần mỗi mức, trừ quy tắc có nhịp (QN-06) |
| BR-05.11 | Nhắc và leo thang chỉ thông báo và ghi nhận, không tự phạt, không tự chặn nghiệp vụ nào ngoài các điều kiện chuyển trạng thái đã nêu ở FR gốc (R8) |
| BR-05.12 | Chống trùng: mỗi bộ (quy tắc, đối tượng, mốc, mức) có tối đa một nhắc đang mở; không gửi lại ngoài nhịp cấu hình |
| BR-05.13 | Lịch ngày làm việc (ngày nghỉ cuối tuần, lễ) là tham số `[CẦN LÀM RÕ VỚI KH]`; mỗi quy tắc dùng ngày làm việc hoặc ngày lịch theo cấu hình (ví dụ 5 ngày làm việc, 15 ngày lịch); hạn rơi vào ngày nghỉ áp theo quy ước đó |
| BR-05.14 | Điều kiện dừng: bước nghiệp vụ hoàn tất, dự án Hủy, hoặc đầu việc chuyển sang bước sau; khi dừng, nhắc đang mở tự đóng và ghi vết |
| BR-05.15 | Quy tắc thiếu tham số: không tính quá hạn và hiển thị cảnh báo "thiếu tham số" cho người quản trị tham số của nhóm đó; không đoán giá trị (BR-02.13, FR-06.3) |
| BR-05.16 | Người nhận không xác định hoặc nghỉ (đơn vị chưa có CSKH, người phụ trách nghỉ): người leo thang nhận trực tiếp và việc vào hàng đợi chung của phòng; hệ thống không bỏ nhắc `[BA đề xuất]` |
| BR-05.17 | Đổi tham số hạn chỉ áp cho đầu việc sinh sau ngày hiệu lực; đầu việc đang mở giữ hạn đã tính `[BA đề xuất]` |
| BR-05.18 | "Ngày bắt đầu đo" (FR-06.5): không sinh nhắc và leo thang cho đầu việc có ngày gốc trước ngày này; dự án cũ không tính vào số liệu tuân thủ (G26) |
| BR-05.19 | Mọi nhắc và leo thang ghi nhận: quy tắc, đối tượng, người nhận, thời điểm; dữ liệu này là nguồn của số liệu SLA (BR-05.33) và là thay đổi cần lưu vết (NFR-G10.1) |
| BR-05.20 | Người dùng có thể xem lý do một nhắc được sinh (quy tắc nào, ngày gốc nào, hạn nào) để kiểm chứng và phản hồi nếu sai |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-05.2.1 | Happy | BR-05.9, BR-05.10 | QN-03 | **Given** đầu việc "Hoàn tất hợp đồng" hạn 10 ngày và đã quá hạn **When** hệ thống chạy đánh giá **Then** nhắc nhân viên Kinh doanh phụ trách, leo thang Trưởng phòng Kinh doanh, ghi nhật ký và không phạt tự động |
| AC-05.2.2 | Happy | BR-05.10 | QN-06 | **Given** hóa đơn "Đang công nợ" đã đòi 3 lần chưa thu **When** đến nhịp 15 ngày kế tiếp **Then** hệ thống leo thang Trưởng phòng Kế toán và vẫn tạo việc đòi nợ cho Kế toán |
| AC-05.2.3 | Happy | BR-05.14 | R3 | **Given** nhắc hạn nộp BBNT đang mở **When** BBNT được nộp **Then** nhắc tự đóng, không có leo thang nào được gửi sau đó |
| AC-05.2.4 | Exception | BR-05.15 | QN-07 | **Given** đối tác chưa có hạn gửi BBNT cấu hình **When** dự án kênh A ở "Chờ nghiệm thu" **Then** hệ thống không tính quá hạn và cảnh báo "thiếu tham số" cho người quản trị tham số |
| AC-05.2.5 | Exception | BR-05.16 | QN-12 | **Given** đơn vị chưa có CSKH phụ trách và việc rà soát bàn giao quá hạn **When** đến hạn **Then** Trưởng phòng CSKH nhận cảnh báo trực tiếp và việc vẫn nằm trong hàng đợi chung `[BA đề xuất]` |
| AC-05.2.6 | Edge | BR-05.13 | QN-03 | **Given** hạn 10 ngày rơi vào chiều chủ nhật và quy tắc cấu hình ngày làm việc **When** hệ thống tính hạn **Then** hạn chuyển sang ngày làm việc kế tiếp theo lịch cấu hình `[CẦN LÀM RÕ VỚI KH]` |
| AC-05.2.7 | Edge | BR-05.12 | R8 | **Given** một nhắc quá hạn đã gửi hôm qua **When** hệ thống chạy lại hôm nay mà chưa đến nhịp mới **Then** không gửi nhắc trùng |
| AC-05.2.8 | Edge | BR-05.17 | NFR-G7.1 | **Given** hạn nộp BBNT đổi từ 5 sang 3 ngày làm việc hiệu lực từ 01/12 **When** có BBNT ký 28/11 (đã có đầu việc hạn cũ) và BBNT ký 02/12 **Then** đầu việc 28/11 giữ hạn 5 ngày, đầu việc 02/12 dùng hạn 3 ngày `[BA đề xuất]` |
| AC-05.2.9 | Edge | BR-05.18 | G26 | **Given** ngày bắt đầu đo là 01/12 và một dự án có BBNT ký 20/11 **When** hệ thống đánh giá quy tắc nhắc **Then** không sinh nhắc hay leo thang cho các mốc có ngày gốc trước 01/12 |
| AC-05.2.10 | Edge | BR-05.20 | BR-05.19 | **Given** Trưởng phòng nhận một leo thang **When** mở chi tiết **Then** thấy quy tắc, ngày gốc, hạn và số ngày trễ đã dùng để sinh leo thang |

---

### FR-05.3 — Thông báo
- **Mô tả**: Gửi thông báo trong ứng dụng và email cho nhắc, leo thang, giao việc và kết quả duyệt; người dùng bệnh viện nhận thông báo trạng thái yêu cầu.
- **Input – Xử lý – Output**: Input: nhắc, leo thang, sự kiện giao việc/duyệt. Xử lý: tạo thông báo, gửi kênh, theo dõi đã đọc. Output: thông báo trong ứng dụng, email.
- **Màn hình UI**: `[WIREFRAME: Trung tâm thông báo]` · `[WIREFRAME: Tùy chọn thông báo]`

**Luồng xử lý:**
- **Happy**: sự kiện → thông báo trong ứng dụng và email → người nhận mở → đánh dấu đã đọc.
- **Exception**: email lỗi; người nhận không có email.
- **Edge/Boundary**: nhiều thông báo cùng lúc; người dùng tắt thông báo thường; thông báo cho người dùng bệnh viện; nội dung nhạy cảm.
- **Alternative**: đẩy thông báo lên điện thoại (giai đoạn sau).

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-05.21 | Mọi nhắc, leo thang, giao việc mới, kết quả duyệt/từ chối tạo thông báo trong ứng dụng (khối "thông báo" của "Việc của tôi") có trạng thái đã đọc |
| BR-05.22 | Ngoài ứng dụng, MVP gửi email cho nhắc và leo thang `[BA đề xuất]`; đẩy thông báo lên điện thoại và tin nhắn để sau `[CẦN LÀM RÕ VỚI KH]` |
| BR-05.23 | Người dùng tắt được thông báo thường nhưng không tắt được nhắc quá hạn và leo thang bắt buộc |
| BR-05.24 | Người dùng bệnh viện nhận thông báo đổi trạng thái yêu cầu của chính bệnh viện mình qua cổng và email; không chứa dữ liệu của bệnh viện khác (NFR-03.1) |
| BR-05.25 | Email chỉ chứa tiêu đề, loại việc, hạn và đường dẫn đăng nhập; không chứa nội dung yêu cầu, tệp hay số liệu tài chính |
| BR-05.26 | Email gửi lỗi được thử lại có giới hạn và ghi nhận; thông báo trong ứng dụng vẫn có để không mất nhắc |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-05.3.1 | Happy | BR-05.21, BR-05.22 | R7 | **Given** Trưởng phòng Kinh doanh nhận một leo thang **When** hệ thống tạo thông báo **Then** có thông báo chưa đọc trong ứng dụng và một email có tiêu đề, loại việc, hạn, đường dẫn |
| AC-05.3.2 | Happy | BR-05.21 | R7 | **Given** người dùng có 3 thông báo chưa đọc **When** mở và đọc một thông báo **Then** thông báo đó chuyển "đã đọc" và số chưa đọc còn 2 |
| AC-05.3.3 | Exception | BR-05.26 | R7 | **Given** email gửi lỗi **When** hệ thống thử lại hết số lần cho phép **Then** ghi nhận lỗi và thông báo trong ứng dụng vẫn hiển thị |
| AC-05.3.4 | Edge | BR-05.23 | R8 | **Given** người dùng đã tắt thông báo thường **When** có nhắc quá hạn hoặc leo thang **Then** người dùng vẫn nhận thông báo |
| AC-05.3.5 | Edge | BR-05.24 | NFR-03.1 | **Given** yêu cầu của bệnh viện A đổi trạng thái **When** hệ thống thông báo **Then** chỉ người dùng của bệnh viện A nhận và nội dung không có dữ liệu bệnh viện khác |
| AC-05.3.6 | Edge | BR-05.25 | G16 | **Given** nhắc liên quan yêu cầu "xóa dữ liệu" **When** gửi email **Then** email không chứa nội dung yêu cầu hay tệp, chỉ có tiêu đề chung, hạn và đường dẫn |

---

### FR-05.4 — Bàn làm việc theo vai trò
- **Mô tả**: Mỗi vai trò có trang đầu gồm các khối phục vụ đầu việc của vai trò đó.
- **Input – Xử lý – Output**: Input: vai trò và phạm vi quyền, dữ liệu giao dịch. Xử lý: chọn khối, lọc theo phạm vi, tính số liệu. Output: bàn làm việc theo vai trò.
- **Màn hình UI**: `[WIREFRAME: Bàn làm việc Trưởng nhóm triển khai]` · `[WIREFRAME: Bàn làm việc Nhân viên Kinh doanh]` · `[WIREFRAME: Bàn làm việc PGĐ Tài chính]` · `[WIREFRAME: Bàn làm việc CSKH / Kế toán / Trưởng phòng]`

**Luồng xử lý:**
- **Happy**: người dùng đăng nhập → thấy bàn làm việc của vai trò → mở khối → thao tác.
- **Exception**: người dùng không có vai trò nào; khối không có quyền dữ liệu.
- **Edge/Boundary**: một người nhiều vai trò; vai trò chưa có bàn riêng; số liệu chưa cập nhật; dữ liệu "Cần xác minh".
- **Alternative**: người dùng ẩn bớt khối.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-05.27 | Bàn làm việc là trang đầu của vai trò; một người nhiều vai trò chọn bàn hoặc gộp khối; bố cục mặc định theo vai trò là cấu hình |
| BR-05.28 | MVP bắt buộc 3 bàn theo đề: Trưởng nhóm triển khai, Nhân viên Kinh doanh, PGĐ Tài chính (nội dung ở bảng dưới); kèm bàn tối thiểu cho CSKH, Kế toán, Trưởng phòng, PGĐ Chuyên môn/Giám đốc (việc chờ duyệt) và Phần mềm (hàng đợi yêu cầu đã duyệt) vì FR-01…04 giao việc cho các vai trò này |
| BR-05.29 | Mỗi dự án trong danh sách trên bàn làm việc hiển thị người phụ trách bước hiện tại và số ngày đã ở bước đó để trả lời "đang kẹt ở ai" (UC-01.4); số liệu tính từ dữ liệu giao dịch, không nhập tay |
| BR-05.30 | Mỗi khối hiển thị thời điểm cập nhật dữ liệu; dự án "Cần xác minh" (FR-06.5) hiển thị riêng, không lẫn vào số liệu tuân thủ |
| BR-05.31 | Khối chỉ hiển thị dữ liệu trong phạm vi quyền; khối cần dữ liệu người dùng không có quyền thì ẩn (ví dụ tỷ lệ chia sẻ với Chuyển giao, Kinh doanh) (NFR-G2.1, BR-02.21) |
| BR-05.32 | Người dùng ẩn/hiện được khối không bắt buộc; không ẩn được khối "việc quá hạn" và "thông báo" |

**Nội dung bàn làm việc của 3 vai trò bắt buộc (mở ra thấy gì đầu tiên, vì sao):**
| Vai trò | Thấy đầu tiên | Các khối chính | Vì sao |
|---|---|---|---|
| Trưởng nhóm triển khai | Việc quá hạn và dự án mình phụ trách theo trạng thái | Dự án "Chờ nghiệm thu" kèm số ngày tồn và lý do chặn; dự án sắp rút/ danh sách tồn chưa xong; việc từ xa của đội; lịch điều động | Người ghi mốc, BBNT và danh sách tồn; trễ ở đây làm đứt chuỗi KK01, KK05 |
| Nhân viên Kinh doanh | Việc "Hoàn tất hợp đồng" kèm đếm ngược 10 ngày | Hồ sơ chờ gọi xác nhận; hợp đồng gốc chưa về; đề nghị xuất hóa đơn chờ gửi; báo giá yêu cầu cần lập | Hạn 10 ngày, 1–2 ngày, 2 tuần là quy định dễ quên nhất; đếm ngược thay cho việc tự nhớ |
| PGĐ Tài chính | Việc cần duyệt (tỷ lệ, ngoại lệ, BBĐS, hóa đơn tạm ứng) và phễu doanh thu | Phễu số dự án và giá trị theo trạng thái trục doanh thu, tách kênh A/B; công nợ treo và số lần đòi; kỳ đối soát đang mở theo đối tác với số dòng lệch; BBNT đối tác chưa gửi | Là người chốt BBĐS và duyệt tham số; cần thấy chỗ kẹt tiền trước khi ra quyết định |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-05.4.1 | Happy | BR-05.27, BR-05.28 | R6 | **Given** người dùng có vai trò Trưởng nhóm triển khai **When** đăng nhập **Then** trang đầu là bàn làm việc Trưởng nhóm với khối việc quá hạn đứng đầu, rồi dự án "Chờ nghiệm thu" kèm số ngày tồn |
| AC-05.4.2 | Happy | BR-05.28 | R6 | **Given** người dùng có vai trò PGĐ Tài chính **When** đăng nhập **Then** thấy việc cần duyệt và phễu doanh thu tách kênh A/B, kỳ đối soát đang mở và công nợ treo |
| AC-05.4.3 | Happy | BR-05.29 | UC-01.4 | **Given** dự án kênh B ở "Chờ hợp đồng" quá 6 ngày, người phụ trách là Kinh doanh K **When** Trưởng phòng Kinh doanh mở bàn làm việc **Then** dòng dự án hiển thị K và "6 ngày ở bước Chờ hợp đồng" |
| AC-05.4.4 | Exception | BR-05.31 | NFR-G2.1 | **Given** người dùng là nhân viên Chuyển giao **When** mở bàn làm việc **Then** không có khối tỷ lệ chia sẻ hay giá trị đối soát |
| AC-05.4.5 | Edge | BR-05.27 | R6 | **Given** một người vừa là Trưởng phòng Chuyển giao vừa là Trưởng nhóm một dự án **When** đăng nhập **Then** chọn được bàn làm việc hoặc xem bàn gộp, không mất khối nào |
| AC-05.4.6 | Edge | BR-05.30 | G26 | **Given** 5 dự án đang mở ở trạng thái "Cần xác minh" **When** PGĐ Tài chính mở phễu **Then** 5 dự án đó hiển thị riêng, không tính vào các trạng thái trục doanh thu |
| AC-05.4.7 | Edge | BR-05.32 | R7 | **Given** người dùng muốn ẩn khối **When** thử ẩn khối "việc quá hạn" **Then** hệ thống không cho ẩn |
| AC-05.4.8 | Edge | BR-05.28 | R6 | **Given** vai trò chưa có bàn riêng trong MVP **When** người dùng đăng nhập **Then** hệ thống hiển thị bàn tối thiểu gồm "Việc của tôi" và thông báo |

---

### FR-05.5 — Số liệu SLA, báo cáo nội bộ và phễu doanh thu
- **Mô tả**: Lưu số liệu đúng hạn/trễ của từng mốc để báo cáo nội bộ; phễu doanh thu trên bàn làm việc PGĐ Tài chính.
- **Input – Xử lý – Output**: Input: ngày gốc, hạn, ngày hoàn thành, nhắc và leo thang đã gửi. Xử lý: tính số ngày trễ, tỷ lệ đúng hạn, tổng hợp theo phòng/người/kỳ, dựng phễu. Output: báo cáo tuân thủ mốc, phễu doanh thu.
- **Màn hình UI**: `[WIREFRAME: Báo cáo tuân thủ mốc]` · `[WIREFRAME: Phễu doanh thu]`

**Luồng xử lý:**
- **Happy**: mốc hoàn thành → lưu số ngày trễ → báo cáo theo phòng/kỳ → Trưởng phòng và PGĐ xem.
- **Exception**: người xem ngoài phạm vi; mốc thiếu ngày gốc.
- **Edge/Boundary**: mốc trước "ngày bắt đầu đo"; mốc đổi hạn giữa chừng; dự án Tạm dừng; dữ liệu cần xác minh.
- **Alternative**: tải báo cáo ra tệp.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-05.33 | Mỗi mốc lưu: ngày gốc, hạn đã tính, ngày hoàn thành thực tế, số ngày trễ (nếu có), người phụ trách; không dùng để phạt, chỉ để báo cáo nội bộ và cảnh báo (R8) |
| BR-05.34 | Báo cáo tuân thủ mốc tối thiểu cho MVP: tỷ lệ đúng hạn và số ngày trễ trung bình của mốc nộp BBNT (5 ngày), ký hợp đồng (10 ngày), gọi xác nhận, đòi nợ (15 ngày, 3 lần), rà soát bàn giao, theo phòng và kỳ; xem theo người trong phạm vi quyền |
| BR-05.35 | Quyền xem báo cáo: Trưởng phòng xem phòng mình; PGĐ xem phòng thuộc mảng mình; Giám đốc xem toàn công ty `[CẦN LÀM RÕ VỚI KH]` (G13 cho PGĐ Kinh doanh) |
| BR-05.36 | Phễu doanh thu trên bàn làm việc PGĐ Tài chính: số dự án theo từng trạng thái trục doanh thu, tách kênh A/B, kèm giá trị hóa đơn và công nợ; báo cáo điều hành đầy đủ thuộc FR-13 |
| BR-05.37 | Số liệu loại trừ dự án trước "ngày bắt đầu đo" và dự án "Cần xác minh"; hiển thị số loại trừ để người xem biết độ phủ dữ liệu (BR-05.18, FR-06.5) |
| BR-05.38 | Số liệu SLA là nguồn cho các chỉ số đo thành công ở Phần A.7 (ví dụ số ngày từ nghiệm thu đến xuất hóa đơn); định nghĩa chỉ số chốt ở Phần A.7, FR-05 chỉ bảo đảm dữ liệu nền |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-05.5.1 | Happy | BR-05.33 | R8 | **Given** BBNT ký 03/12, hạn nộp 5 ngày làm việc (10/12), nộp ngày 12/12 **When** mốc hoàn thành **Then** lưu số ngày trễ 2 và người phụ trách, không áp chế tài |
| AC-05.5.2 | Happy | BR-05.34 | R8 | **Given** phòng Chuyển giao có 20 mốc nộp BBNT trong tháng, 15 đúng hạn **When** Trưởng phòng mở báo cáo **Then** hiển thị tỷ lệ đúng hạn 75% và số ngày trễ trung bình của 5 mốc trễ |
| AC-05.5.3 | Happy | BR-05.36 | R6 | **Given** có dự án ở các trạng thái trục doanh thu của cả hai kênh **When** PGĐ Tài chính mở phễu **Then** thấy số dự án mỗi trạng thái tách kênh A/B và công nợ theo hóa đơn |
| AC-05.5.4 | Exception | BR-05.35 | NFR-G2.1 | **Given** Trưởng phòng Kinh doanh **When** mở báo cáo của phòng Chuyển giao **Then** hệ thống từ chối hiển thị |
| AC-05.5.5 | Edge | BR-05.37 | G26 | **Given** 30 dự án có ngày gốc trước ngày bắt đầu đo **When** mở báo cáo tuân thủ **Then** 30 dự án bị loại khỏi số liệu và báo cáo ghi "30 dự án ngoài phạm vi đo" |
| AC-05.5.6 | Edge | BR-05.33 | BR-05.17 | **Given** hạn của một đầu việc đã tính theo tham số cũ **When** tham số đổi sau đó **Then** số ngày trễ vẫn tính theo hạn đã lưu lúc sinh đầu việc |
| AC-05.5.7 | Edge | BR-05.34 | R8 | **Given** Trưởng phòng muốn xem tỷ lệ đúng hạn theo người **When** mở báo cáo **Then** chỉ thấy người trong phạm vi phòng mình |

---

## 5. Business Rule chung của tính năng
Các BR áp dụng cho toàn tính năng: **BR-05.1** (mọi bước có chủ sinh đầu việc có người phụ trách), **BR-05.9** (quy tắc nhắc là cấu hình, không code cứng), **BR-05.10** (nhắc người chủ trước, leo thang Trưởng phòng), **BR-05.11** (không phạt tự động). BR riêng từng sub-feature nằm trong từng sub; mã `BR-05.n` duy nhất trong toàn tính năng. FR-05 không sở hữu nghiệp vụ của FR-01…04; danh mục QN (mục 3.3) chỉ vận hành cơ chế.

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Khác biệt — độ tin cậy của nhắc: bỏ sót hoặc nhắc trùng làm mất lòng tin vào toàn bộ hệ thống cảnh báo (rủi ro đã nêu ở FR-01 mục 7)`.

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| NFR-05.1 | Độ tin cậy | 100% đầu việc quá hạn có quy tắc hợp lệ sinh đúng một nhắc cho đúng người trong vòng 1 ngày làm việc kể từ ngày đến hạn, và 0 nhắc trùng ngoài nhịp cấu hình. Đo bằng kiểm thử với bộ dữ liệu mẫu theo từng QN-01…QN-15 (đổi ngày hệ thống giả lập), đối chiếu số nhắc sinh ra với số mong đợi. *(nguồn: R7, R8; BA đề xuất — ngưỡng "1 ngày làm việc" suy từ đơn vị ngày của các quy định, chờ KH xác nhận)* | NFR chung không có nhóm độ tin cậy nào; ở đây thành tiêu chí đo được cho riêng cơ chế nhắc |

## 6b. Phân quyền
> Action: V xem · C tạo · E sửa · A duyệt/xác nhận · X thực hiện. Phạm vi dữ liệu chi tiết do FR-06.

| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-05.1 Xem "Việc của tôi" | Mọi người dùng nội bộ | V | Đầu việc của chính mình và trong phạm vi quyền |
| FR-05.1 Đổi người phụ trách đầu việc | Trưởng phòng của người phụ trách | E | Lưu vết |
| FR-05.1 Ghi kết quả đầu việc | Người phụ trách | E, X | Đầu việc của mình |
| FR-05.2 Xem cảnh báo leo thang | Trưởng phòng (phòng mình), PGĐ, Giám đốc | V | Theo quy tắc QN |
| FR-05.2 Cấu hình quy tắc nhắc (QN) | Người quản trị tham số theo nhóm (chi tiết FR-06.3) | C, E | Theo quy trình duyệt tham số; IT nội bộ chỉ quản trị khung |
| FR-05.3 Nhận và quản lý thông báo | Mọi người dùng | V, E | Chỉ tắt được thông báo thường |
| FR-05.3 Nhận thông báo yêu cầu | Người dùng bệnh viện, IT bệnh viện | V | Chỉ bệnh viện mình (NFR-03.1) |
| FR-05.4 Xem bàn làm việc | Mọi người dùng theo vai trò | V | Phạm vi quyền (NFR-G2.1) |
| FR-05.4 Ẩn/hiện khối, cấu hình bố cục mặc định | Người dùng (khối cá nhân); người quản trị tham số (bố cục mặc định) | E | Không ẩn khối bắt buộc |
| FR-05.5 Xem báo cáo tuân thủ mốc | Trưởng phòng, PGĐ, Giám đốc | V | BR-05.35 `[CẦN LÀM RÕ VỚI KH]` |
| FR-05.5 Xem phễu doanh thu | PGĐ Tài chính, Giám đốc | V | — |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| Người dùng không mở hệ thống nên không thấy nhắc | D3 không có hiệu lực, mốc vẫn trễ | Email nhắc và leo thang (BR-05.22); bàn làm việc là trang đầu |
| Quá nhiều nhắc/leo thang → người dùng bỏ qua | Mất lòng tin vào cảnh báo | Chống trùng (BR-05.12), "ngày bắt đầu đo" (BR-05.18), NFR-05.1 |
| Dữ liệu mốc không được cập nhật | Hạn tính sai, nhắc sai | Bước chặn ở FR-01…04; đầu việc chỉ hoàn thành khi bước xong (BR-05.4); chỉ số độ phủ (BR-05.37) |
| Người dùng sợ bị "chấm điểm" từ số liệu SLA | Né ghi nhận mốc thật | Cam kết R8 không phạt; quyền xem báo cáo hạn chế (BR-05.35); giải thích rõ lý do nhắc (BR-05.20) |
| Lịch ngày làm việc/lễ chưa chốt | Hạn tính lệch 1–2 ngày | Lịch là tham số, hỏi KH |
| Nhiều tham số hạn còn `[CẦN LÀM RÕ VỚI KH]` | Một số QN chưa chạy được | Quy tắc thiếu tham số không tính quá hạn, chỉ cảnh báo (BR-05.15) |
| Phụ thuộc FR-01…04 (nguồn sự kiện) | Không có sự kiện thì không có nhắc | Phát hành cùng trong MVP |
| Phụ thuộc FR-06 (tham số, cơ cấu leo thang, quyền, vết, ngày bắt đầu đo) | Không cấu hình được QN và không biết ai leo thang cho ai | Phát hành cùng FR-06 |
| Hiệu năng khi số đầu việc lớn | Trang chậm | NFR hiệu năng chưa có số liệu từ KH (G16) |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-05.1 và FR-05.3: "Việc của tôi", thông báo; chạy thử với nhóm Kinh doanh và Chuyển giao | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-05.2: danh mục QN-01…QN-15 với người chủ mốc và Trưởng phòng; xác nhận hạn, loại ngày, lịch nghỉ | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-05.4 và FR-05.5: ba bàn làm việc bắt buộc, báo cáo tuân thủ mốc, phễu PGĐ Tài chính | Chưa bắt đầu |

## 9. Traceability (mini)
> Cột Model, Wireframe, Test để các bước sau điền (⏳).

{{TRACE}}

**NFR liên quan:**
| Mã NFR | Nguồn | NFR Test Scenario |
|---|---|---|
| NFR-05.1 | Mục 6 tính năng này (R7, R8) | ⏳ |
| NFR-G2.1 | Backbone mục 3 (R4) | ⏳ |
| NFR-G7.1 | Backbone mục 3 (R5) | ⏳ |
| NFR-G8.1 | Backbone mục 3 (R1) | ⏳ |
| NFR-G10.1 | Backbone mục 3 (R4) | ⏳ |

## 10. Câu hỏi cần làm rõ với KH (tổng hợp)
| STT | Vị trí trong spec | Câu hỏi cần làm rõ | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | BR-05.13 | Ngày làm việc của công ty (thứ bảy, lễ) và quy định nào tính ngày lịch, quy định nào tính ngày làm việc? | Nice-to-know | Chưa hỏi |
| 2 | BR-05.22 | Kênh thông báo mong muốn ngoài ứng dụng (email, tin nhắn, đẩy trên điện thoại)? | Nice-to-know | Chưa hỏi |
| 3 | BR-05.35 | Quyền xem báo cáo tuân thủ mốc của PGĐ Kinh doanh và PGĐ Chuyên môn; có hiển thị theo từng người không? (G13) | Nice-to-know | Chưa hỏi |
| 4 | QN-01, QN-08…QN-12, QN-11 | Các hạn chưa nêu trong đề: Chờ nghiệm thu, chốt BBĐS, phản hồi của IT bệnh viện, khách xác nhận, SLA xử lý yêu cầu (G14), rà soát bàn giao | Nice-to-know | Chưa hỏi |
| 5 | BR-05.8 | Khi dự án Tạm dừng, các hạn có tiếp tục chạy không? | Nice-to-know | Chưa hỏi |
| 6 | Mục 7 | Số người dùng đồng thời và thời gian phản hồi mong muốn của trang "Việc của tôi" (G16) | Nice-to-know | Chưa hỏi |
| 7 | BR-05.16, BR-05.17, BR-05.22, NFR-05.1 | Xác nhận các bổ sung `[BA đề xuất]` (người nhận vắng thì leo thang nhận trực tiếp, đổi tham số chỉ áp cho việc mới, email nhắc, ngưỡng "1 ngày làm việc" cho nhắc) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1, Phần A; danh mục QN-01…QN-15 tổng hợp từ FR-01…04 | — | Draft |
