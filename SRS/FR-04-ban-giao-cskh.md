# Feature Spec: Bàn giao Triển khai → CSKH kèm danh sách việc tồn, việc từ xa

**Mã**: FR-04  
**Loại**: Lớn  
**Effort**: Chưa ước lượng  
**Thuộc Backbone**: ERP-CongTyA (mục 4, dòng FR-04)  
**Trạng thái**: Draft  
**Người phụ trách**: PO/BA  
**Approach**: UC-driven (+US ánh xạ)  
**User Story liên quan**: US-14, US-15  
**Khó khăn nguồn**: KK05 · **Rx nguồn**: R3, R18, R29 · **Giả định liên quan**: G09, G10, G12, G20, G21

> **Quy ước:** `Rx` là mã của Backbone mục 5.1. `[BA đề xuất]` là bổ sung chưa nằm trong Backbone v1.1, chờ xác nhận (tổng hợp ở mục 10, dòng cuối). Tên trạng thái đúng với Phần A mục A.4a (trục *Triển khai & bàn giao*). Cột "Nguồn" của AC ghi US/UC; số thứ tự kịch bản `A1…` của US ở Phần B.2 sẽ đối chiếu lại khi chốt.

---

## 1. Mục đích
Không để đội triển khai rút đi mà việc dở không có danh sách. Một dự án chỉ sang "Đã bàn giao - bảo hành" khi Trưởng nhóm đã lập danh sách việc tồn (kể cả việc từ xa, hoặc xác nhận "không còn tồn") và CSKH đã xác nhận nhận. Mỗi việc tồn trở thành đầu việc có người phụ trách và hạn, nên CSKH biết mình nhận những gì và khách không còn phải phàn nàn vì việc bị bỏ quên. Tính năng cũng quản lý toàn bộ trục *Triển khai & bàn giao* của Dự án (điều động tối thiểu, tạm dừng, hủy, đóng) vì trục này là phần còn lại của D2.

## 2. Input – Xử lý – Output (tổng quan)
| | Mô tả |
|---|---|
| Input | Điều động tối thiểu (Trưởng nhóm, nhân sự, vai trò, từ ngày); mốc Bắt đầu và Rút; danh sách việc tồn (mô tả, hình thức tại đơn vị/từ xa, người phụ trách dự kiến, hạn); xác nhận "không còn tồn"; kết quả rà soát của CSKH (nhận/trả lại, lý do); cập nhật tiến độ và kết quả việc tồn; lý do tạm dừng/hủy |
| Xử lý | Chuyển trạng thái trục triển khai; kiểm tra điều kiện bàn giao; giao việc rà soát cho CSKH; biến việc tồn thành đầu việc; đưa việc từ xa còn mở vào danh sách; nhắc và leo thang khi quá hạn (cơ chế chung FR-05); kiểm tra điều kiện đóng dự án; lưu vết |
| Output | Dự án ở đúng trạng thái trục triển khai; danh sách việc tồn và lịch sử trả lại/bổ sung; đầu việc của CSKH trong "Việc của tôi"; chỉ số tồn đọng theo đơn vị; cảnh báo quá hạn; nhật ký lưu vết |

Sơ đồ: `[MODEL: Sequence]` Bàn giao Triển khai → CSKH (Hình A.3.3) · `[MODEL: State]` Dự án — trục Triển khai & bàn giao (Hình A.4a) · `[MODEL: ERD]` `DU_AN`, `DIEU_DONG`, `DAU_VIEC`, `NHAN_SU` (Hình A.5)

## 3. Use Case chi tiết

### UC-04.1 — Chuẩn bị rút: lập danh sách việc tồn và ghi mốc Rút
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Trưởng nhóm lập danh sách việc tồn (hoặc xác nhận không còn tồn) rồi ghi mốc Rút; hệ thống giao việc rà soát cho CSKH |
| **Actor chính** | Trưởng nhóm triển khai |
| **Actor phụ** | CSKH, Trưởng phòng Chuyển giao, Hệ thống |
| **Trigger** | Đội chuẩn bị rút khỏi đơn vị |
| **Tiền điều kiện** | Dự án trục triển khai "Đang triển khai" |
| **Hậu điều kiện thành công** | Danh sách tồn được lưu; mốc Rút ghi kèm người ghi và thời điểm; dự án "Đã rút - chờ bàn giao"; việc "Rà soát bàn giao" giao CSKH |
| **Hậu điều kiện thất bại** | Không đổi trạng thái, không ghi mốc Rút |
| **Business Rule liên quan** | BR-04.1 đến BR-04.7 |
| **Sub-feature liên quan** | FR-04.1 |

**Luồng chính (Happy):**
1. Trưởng nhóm mở dự án và chọn "Chuẩn bị rút". → 2. Hệ thống hiển thị danh sách tồn ở trạng thái Nháp, đã gồm sẵn các việc từ xa còn mở. → 3. Trưởng nhóm thêm từng việc tồn (mô tả, hình thức, người phụ trách dự kiến, hạn), hoặc bấm "Không còn tồn". → 4. Trưởng nhóm ghi mốc Rút (ngày rút). → 5. Hệ thống kiểm tra điều kiện, chuyển trục triển khai sang *Đã rút - chờ bàn giao*, gửi danh sách cho CSKH và tạo việc "Rà soát bàn giao".

**Luồng phụ (Alternative):**
- 1a. Trưởng nhóm đã đổi giữa chừng: người mới tiếp quản danh sách Nháp, sự kiện được ghi (BR-01.8).
- 3a. Việc từ xa sẽ tiếp tục sau khi rút: ghi như đầu việc thường với hình thức "từ xa", có người phụ trách và hạn (G12).
- 5a. Đội quay lại đơn vị sau khi đã rút: Trưởng phòng Chuyển giao mở lại về "Đang triển khai" kèm lý do `[BA đề xuất]`.

