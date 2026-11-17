# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 522; đối tượng sau duyệt: 508; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 463 | 463/522 = 88.7% | 522 box máy |
| edited (sửa) | 22 | 22/522 = 4.2% | 522 box máy |
| deleted (xoá) | 37 | 37/522 = 7.1% | 522 box máy |
| added (vẽ thêm) | 23 | 23/508 = 4.5% | 508 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 18.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 262 | 2 | 4 | 5 |
| bicycle | 5 | 2 | 3 | 0 |
| motorbike | 131 | 7 | 1 | 7 |
| car | 50 | 0 | 4 | 4 |
| bus | 2 | 0 | 1 | 1 |
| truck | 3 | 0 | 3 | 1 |
| three_wheeler | 10 | 11 | 21 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 83 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 51 | 23 | 32 | 68.9% (KTC 95%: 58–78%) | 61.4% (KTC 95%: 51–71%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - wrong_class: 2/3
  - missing: 1/3
  - extra: 2/3
  - lọt: img_029.jpg (loose), img_013.jpg (missing), img_014.jpg (wrong_class), img_034.jpg (loose), img_025.jpg (loose), img_024.jpg (missing), img_005.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
