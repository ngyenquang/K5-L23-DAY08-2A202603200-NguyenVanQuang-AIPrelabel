# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603200 - Nguyễn Văn Quang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | Có. Kính thuộc mask xe kể cả khi nhìn xuyên thấy người bên trong. Người vẫn là box `person` riêng, mask xe không cắt lỗ cho người. | img_013 | CAL-03 |
| 7 | Có đặt mức tin cậy tối thiểu để chấp nhận box máy không cần sửa không? | Không. Mọi box máy phải được duyệt bằng mắt. `conf` chỉ dùng để ưu tiên xem box điểm thấp, không dùng để bỏ qua box điểm cao (ví dụ truck 0.73 sai ở img_002, box đúng chỉ 0.27 ở img_032). | img_002, img_032 | CAL-04 |
| 1, 2, 3, 4, 6 | Xe đạp điện; khối xe máy dày gần camera; hàng nhô; người trong xe có mui; box máy chồng nhau trong ảnh đêm | **Chưa chốt** trong bản v1.1 này: chưa đủ ca để quyết định chắc chắn (xem phần ghi chú trong `REPORT.md`). | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_002, img_032 | Tuk-tuk xanh, xích lô | YOLO26 bỏ qua hoàn toàn (không có box car/truck); YOLOE tìm đúng (0.93 và 0.92) | Gán `three_wheeler` | Mục 2 và 5: tuk-tuk, xích lô luôn là `three_wheeler`, dù máy gọi gì hoặc bỏ qua (CAL-01) |
| img_004, img_013 | Tuk-tuk gần camera | YOLO26 gọi tuk-tuk là `car` (0.37, 0.34, 0.42) | Đổi nhãn thành `three_wheeler` | Như trên (CAL-01) |
| img_032 | Xích lô có người đạp ở phía sau | YOLO26 gọi khung xích lô bên phải là `bicycle` 0.36 | Một xe một box: phần đạp thuộc box `three_wheeler`, không vẽ `bicycle` riêng | Mục 5: thêm "một xích lô chỉ có một box `three_wheeler`, không tách phần đạp thành `bicycle`" (CAL-01) |
| img_002 | Xe bán tải trắng ở bên phải ảnh | YOLO26 gán `truck` 0.73 | Đổi thành `car` (bán tải thuộc `car` theo mục 2 v1.0) | Không đổi guideline: máy gán sai, guideline đã rõ (CAL-02) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi 5 (kính thuộc mask xe) và câu hỏi 7 (không có ngưỡng `conf` tự chấp nhận); làm rõ tuk-tuk và xích lô luôn là `three_wheeler`, một xe một box | CAL-01, CAL-03, CAL-04 (CAL-02 không đổi guideline) |