**Luồng ngoại lệ (Exception):**
- E1. (bước 4) Chưa có danh sách và chưa xác nhận "không còn tồn" → chặn ghi Rút.
- E2. (bước 3) Việc tồn thiếu mô tả, hình thức, người phụ trách dự kiến hoặc hạn → chặn lưu việc đó.
- E3. (bước 4) Dự án đang "Tạm dừng" → không ghi Rút.
- E4. (bước 4) Ngày Rút trước ngày Bắt đầu triển khai hoặc ở tương lai → chặn.

### UC-04.2 — Rà soát và xác nhận nhận bàn giao
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | CSKH rà soát danh sách tồn, xác nhận nhận hoặc trả lại kèm lý do |
| **Actor chính** | CSKH (phụ trách đơn vị) |
| **Actor phụ** | Trưởng nhóm triển khai, Trưởng phòng Chuyển giao, Trưởng phòng CSKH, Hệ thống |
| **Trigger** | Dự án ở "Đã rút - chờ bàn giao" và việc rà soát được giao |
| **Tiền điều kiện** | Danh sách tồn đã gửi (có thể rỗng đã xác nhận) |
| **Hậu điều kiện thành công** | Dự án "Đã bàn giao - bảo hành"; mỗi việc tồn là đầu việc có người phụ trách và hạn |
| **Hậu điều kiện thất bại** | Dự án giữ "Đã rút - chờ bàn giao"; Trưởng nhóm được yêu cầu bổ sung |
| **Business Rule liên quan** | BR-04.8 đến BR-04.14 |
| **Sub-feature liên quan** | FR-04.2 |

**Luồng chính (Happy):**
1. CSKH mở việc "Rà soát bàn giao" và xem danh sách tồn. → 2. CSKH đối chiếu với tình trạng thực tế tại đơn vị. → 3. CSKH xác nhận nhận. → 4. Hệ thống chuyển trục triển khai sang *Đã bàn giao - bảo hành*, biến mỗi việc tồn thành đầu việc của CSKH (hoặc người dự kiến) có hạn, hiện trong "Việc của tôi" và ghi vết.

**Luồng phụ (Alternative):**
- 3a. Danh sách chưa đủ: CSKH trả lại kèm lý do; hệ thống yêu cầu Trưởng nhóm bổ sung và báo Trưởng phòng Chuyển giao; sau khi bổ sung, quay lại bước 1 (G12).
- 3b. Danh sách rỗng ("không còn tồn"): CSKH vẫn phải rà soát và xác nhận nhận.
- 3c. Việc tồn là lỗi hoặc yêu cầu thay đổi cần Phần mềm: CSKH tạo Yêu cầu nguồn "nhân viên triển khai" qua cổng FR-03 và liên kết với việc tồn `[BA đề xuất]`.

**Luồng ngoại lệ (Exception):**
- E1. (bước 3a) Trả lại không nêu lý do → chặn.
- E2. (bước 1) Quá hạn rà soát (tham số) → nhắc CSKH, leo thang Trưởng phòng CSKH, Trưởng phòng Chuyển giao thấy tình trạng `[BA đề xuất]`.
- E3. (bước 3) Trưởng nhóm cố tự xác nhận nhận → chặn (tách người lập và người nhận).
- E4. (bước 1) Đơn vị chưa có CSKH phụ trách → việc vào hàng đợi chung, Trưởng phòng CSKH phân công `[BA đề xuất]`.

### UC-04.3 — Xử lý việc tồn và theo dõi việc từ xa
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Người phụ trách xử lý đầu việc tồn/việc từ xa đến khi xong; hệ thống nhắc khi quá hạn |
| **Actor chính** | CSKH, nhân viên triển khai (việc từ xa) |
| **Actor phụ** | Trưởng nhóm, Trưởng phòng, Hệ thống |
| **Trigger** | Đầu việc được giao hoặc đến hạn |
| **Tiền điều kiện** | Dự án từ "Đang triển khai" đến "Đã bàn giao - bảo hành" |
| **Hậu điều kiện thành công** | Đầu việc "Hoàn thành" kèm kết quả; số tồn đọng của đơn vị giảm |
| **Hậu điều kiện thất bại** | Đầu việc vẫn mở; cảnh báo và leo thang được phát |
| **Business Rule liên quan** | BR-04.15 đến BR-04.19 |
| **Sub-feature liên quan** | FR-04.3 |

**Luồng chính (Happy):**
1. Đầu việc (tồn hoặc từ xa) xuất hiện trong "Việc của tôi" của người phụ trách. → 2. Người phụ trách xử lý và cập nhật tiến độ. → 3. Hoàn thành, ghi kết quả. → 4. Hệ thống cập nhật số tồn đọng của đơn vị.

**Luồng phụ (Alternative):**
- 2a. Đổi người phụ trách (người nghỉ, quá tải): Trưởng phòng đổi, có lưu vết.
- 3a. Việc không còn cần: hủy kèm lý do, người có quyền thực hiện.

**Luồng ngoại lệ (Exception):**
- E1. (bước 2) Quá hạn → nhắc người phụ trách, leo thang Trưởng phòng (cơ chế FR-05).
- E2. (bước 3) Hoàn thành không ghi kết quả → chặn.

### UC-04.4 — Vòng đời trục Triển khai & bàn giao
| Trường | Nội dung |
|---|---|
| **Mô tả ngắn** | Điều động tối thiểu, ghi mốc Bắt đầu, tạm dừng/tiếp tục, hủy, đóng dự án |
| **Actor chính** | Trưởng phòng Chuyển giao, Trưởng nhóm, CSKH |
| **Actor phụ** | Hệ thống |
| **Trigger** | Dự án được tạo (FR-01.1 hoặc FR-02.1) |
| **Tiền điều kiện** | Dự án ở "Mới tạo" |
| **Hậu điều kiện thành công** | Trục triển khai chuyển đúng theo A.4a; dự án "Đóng" khi đủ điều kiện |
| **Hậu điều kiện thất bại** | Giữ trạng thái; nêu điều kiện còn thiếu |
| **Business Rule liên quan** | BR-04.20 đến BR-04.24 |
| **Sub-feature liên quan** | FR-04.4 |

**Luồng chính (Happy):**
1. Trưởng phòng Chuyển giao cử Trưởng nhóm và nhân sự (vai trò, từ ngày). → 2. Hệ thống chuyển "Đã điều động". → 3. Trưởng nhóm ghi mốc Bắt đầu triển khai; hệ thống chuyển "Đang triển khai". → 4. (Theo UC-04.1, UC-04.2) dự án sang "Đã bàn giao - bảo hành". → 5. Khi hết bảo hành, trục doanh thu hoàn tất và không còn việc tồn mở, hệ thống tạo việc "Xác nhận đóng dự án" cho CSKH; CSKH xác nhận; dự án "Đóng".

**Luồng phụ (Alternative):**
- 3a. Tạm dừng có lý do rồi tiếp tục.
- 1a. Hủy khi dự án chưa triển khai (ở "Mới tạo" hoặc "Đã điều động") kèm lý do.

**Luồng ngoại lệ (Exception):**
- E1. (bước 3) Ghi mốc Bắt đầu khi chưa có điều động → chặn.
- E2. (bước 5) Xác nhận đóng khi chưa đủ điều kiện → chặn, nêu điều kiện còn thiếu.
- E3. Hủy khi dự án đã "Đang triển khai" → chặn; dùng Tạm dừng.

## 4. Phân rã Sub-features
| Mã sub | Mã Atom (Rx) | Tên sub-feature | Effort | Trạng thái | Ngày confirm |
|---|---|---|---|---|---|
| FR-04.1 | R3, R29 | Danh sách việc tồn và ghi mốc Rút | Chưa ước lượng | Draft | |
| FR-04.2 | R3, R29 | Rà soát và xác nhận nhận bàn giao | Chưa ước lượng | Draft | |
| FR-04.3 | R29 | Việc từ xa và theo dõi tồn sau bàn giao | Chưa ước lượng | Draft | |
| FR-04.4 | R3, R18 | Vòng đời trục Triển khai & bàn giao (điều động tối thiểu, tạm dừng, hủy, đóng) | Chưa ước lượng | Draft | |

Nhắc, leo thang và "Việc của tôi" dùng cơ chế chung ở FR-05; vai trò, phân quyền, tham số, lưu vết ở FR-06. Điều động đầy đủ (kiểm tra trùng lịch, lệnh công tác, nghỉ bù) thuộc FR-08 (Phát hành 3); MVP chỉ thu dữ liệu điều động tối thiểu.

---

### FR-04.1 — Danh sách việc tồn và ghi mốc Rút
- **Mô tả**: Lập danh sách việc tồn (kể cả việc từ xa) hoặc xác nhận không còn tồn; ghi mốc Rút; giao việc rà soát cho CSKH.
- **Input – Xử lý – Output**: Input: việc tồn, xác nhận "không còn tồn", ngày Rút. Xử lý: kiểm tra điều kiện, gom việc từ xa còn mở, chuyển trạng thái, giao việc. Output: dự án "Đã rút - chờ bàn giao", danh sách tồn gửi CSKH.
- **Màn hình UI**: `[WIREFRAME: Chuẩn bị rút và danh sách việc tồn]` · `[WIREFRAME: Ghi mốc Rút]`

**Luồng xử lý:**
- **Happy**: Trưởng nhóm lập danh sách (hoặc xác nhận không còn tồn) → ghi Rút → "Đã rút - chờ bàn giao" → CSKH được giao rà soát.
- **Exception**: chưa có danh sách; việc tồn thiếu trường; dự án Tạm dừng; ngày Rút không hợp lệ.
- **Edge/Boundary**: Rút trước hoặc sau BBNT (G21); đổi trưởng nhóm khi đang lập danh sách; việc từ xa còn mở; đội quay lại sau rút.
- **Alternative**: danh sách rỗng có xác nhận.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-04.1 | Danh sách việc tồn là điều kiện bắt buộc để ghi mốc Rút; danh sách được phép rỗng nhưng Trưởng nhóm phải xác nhận "không còn tồn" (G12) |
| BR-04.2 | Mỗi việc tồn bắt buộc: mô tả, hình thức (tại đơn vị / từ xa), người phụ trách dự kiến, hạn; có thể kèm tệp và ghi chú |
| BR-04.3 | Chỉ ghi mốc Rút khi dự án "Đang triển khai" và thỏa BR-04.1; ngày Rút không trước ngày Bắt đầu và không ở tương lai; ghi người ghi, thời điểm; sửa mốc lưu vết trước/sau (G09, BR-01.6, NFR-G10.1) |
| BR-04.4 | Ghi mốc Rút chuyển trục triển khai sang "Đã rút - chờ bàn giao", gửi danh sách cho CSKH và tạo việc "Rà soát bàn giao" cho CSKH phụ trách đơn vị, hạn là tham số `[CẦN LÀM RÕ VỚI KH]` |
| BR-04.5 | Việc từ xa còn mở tại thời điểm Rút tự động đưa vào danh sách tồn; Trưởng nhóm không gỡ được, chỉ hoàn thành hoặc hủy (có lý do) trước khi Rút |
| BR-04.6 | Đổi Trưởng nhóm khi danh sách đang Nháp: người mới tiếp quản và sửa được; sự kiện đổi lưu vết (BR-01.8) |
| BR-04.7 | `[BA đề xuất]` Đội quay lại sau khi đã Rút (chưa bàn giao xong): Trưởng phòng Chuyển giao mở lại về "Đang triển khai" kèm lý do; mốc Rút cũ giữ trong vết, danh sách tồn cũ giữ nguyên; ghi mốc Rút mới khi rút lần cuối |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-04.1.1 | Happy | BR-04.1, BR-04.3, BR-04.4 | US-14 / UC-04.1 | **Given** dự án "Đang triển khai" và Trưởng nhóm đã lập danh sách gồm 2 việc tồn đủ trường **When** ghi mốc Rút **Then** trục triển khai chuyển "Đã rút - chờ bàn giao", mốc Rút lưu người ghi và thời điểm, và hệ thống tạo việc "Rà soát bàn giao" cho CSKH |
| AC-04.1.2 | Happy | BR-04.1 | US-14 / UC-04.1 | **Given** không còn việc tồn **When** Trưởng nhóm bấm "Không còn tồn" rồi ghi mốc Rút **Then** hệ thống chấp nhận, ghi xác nhận kèm người và thời điểm, và vẫn giao việc rà soát cho CSKH |
| AC-04.1.3 | Exception | BR-04.1 | US-14 / UC-04.1 E1 | **Given** chưa có việc tồn nào và chưa xác nhận "không còn tồn" **When** ghi mốc Rút **Then** hệ thống chặn và nêu "thiếu danh sách việc tồn" |
| AC-04.1.4 | Exception | BR-04.2 | UC-04.1 E2 | **Given** một việc tồn thiếu hạn hoặc người phụ trách dự kiến **When** lưu **Then** hệ thống chặn và nêu trường thiếu |
| AC-04.1.5 | Exception | BR-04.3 | UC-04.1 E3 | **Given** dự án đang "Tạm dừng" **When** ghi mốc Rút **Then** hệ thống không cho ghi và nêu lý do |
| AC-04.1.6 | Exception | BR-04.3 | UC-04.1 E4 | **Given** ngày Rút trước ngày Bắt đầu triển khai hoặc ở tương lai **When** lưu **Then** hệ thống chặn |
| AC-04.1.7 | Edge | BR-04.5 | US-14 / UC-04.1 | **Given** dự án có 2 việc từ xa còn mở **When** Trưởng nhóm mở "Chuẩn bị rút" **Then** 2 việc đó đã nằm sẵn trong danh sách tồn và không có nút gỡ |
| AC-04.1.8 | Edge | BR-04.6 | UC-04.1 | **Given** Trưởng phòng Chuyển giao đổi Trưởng nhóm khi danh sách đang Nháp **When** Trưởng nhóm mới mở dự án **Then** thấy và sửa được danh sách Nháp; sự kiện đổi (thời điểm, người cũ, người mới) được ghi vết |
| AC-04.1.9 | Edge | BR-04.3 | UC-04.1 | **Given** dự án chưa có BBNT (trục doanh thu "Chờ nghiệm thu") **When** ghi mốc Rút **Then** trục triển khai chuyển bình thường và trục doanh thu giữ nguyên, hạn "Chờ nghiệm thu" vẫn tiếp tục chạy (G21) |
| AC-04.1.10 | Edge | BR-04.7 | UC-04.1 | **Given** dự án "Đã rút - chờ bàn giao" nhưng đội phải quay lại **When** Trưởng phòng Chuyển giao mở lại kèm lý do **Then** dự án về "Đang triển khai", mốc Rút cũ giữ trong vết và việc rà soát đang mở bị hủy `[BA đề xuất]` |

---

### FR-04.2 — Rà soát và xác nhận nhận bàn giao
- **Mô tả**: CSKH rà soát, xác nhận nhận hoặc trả lại; việc tồn thành đầu việc có người phụ trách và hạn.
- **Input – Xử lý – Output**: Input: kết quả rà soát, lý do trả lại. Xử lý: chuyển trạng thái, sinh đầu việc, nhắc quá hạn. Output: "Đã bàn giao - bảo hành" hoặc yêu cầu bổ sung.
- **Màn hình UI**: `[WIREFRAME: CSKH rà soát bàn giao]` · `[WIREFRAME: Trả lại và bổ sung danh sách]`

**Luồng xử lý:**
- **Happy**: CSKH rà soát đủ → xác nhận nhận → "Đã bàn giao - bảo hành" → việc tồn thành đầu việc.
- **Exception**: trả lại không lý do; quá hạn rà soát; Trưởng nhóm tự xác nhận.
- **Edge/Boundary**: trả lại nhiều vòng; đơn vị chưa có CSKH; việc tồn cần Phần mềm; danh sách rỗng.
- **Alternative**: trả lại rồi bổ sung.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-04.8 | Dự án chỉ sang "Đã bàn giao - bảo hành" khi CSKH xác nhận nhận; không có đường tắt cho Trưởng nhóm hay Trưởng phòng Chuyển giao (G12, KK05) |
| BR-04.9 | CSKH được nhận hoặc trả lại; trả lại bắt buộc lý do; dự án giữ "Đã rút - chờ bàn giao"; hệ thống yêu cầu Trưởng nhóm bổ sung và báo Trưởng phòng Chuyển giao mỗi lần trả lại; mọi vòng trả lại/bổ sung giữ trong lịch sử (G12) |
| BR-04.10 | Khi CSKH nhận, mỗi việc tồn tạo thành một đầu việc có người phụ trách (CSKH hoặc người dự kiến), hạn và hình thức; hiện trong "Việc của tôi" của người đó (FR-05) |
| BR-04.11 | Quá hạn rà soát (tham số `[CẦN LÀM RÕ VỚI KH]`) → nhắc CSKH phụ trách; leo thang Trưởng phòng CSKH; Trưởng phòng Chuyển giao thấy tình trạng; ghi vết; không phạt tự động (D3, R8) `[BA đề xuất]` phần leo thang Trưởng phòng CSKH |
| BR-04.12 | Người lập danh sách (Trưởng nhóm) không được xác nhận nhận; chỉ vai trò CSKH được xác nhận nhận (R4) |
| BR-04.13 | `[BA đề xuất]` Đơn vị chưa có CSKH phụ trách: việc rà soát vào hàng đợi chung của phòng CSKH; Trưởng phòng CSKH phân công; chưa phân công trong hạn thì áp BR-04.11 |
| BR-04.14 | `[BA đề xuất]` Việc tồn là lỗi hoặc yêu cầu thay đổi cần Phần mềm: CSKH tạo Yêu cầu nguồn "nhân viên triển khai" qua cổng FR-03 (BR-03.1) và liên kết với đầu việc tồn; không giao Phần mềm ngoài luồng (R3) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-04.2.1 | Happy | BR-04.8, BR-04.10 | US-15 / UC-04.2 | **Given** dự án "Đã rút - chờ bàn giao" với danh sách 3 việc tồn **When** CSKH xác nhận nhận **Then** dự án chuyển "Đã bàn giao - bảo hành" và 3 đầu việc có người phụ trách, hạn, hình thức hiện trong "Việc của tôi" |
| AC-04.2.2 | Happy | BR-04.8 | US-15 / UC-04.2 | **Given** danh sách "không còn tồn" **When** CSKH xác nhận nhận **Then** dự án chuyển "Đã bàn giao - bảo hành" và không sinh đầu việc tồn nào |
| AC-04.2.3 | Exception | BR-04.9 | US-15 / UC-04.2 E1 | **Given** CSKH chọn trả lại **When** không nhập lý do **Then** hệ thống chặn |
| AC-04.2.4 | Happy | BR-04.9 | US-15 / UC-04.2 | **Given** CSKH trả lại kèm lý do "thiếu việc từ xa module X" **When** lưu **Then** dự án giữ "Đã rút - chờ bàn giao", Trưởng nhóm nhận yêu cầu bổ sung, Trưởng phòng Chuyển giao được báo và lý do ghi vào lịch sử |
| AC-04.2.5 | Exception | BR-04.11 | UC-04.2 E2 | **Given** quá hạn rà soát mà CSKH chưa xử lý **When** đến hạn **Then** hệ thống nhắc CSKH phụ trách, báo Trưởng phòng CSKH, hiển thị cho Trưởng phòng Chuyển giao và ghi vết `[BA đề xuất]` |
| AC-04.2.6 | Exception | BR-04.12 | UC-04.2 E3 | **Given** người dùng là Trưởng nhóm đã lập danh sách **When** bấm xác nhận nhận **Then** hệ thống chặn và nêu "chỉ CSKH được xác nhận nhận" |
| AC-04.2.7 | Edge | BR-04.13 | UC-04.2 E4 | **Given** đơn vị chưa có CSKH phụ trách **When** việc rà soát được tạo **Then** việc vào hàng đợi chung và Trưởng phòng CSKH thấy để phân công `[BA đề xuất]` |
| AC-04.2.8 | Edge | BR-04.14 | UC-04.2 | **Given** một việc tồn là lỗi cần Phần mềm sửa **When** CSKH nhận bàn giao **Then** CSKH tạo được Yêu cầu nguồn "nhân viên triển khai" liên kết với việc tồn và yêu cầu đi qua các cổng FR-03 `[BA đề xuất]` |
| AC-04.2.9 | Edge | BR-04.9 | UC-04.2 | **Given** danh sách đã bị trả lại hai lần **When** xem chi tiết bàn giao **Then** hiển thị đủ hai lần trả lại (lý do, người, thời điểm) và hai lần bổ sung |

---

### FR-04.3 — Việc từ xa và theo dõi tồn sau bàn giao
- **Mô tả**: Quản lý việc từ xa như đầu việc thường; theo dõi tồn đọng của đơn vị đến khi hết.
- **Input – Xử lý – Output**: Input: việc từ xa, tiến độ, kết quả, đổi người, lý do hủy. Xử lý: sinh đầu việc, nhắc quá hạn, tính tồn đọng. Output: đầu việc trong "Việc của tôi", số tồn đọng theo đơn vị.
- **Màn hình UI**: `[WIREFRAME: Tạo việc từ xa]` · `[WIREFRAME: Tồn đọng của đơn vị]`

**Luồng xử lý:**
- **Happy**: tạo việc từ xa → xử lý → ghi kết quả → hoàn thành.
- **Exception**: hoàn thành không kết quả; quá hạn; tạo việc từ xa khi dự án đã Đóng.
- **Edge/Boundary**: người phụ trách nghỉ; hủy việc; việc từ xa tạo sau bàn giao; nhiều việc cùng hạn.
- **Alternative**: đổi người phụ trách.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-04.15 | Việc từ xa là đầu việc thường với hình thức "từ xa", có người phụ trách và hạn; tạo được từ khi dự án "Đang triển khai" đến "Đã bàn giao - bảo hành"; hiện trong "Việc của tôi" (G12) |
| BR-04.16 | Hoàn thành đầu việc tồn/việc từ xa bắt buộc ghi kết quả (ghi chú hoặc tệp) |
| BR-04.17 | Quá hạn → nhắc người phụ trách và leo thang Trưởng phòng của người đó; không phạt tự động (D3, R8; cơ chế FR-05, QN-14) |
| BR-04.18 | Đổi người phụ trách hoặc hủy việc (bắt buộc lý do) do Trưởng phòng của người phụ trách hoặc CSKH phụ trách đơn vị thực hiện; lưu vết; người nghỉ thì việc phải chuyển trước khi ngừng tài khoản (FR-06.1) |
| BR-04.19 | Mỗi đơn vị (khách hàng, dự án) có chỉ số tồn đọng: số đầu việc tồn còn mở, tuổi lâu nhất, số quá hạn; hiển thị trên bàn làm việc CSKH và Trưởng phòng (FR-05) |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-04.3.1 | Happy | BR-04.15 | US-14 / UC-04.3 | **Given** dự án "Đang triển khai" **When** nhân viên triển khai tạo việc hình thức "từ xa" với người phụ trách và hạn **Then** việc hiện trong "Việc của tôi" của người phụ trách và có thể xem theo dự án |
| AC-04.3.2 | Exception | BR-04.16 | UC-04.3 E2 | **Given** đầu việc đang xử lý **When** bấm hoàn thành mà chưa ghi kết quả **Then** hệ thống chặn |
| AC-04.3.3 | Exception | BR-04.17 | UC-04.3 E1 | **Given** đầu việc tồn quá hạn **When** đến hạn **Then** hệ thống nhắc người phụ trách, báo Trưởng phòng của người đó và ghi vết, không phạt tự động |
| AC-04.3.4 | Edge | BR-04.18 | UC-04.3 | **Given** người phụ trách nghỉ việc **When** Trưởng phòng đổi sang người khác **Then** đầu việc chuyển sang "Việc của tôi" của người mới và sự kiện đổi được ghi vết |
| AC-04.3.5 | Edge | BR-04.18 | UC-04.3 | **Given** việc tồn không còn cần **When** CSKH hủy không nhập lý do **Then** hệ thống chặn; có lý do thì hủy và ghi vết |
| AC-04.3.6 | Edge | BR-04.19 | UC-04.3 | **Given** đơn vị có 4 đầu việc tồn mở, 1 quá hạn **When** CSKH mở bàn làm việc **Then** hiển thị số tồn 4, số quá hạn 1 và tuổi lâu nhất |
| AC-04.3.7 | Edge | BR-04.15 | UC-04.3 | **Given** dự án đã "Đóng" **When** người dùng tạo việc từ xa **Then** hệ thống chặn và gợi ý tạo Yêu cầu qua FR-03 `[BA đề xuất]` |

---

### FR-04.4 — Vòng đời trục Triển khai & bàn giao
- **Mô tả**: Điều động tối thiểu, bắt đầu triển khai, tạm dừng, hủy, đóng dự án.
- **Input – Xử lý – Output**: Input: Trưởng nhóm, nhân sự, vai trò, từ ngày; mốc Bắt đầu; lý do; xác nhận đóng. Xử lý: chuyển trạng thái theo A.4a, kiểm tra điều kiện, tính hết bảo hành. Output: trục triển khai đúng trạng thái, dự án "Đóng".
- **Màn hình UI**: `[WIREFRAME: Điều động tối thiểu]` · `[WIREFRAME: Trạng thái và đóng dự án]`

**Luồng xử lý:**
- **Happy**: điều động → Bắt đầu → Đang triển khai → (UC-04.1, UC-04.2) → Đã bàn giao - bảo hành → Đóng.
- **Exception**: Bắt đầu khi chưa điều động; Hủy khi đang triển khai; Đóng khi chưa đủ điều kiện.
- **Edge/Boundary**: một người tham gia nhiều dự án; tạm dừng kéo dài; hết bảo hành nhưng còn công nợ; dự án kênh A đóng.
- **Alternative**: Hủy khi chưa triển khai.

**Business Rule:**
| Mã | Business Rule |
|---|---|
| BR-04.20 | Điều động tối thiểu: Trưởng phòng Chuyển giao cử Trưởng nhóm và nhân sự, mỗi dòng gồm người, vai trò, từ ngày (đến ngày khi kết thúc); dự án "Mới tạo" → "Đã điều động"; MVP không kiểm tra trùng lịch, công tác, nghỉ bù (FR-08, Phát hành 3) nhưng lưu dữ liệu `DIEU_DONG` để dùng cho FR-08, FR-09 |
| BR-04.21 | Ghi mốc Bắt đầu triển khai chỉ khi dự án "Đã điều động"; chuyển "Đang triển khai"; mốc Bắt đầu lưu người ghi và thời điểm (BR-01.6) |
| BR-04.22 | Tạm dừng/tiếp tục bắt buộc lý do; trong khi Tạm dừng không ghi mốc Vận hành hoặc Rút (BR-01.7, AC-01.1.4); thời gian Tạm dừng được ghi riêng; việc có tính vào hạn nhắc hay không `[CẦN LÀM RÕ VỚI KH]` |
| BR-04.23 | Hủy chỉ ở "Mới tạo" hoặc "Đã điều động", bắt buộc lý do, do Trưởng phòng Chuyển giao (kênh B: Kinh doanh được đề nghị); dự án đang triển khai không hủy mà dùng Tạm dừng |
| BR-04.24 | Dự án chỉ "Đóng" khi đủ cả: (a) trục triển khai "Đã bàn giao - bảo hành"; (b) hết bảo hành = ngày BBNT + số tháng mốc/bảo hành của sản phẩm (tham số, G20); (c) trục doanh thu "Đã thu"; (d) không còn đầu việc tồn mở. Khi đủ điều kiện hệ thống tạo việc "Xác nhận đóng dự án" cho CSKH phụ trách; CSKH xác nhận mới "Đóng" `[BA đề xuất]` phần xác nhận đóng (BR-01.2, D2). Việc nhắc ký bảo trì khi hết bảo hành thuộc FR-07 |

**Acceptance Criteria:**
| Mã | Loại luồng | BR liên quan | Nguồn | Given / When / Then |
|---|---|---|---|---|
| AC-04.4.1 | Happy | BR-04.20 | UC-04.4 | **Given** dự án "Mới tạo" **When** Trưởng phòng Chuyển giao cử Trưởng nhóm và 3 nhân sự kèm vai trò, từ ngày **Then** dự án chuyển "Đã điều động" và 4 dòng `DIEU_DONG` được lưu với người cử và thời điểm |
| AC-04.4.2 | Happy | BR-04.21 | UC-04.4 | **Given** dự án "Đã điều động" **When** Trưởng nhóm ghi mốc Bắt đầu **Then** dự án chuyển "Đang triển khai" |
| AC-04.4.3 | Exception | BR-04.21 | UC-04.4 E1 | **Given** dự án chưa điều động **When** ghi mốc Bắt đầu **Then** hệ thống chặn và nêu "chưa điều động" |
| AC-04.4.4 | Happy | BR-04.22 | UC-04.4 | **Given** dự án "Đang triển khai" **When** Trưởng nhóm tạm dừng kèm lý do rồi tiếp tục **Then** dự án đi "Tạm dừng" rồi "Đang triển khai", khoảng thời gian tạm dừng ghi riêng |
| AC-04.4.5 | Exception | BR-04.23 | UC-04.4 E3 | **Given** dự án "Đang triển khai" **When** có người chọn Hủy **Then** hệ thống chặn và hướng dẫn dùng Tạm dừng |
| AC-04.4.6 | Edge | BR-04.23 | UC-04.4 | **Given** dự án "Đã điều động" nhưng khách đổi ý **When** Trưởng phòng Chuyển giao hủy kèm lý do **Then** dự án "Hủy" và không sinh đầu việc doanh thu nào |
| AC-04.4.7 | Happy | BR-04.24 | UC-04.4 | **Given** dự án "Đã bàn giao - bảo hành", đã hết bảo hành, trục doanh thu "Đã thu" và không còn việc tồn mở **When** hệ thống kiểm tra **Then** tạo việc "Xác nhận đóng dự án" cho CSKH phụ trách; sau khi CSKH xác nhận dự án "Đóng" `[BA đề xuất]` |
| AC-04.4.8 | Exception | BR-04.24 | UC-04.4 E2 | **Given** trục doanh thu của dự án còn "Đang công nợ" **When** CSKH xác nhận đóng **Then** hệ thống chặn và nêu điều kiện còn thiếu |
| AC-04.4.9 | Edge | BR-04.20 | UC-04.4 | **Given** một nhân viên được điều động vào hai dự án cùng khoảng ngày **When** cử người **Then** hệ thống vẫn cho cử (MVP không kiểm tra trùng lịch) và lưu cả hai dòng điều động |
| AC-04.4.10 | Edge | BR-04.24 | UC-04.4 | **Given** dự án kênh A có trục doanh thu "Đã thu" sau khi đối soát **When** hệ thống kiểm tra điều kiện đóng **Then** áp cùng điều kiện (a)–(d) như kênh B |

---

## 5. Business Rule chung của tính năng
Các BR áp dụng cho toàn tính năng: **BR-04.1** (danh sách tồn hoặc xác nhận "không còn tồn" là điều kiện để Rút), **BR-04.8** (chỉ CSKH xác nhận nhận mới sang "Đã bàn giao - bảo hành") và **BR-04.24** (điều kiện đóng dự án, tiếp nối BR-01.2). BR riêng từng sub-feature nằm trong từng sub; mã `BR-04.n` duy nhất trong toàn tính năng.

## 6. Yêu cầu phi chức năng riêng
**Per-feature checkpoint:** `Giống Backbone (không khác biệt)`. Tính năng dùng NFR chung `NFR-G2.1` (phạm vi dữ liệu), `NFR-G7.1` (hạn rà soát, số tháng bảo hành là tham số), `NFR-G8.1` (Trưởng nhóm lập danh sách trên điện thoại tại đơn vị), `NFR-G10.1` (lưu vết xác nhận nhận/trả lại, mốc, đổi người).

| Mã | Nhóm NFR | Yêu cầu đặc thù của feature này | Khác gì so với NFR chung |
|---|---|---|---|
| — | — | Theo NFR chung (Backbone mục 3) | Không khác |

## 6b. Phân quyền
> Action: V xem · C tạo · E sửa · A duyệt/xác nhận · X thực hiện chuyển trạng thái. Phạm vi dữ liệu chi tiết do FR-06.

| Chức năng/thao tác (theo sub-feature) | Role được phép | Action | Điều kiện |
|---|---|---|---|
| FR-04.1 Lập danh sách tồn, xác nhận không còn tồn | Trưởng nhóm triển khai | C, E | Dự án "Đang triển khai"; chỉ dự án được điều động |
| FR-04.1 Ghi mốc Rút | Trưởng nhóm triển khai | X | BR-04.1, BR-04.3 |
| FR-04.1 Mở lại sau Rút | Trưởng phòng Chuyển giao | X | Có lý do `[BA đề xuất]` |
| FR-04.2 Rà soát, xác nhận nhận, trả lại | CSKH | A, X | Không phải người lập danh sách (BR-04.12) |
| FR-04.2 Phân công hàng đợi chung | Trưởng phòng CSKH | E | `[BA đề xuất]` |
| FR-04.2 Xem tình trạng bàn giao | Trưởng phòng Chuyển giao, Trưởng phòng CSKH | V | — |
| FR-04.3 Tạo việc từ xa | Nhân viên triển khai, Trưởng nhóm, CSKH | C | Dự án chưa "Đóng" |
| FR-04.3 Xử lý, hoàn thành đầu việc tồn | Người phụ trách | X | Có kết quả |
| FR-04.3 Đổi người phụ trách, hủy việc | Trưởng phòng của người phụ trách; CSKH phụ trách đơn vị | E | Hủy bắt buộc lý do |
| FR-04.4 Điều động tối thiểu, hủy dự án | Trưởng phòng Chuyển giao | C, X | BR-04.20, BR-04.23 |
| FR-04.4 Ghi mốc Bắt đầu, tạm dừng/tiếp tục | Trưởng nhóm triển khai | X | Bắt buộc lý do khi tạm dừng |
| FR-04.4 Xác nhận đóng dự án | CSKH | A, X | BR-04.24 `[BA đề xuất]` |

## 7. Rủi ro & Phụ thuộc
| Rủi ro / Phụ thuộc | Ảnh hưởng | Hướng xử lý |
|---|---|---|
| Giả định G12 (Trưởng nhóm lập danh sách, CSKH xác nhận) sai | Đổi người chịu trách nhiệm ở A.3.3 và máy trạng thái A.4a | Chạy thử một bàn giao thật với CSKH và Trưởng nhóm (Phần B, B.5) trước khi phát triển |
| Trưởng nhóm lập danh sách "cho có" hoặc bỏ sót | CSKH vẫn nhận thiếu, KK05 không giải được | CSKH rà soát bắt buộc; theo dõi số đầu việc phát sinh sau bàn giao (đo ở FR-13) |
| CSKH quá tải nên rà soát chậm | Dự án kẹt ở "Đã rút - chờ bàn giao" | Hạn rà soát là tham số, leo thang Trưởng phòng CSKH, hàng đợi chung |
| Việc từ xa không được ghi lại | Danh sách tồn thiếu việc từ xa | Việc từ xa là đầu việc thường trong "Việc của tôi"; việc còn mở tự vào danh sách (BR-04.5) |
| Hai chiều với FR-01 (trục doanh thu, điều kiện Đóng) | Đổi mô hình hai trục ảnh hưởng cả hai | Chốt `DU_AN` và hai trục trạng thái trước (A.5) |
| Hai chiều với FR-03 (việc tồn là lỗi/yêu cầu thay đổi) | Việc tồn bị xử lý ngoài cổng | BR-04.14: tạo Yêu cầu qua cổng, liên kết với việc tồn |
| Phụ thuộc FR-05 (nhắc, "Việc của tôi") | Không có nhắc thì rà soát và việc tồn chỉ là dữ liệu | Phát hành cùng FR-05 trong MVP |
| Phụ thuộc FR-06 (quyền, danh mục nhân sự, vết) | Không phân biệt người lập và người nhận | Phát hành cùng FR-06 |
| Điều động tối thiểu chưa kiểm tra trùng lịch | Một người ở nhiều dự án cùng lúc không bị cảnh báo | Chấp nhận trong MVP; FR-08 xử lý ở Phát hành 3 |
| Dữ liệu dự án đang mở lúc bắt đầu dùng | Dự án sắp rút nhưng chưa có điều động hoặc mốc | Nhập ban đầu theo FR-06.5, trạng thái chưa rõ ghi "Cần xác minh" |

## 8. Mốc kiểm tra giữa chừng (Milestone Review)
| Mốc | Ngày dự kiến | Nội dung review với KH | Trạng thái |
|---|---|---|---|
| Milestone 1 | Chưa lên lịch | FR-04.1 và FR-04.4: màn "Chuẩn bị rút" và danh sách tồn, điều động tối thiểu, trục trạng thái; xác nhận G12, G21 với Trưởng nhóm | Chưa bắt đầu |
| Milestone 2 | Chưa lên lịch | FR-04.2: CSKH rà soát, nhận/trả lại, đầu việc tồn trong "Việc của tôi"; xác nhận hạn rà soát | Chưa bắt đầu |
| Milestone 3 | Chưa lên lịch | FR-04.3 và điều kiện đóng dự án: việc từ xa, tồn đọng đơn vị, xác nhận đóng; xác nhận G20 (số tháng mốc) | Chưa bắt đầu |

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
| 1 | BR-04.1, BR-04.8 | Có trường hợp nào được rút mà không cần CSKH xác nhận nhận, hoặc việc nào bắt buộc xong trước khi rút? (G12) | Nice-to-know | Chưa hỏi |
| 2 | BR-04.4, BR-04.11 | Hạn CSKH phải rà soát sau khi đội rút là bao nhiêu ngày? | Nice-to-know | Chưa hỏi |
| 3 | BR-04.24 | Thời hạn bảo hành theo từng sản phẩm và theo kênh (kênh A bảo hành do ai, tính từ ngày nào)? "Số tháng mốc" tính từ ngày nào? (G20) | Nice-to-know | Chưa hỏi |
| 4 | BR-04.22 | Thời gian Tạm dừng có tính vào hạn nhắc (Chờ nghiệm thu…) không? | Nice-to-know | Chưa hỏi |
| 5 | BR-04.3, AC-04.1.9 | Thực tế đội thường rút trước hay sau khi bệnh viện ký BBNT? (G21) | Nice-to-know | Chưa hỏi |
| 6 | BR-04.24 | Ai có quyền xác nhận đóng dự án (CSKH hay Trưởng phòng Chuyển giao)? | Nice-to-know | Chưa hỏi |
| 7 | BR-04.7, BR-04.11, BR-04.13, BR-04.14, BR-04.24 | Xác nhận các bổ sung `[BA đề xuất]` (mở lại sau Rút, leo thang Trưởng phòng CSKH, hàng đợi chung, việc tồn là lỗi đi qua cổng FR-03, xác nhận đóng dự án) | Blocking trước khi xác nhận spec | Chưa hỏi |

## 11. Change Request Log
| Version | Ngày | Người sửa/update | Nội dung thay đổi / CR | Ảnh hưởng (FR/AC/BR) | Trạng thái |
|---|---|---|---|---|---|
| v1.0 | 25/10/2026 | PO/BA | Bản draft đầu, theo Backbone v1.1, Phần A (A.3.3, A.4a, A.5) | — | Draft |
