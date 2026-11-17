# Báo cáo Day 08 — 2A202603200 - Nguyễn Văn Quang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | bicycle=0.2; person, motorbike, car, bus, truck dùng mặc định 0.3 (threshold_sweep.md: person recall tối đa 76%/38 vật, motorbike 83%/23, car 88%/8 đều không đạt 90% ở ngưỡng ≤ 0.5; bus 0 vật, truck 1 vật trong gold nên không chọn ngưỡng riêng) | YOLO26 nhận tốt người và xe máy (r01: person=131, motorbike=55 trên 8 ảnh pilot) nên dùng cho 6 lớp COCO; r02 cho thấy tăng ngưỡng lên 0.5 chỉ làm mất 76/215 box (-35%) mà không cứu được lớp lạ nên không dùng một ngưỡng chung; r05 là nguồn ở ngưỡng 0.05. bicycle=0.2 vì threshold_sweep cho recall 100% trên 6 vật gold. |
| three_wheeler | YOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 (threshold_sweep.md: recall 100%, precision 78%, 7 vật gold) | YOLO26 bỏ qua hoàn toàn tuk-tuk (img_002) và xích lô (img_032) (r01, r02), còn YOLOE tìm đúng (img_002 0.93, img_032 0.92; r03). Ngưỡng 0.15 thấp vì điểm đúng của YOLOE có khi chỉ 0.27–0.28 (img_032, img_011); đổi lại precision chỉ 78% (r06). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 base (`sam2.1_b.pt`). Lý do: mask bám biên rất khít khi box đúng (r04, img_004: người 0.93), chạy 1.8 s/ảnh trên T4, bản base đủ nhẹ, không cần xin quyền như SAM 3. Tôi không so sánh với SAM 1 nên chưa biết khác biệt về biên mask.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 hoặc bỏ qua hẳn tuk-tuk và xích lô (img_002, img_032; r01, r02) hoặc gọi chúng là `car` (r04: img_004 car 0.37 và 0.34, img_013 car 0.42), vì COCO không có lớp xe ba bánh; tăng ngưỡng (r02) không thêm được lớp này. Mask SAM cũng không cứu được nhãn sai lớp. Cách sửa: dùng YOLOE cho three_wheeler (đưa file YOLOE trước trong `merge_prelabels.py` để ưu tiên) và đổi nhãn bằng tay khi review.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_002 xe bán tải trắng bị gán `truck` 0.73 (guideline: bán tải là `car`). Điểm thấp mà đúng: img_032 xích lô bên phải 0.27 và img_011 tuk-tuk mép trái 0.28 (YOLOE).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: precision/recall 68.9% / 61.4% được đo trên đúng 4 ảnh dùng để chọn ngưỡng (in-sample), lớp ít vật (bus 0, truck 1) không chọn được ngưỡng riêng, nên kết luận về ngưỡng chỉ là ước lượng thô và máy có thể kém hơn trên ảnh mới.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 463 | 88.7% | 522 box máy |
| edited | 22 | 4.2% | 522 box máy |
| deleted | 37 | 7.1% | 522 box máy |
| added | 23 | 4.5% | 508 đối tượng sau duyệt |

(Nguồn: `qc/review_stats.md`, chạy trên đúng file `labels/batch_reviewed.json` nộp; đã loại 23 đối tượng quanh bẫy. Trong deleted + added có 5 trường hợp đổi lớp; máy sót thật ≈ 23 − 5 = 18.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: máy có 42 box (10 accepted + 11 edited + 21 deleted), chỉ 10/42 = 23.8% giữ nguyên, và 21/42 = 50.0% bị xóa; nguyên nhân là ngưỡng 0.15 rất thấp nên YOLOE sinh nhiều box thừa (ví dụ ở img_002: hai box 0.37 trên mặt tiền cửa hàng và xe bán tải). Các lớp khác máy làm khá tốt (person: 262 accepted / 2 edited / 4 deleted; motorbike: 131 / 7 / 1). Số vật sót (added) thấp ở mọi lớp (person 5, motorbike 7, car 4, three_wheeler 5, bicycle 0), nhưng recall trên gold chỉ 61.4%, nên `added` thấp không có nghĩa máy không sót: những vật cả tôi và máy cùng bỏ sót thì không được tính. (Số theo lớp từ `review_stats.md`.)
- So với gold: precision / recall của **máy** 68.9% (KTC 95%: 58–78%) / 61.4% (KTC 95%: 51–71%) (mẫu số: 83 đối tượng gold, 4 ảnh; TP 51, FP 23, FN 32). Số này lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy (trên file nộp `batch_reviewed.json`): 5/12 = 41.7% (KTC 95%: 19–68%). Theo loại: loose 0/3, wrong_class 2/3, missing 1/3, extra 2/3. Lọt: img_029 (loose), img_034 (loose), img_025 (loose), img_013 (missing), img_024 (missing), img_014 (wrong_class), img_005 (extra).
- **Lịch sử các lần đo:** lần 1 export chưa sửa gì: 0/12; lần 2 duyệt sau khi đã thấy danh sách ảnh có bẫy (từ lần chạy `review_stats` đầu): 5/12; lần 3 duyệt lại sau khi đã thấy danh sách bẫy lọt của lần 2: 8/12 (số này bị nhiễm nặng vì tôi sửa đúng các ảnh bị báo lọt). File nộp và các số trong báo cáo là **lần 2 (5/12)**; số 5/12 cũng chỉ là mốc gần thật, vì lúc đó tôi đã biết 12 ảnh nào có bẫy.
- Vì sao lọt (bạn đã bỏ bước nào của checklist)? Loại lọt nhiều nhất là `loose`: cả 3/3 lọt (img_029, img_034, img_025), tức bước 4 (kiểm box có ôm sát không) là bước tôi làm lướt nhất vì lệch vài pixel khó thấy; ngoài ra lọt 2/3 `missing` (img_013, img_024), 1/3 `wrong_class` (img_014) và 1/3 `extra` (img_005). Các ảnh lọt đều khó nhìn: ảnh chất lượng kém (chụp lúc xe đang chạy nên mờ, lóa), ảnh rất đông người chờ đèn đỏ, và ảnh chụp từ trên cao ở ngã tư nên vật nhỏ, dày. Ở ảnh đông tôi chưa quét theo góc phần tư như checklist yêu cầu.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 88.7% và added chỉ 4.5% nhìn như máy rất tốt, nhưng bắt bẫy chỉ 41.7% (dưới ngưỡng 80%) và recall của máy trên gold chỉ 61.4%. Hai điều đó cho thấy accepted cao và added thấp một phần là do tôi giữ lại lỗi nhỏ và bỏ sót vật ở ảnh khó (cả tôi và máy cùng sót thì không được tính vào `added`), chứ không chỉ do máy tốt. Accepted cao không có nghĩa tôi kiểm kỹ.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 10 phút / 4 ảnh = 2.5 phút/ảnh vs duyệt 15 phút / 24 ảnh = 0.625 phút/ảnh (khoảng 37 giây/ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): tôi gán gold trước nên đã quen kiểu cảnh này, làm cho thời gian duyệt có vẻ ngắn hơn; ngoài ra ảnh gold (img_001, img_036) đông xe máy nên mỗi ảnh gold có nhiều vật hơn trung bình ảnh batch; con số 15 phút cho nhiều lượt duyệt nhiều khả năng thấp hơn thời gian làm thật, nên mức tiết kiệm có thể bị phóng đại, và tôi không đo thời gian riêng từng lượt.
- Kết luận: **chỉ cho một số lớp**, dựa trên số liệu: máy đáng dùng cho `person`, `motorbike`, `car` (accepted: person 262/268 = 97.8%, motorbike 131/139 = 94.2%, car 50/54 = 92.6% box máy giữ nguyên), nhưng **không đáng dùng cho `three_wheeler`** (chỉ 10/42 = 23.8% giữ nguyên, 21/42 = 50.0% bị xóa). Vì recall của máy trên gold chỉ 61.4% (83 vật) và bắt bẫy 5/12 = 41.7% (dưới 80%), thời gian tiết kiệm không đủ để kết luận pre-label an toàn: phải thêm một bước quét ảnh trước khi nhìn nhãn máy.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy và đếm vật theo lớp trước ở 100% ảnh, ghi số đếm vào một file log (kiểm bằng file log có đủ 24 dòng).
  2. Chỉ chạy `review_stats.py` đúng một lần, sau khi đã export lần cuối (kiểm bằng lịch sử lệnh: một lần chạy cho mỗi lượt duyệt, không chạy giữa chừng).
  3. Ảnh có hơn 30 vật thì quét theo góc phần tư và dành ít nhất 3 phút mỗi ảnh (kiểm bằng `time_log.csv`: tổng thời gian duyệt 24 ảnh tối thiểu 72 phút).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có khách, người lái) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Không gồm khách ngồi sau; người lái ngồi bên phải gần như bị loại nhưng mask còn lỗ chấm đen li ti ở vùng thân người | Nhiễu lỗ chấm ở người lái (viền người và xe lẫn màu); mask gồm cả kính chắn gió | `masks/tuktuk_img013.jpg` |
| img_004 (tuk-tuk xanh dương lớn) | SAM 2.1 base (`sam2.1_b.pt`) | 6 (3 dương, 3 âm); lần đầu chỉ 304 px vì click trượt vật, lần hai 91.474 px (13,1% ảnh) | Không gồm người đứng bên phải và tuk-tuk xanh lá bên cạnh; không có người lái trong xe | Biển TAXI trên nóc có vẻ không nằm trong mask; mui, thân, ghế, hai bánh đều được phủ | `masks/tuktuk_img004.jpg` |
| img_032 (xích lô) | SAM 2.1 base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm); lần đầu click sát mép nên không ra mask, lần hai 49.557 px (8,4% ảnh) | Xe trống, không có người lái ngồi trong xe; không gồm hai người đứng cạnh (nhờ hai điểm âm) | Phủ cả mui, đệm hoa, khung trước và hai bánh; viền ở phần khung nan có vẻ hơi dày | `masks/xichlo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc giúp: "hai vật chạm nhau: mỗi vật một polygon, không chồng lên nhau" và "kính thuộc vật" (mask img_013 gồm kính chắn gió, khớp quy tắc), và quy tắc người là vật riêng (mục 4.1.4) khiến mask xe đúng khi không gồm người. Quy tắc còn thiếu: biển TAXI gắn trên nóc có thuộc xe không (img_004), và lỗ thủng khi người ngồi trong xe có mui che (câu hỏi mở số 4 và số 5), vì mask img_013 có chấm đen ở người lái mà guideline chưa nói rõ cách xử lý.
- Nhận xét chung: số click cần tăng khi vật có phần phức tạp (img_004 cần 6 click vì có nhiều xe giống nhau cạnh nhau); click dương nên đặt giữa các phần lớn của vật (mui, thân, ghế) chứ không đặt sát mép hoặc vào chi tiết nhỏ (lần đầu img_004 và img_032 click trượt nên SAM không ra mask).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: không thử (chỉ làm bằng `sam2_points.py` trên Colab, không dùng Interactor trong CVAT).

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-02 là `loi_may`; CAL-03, CAL-04 là `cau_hoi_mo`); quan trọng nhất: CAL-01 (YOLO26 không có lớp xe ba bánh nên bỏ qua hoặc gọi tuk-tuk/xích lô là car, bicycle; ảnh img_002, img_032, img_004, img_013).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: tuk-tuk và xích lô luôn là `three_wheeler` dù máy gọi gì, một xe một box (phần đạp không tách thành `bicycle`) (CAL-01); kính chắn gió thuộc mask xe, người bên trong vẫn là box riêng (CAL-03, chốt câu hỏi 5); không đặt ngưỡng `conf` để tự chấp nhận box máy (CAL-04, chốt câu hỏi 7). CAL-02 (bán tải bị gán `truck`) không đổi guideline.
- Nguyên nhân gốc: CAL-01 và CAL-02 là lỗi máy (guideline đã rõ ở mục 2 v1.0 cho bán tải; với xích lô, mục 5 chưa nói phần đạp phía sau nên làm rõ thêm); CAL-03 và CAL-04 là guideline mơ hồ (câu hỏi mở 5 và 7 của v1.0). Các câu 1, 2, 3, 4, 6 chưa chốt vì chưa đủ ca trong 4 ảnh gold và 24 ảnh batch.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42: img_003, 006, 014, 031, 035; 102 đối tượng); người kiểm: Claude (trợ lý AI) xem ảnh có vẽ nhãn và ghi phiếu, tôi xác nhận lại (**hạn chế**: tôi vừa là người duyệt; Claude chỉ nhìn được ảnh nén nên box nhỏ chồng nhau ở ảnh đông có thể bị sót; lỗi img_014 được xác định bằng cách đối chiếu `traps_key.json` sau khi đã export, nên không phải phát hiện độc lập bằng mắt).
- Lỗi theo **đối tượng**: 2/102 = 2.0% · theo **ảnh**: 2/5 = 40.0% (ngưỡng Release ≤ 2%) (nguồn: `qc/qc_report_auto.md`).
- Top lỗi: `sai_lop` 1 (img_014, object 141: box 12×33 px gán `bicycle` nhưng là người ở xa); `thua` 1 (img_031, object 440: box `person` 0.34 bao chiếc mũ treo trên giá hàng, không phải người). Các loại khác (bỏ sót, sai vị trí/kích thước, sai thuộc tính, sai mask): 0 trong mẫu.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): có 2 ảnh lỗi trong 5 ảnh, cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là 77%; mẫu 5 ảnh không thể chứng minh ngưỡng 2%, và 0 lỗi ở 3 ảnh còn lại không có nghĩa chúng sạch (ảnh đông, box nhỏ khó kiểm).
- Script kiểm được / không kiểm được: kiểm được tên file, lớp hợp lệ, đủ ảnh, số liệu accepted/edited/deleted/added, tỉ lệ lỗi theo phiếu. Không kiểm được box có ôm sát vật thật không, lớp đúng thực tế không, vật cả tôi và máy cùng bỏ sót, và việc tôi có theo checklist thật hay không (lần duyệt 2 và 3 đã bị nhiễm vì tôi biết ảnh có bẫy).

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đủ rõ cho các ca đã gặp (các ca mơ hồ đã chốt trong v1.1) và dữ liệu không hỏng (validate_labels sạch); Rework vì ảnh có lỗi 2/5 = 40.0% > 2% và bắt bẫy 5/12 = 41.7% < 80% (lô duyệt chưa đáng tin); lỗi thật tìm thấy trong mẫu cũng là loại đã lọt qua tay tôi khi duyệt.
Nếu Rework / Hold, việc cần làm tiếp: sửa object 141 (img_014) và object 440 (img_031) trong CVAT rồi export `labels/qc_fixed.json`; duyệt lại các ảnh bẫy lọt (img_029, 034, 025, 013, 024, 014, 005) theo checklist, quét trước khi nhìn nhãn máy; chạy lại `build_release.py` và `qc_sample.py --force` với mẫu lớn hơn và ghi vòng sau vào báo cáo.

**Vòng 2 (sau Rework):**
- Đã sửa hai lỗi của vòng 1 (img_014: box `bicycle` nhỏ đổi thành `person`; img_031: xóa box `person` bao chiếc mũ) và export `labels/qc_fixed.json`. Lưu ý quan trọng: `qc_fixed.json` được export từ trạng thái hiện tại của Task trong CVAT nên **không chỉ có hai sửa này**: nó gồm cả các sửa của lần duyệt thứ 3 (630 đối tượng, so với 519 trong `batch_reviewed.json`; ví dụ img_014 từ 19 lên 25 đối tượng), và `build_release.py` đã thay cả 24 ảnh bằng bản này (release 630 đối tượng, trong đó `ignore_region` 1). Vì vậy số đo máy và bắt bẫy ở mục 2–3 (đo trên `batch_reviewed.json`) và nhãn trong `release/` là hai bản khác nhau.
- Chạy lại `qc_sample.py --n 5 --seed 42 --force`: cùng 5 ảnh (img_003, 006, 014, 031, 035), nay 107 đối tượng. Kết quả: lỗi theo **đối tượng** 0/107 = 0.0%, theo **ảnh** 0/5 = 0.0% (cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là 43%).
- Giới hạn: vòng 2 dùng **đúng 5 ảnh** của vòng 1, nơi tôi đã biết lỗi và vừa sửa, nên 0/5 không phải bằng chứng độc lập; 3 ảnh còn lại (img_003, 006, 035) vẫn chỉ được xem qua ảnh nén ở ảnh đông. Muốn kết luận đáng tin cần mẫu mới (seed khác, nhiều ảnh hơn).
- **Quyết định sau vòng 2: vẫn Rework**, vì bắt bẫy 5/12 = 41.7% < 80% (người duyệt chưa đáng tin; số này đo ở lần duyệt thứ 2 và không đổi khi sửa nhãn), dù lỗi theo ảnh trong mẫu giảm về 0/5. Việc cần làm tiếp: lấy mẫu QC mới và lớn hơn (ví dụ seed khác, n = 8) kiểm nhãn trong `release/`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (trợ lý AI) dùng để đọc và giải thích đề, hướng dẫn thao tác CVAT và Colab, hướng dẫn chạy các script, giải thích cách đọc số đo. Theo hướng dẫn của labcoach (chỉ việc gán nhãn phải làm không dùng AI, báo cáo được dùng AI).
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không: tôi gán 4 ảnh gold hoàn toàn bằng tay trong CVAT, không nhìn pre-label và không nhờ AI xem ảnh gold.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, RTX 3050 Ti 4 GB, RAM 7,6 GB; model chạy trên Google Colab (T4); CVAT v2.74.1 chạy bằng Docker (bản có sẵn trên máy, lab dùng 2.81), tắt bớt container phụ bằng `lab.py lite` của lab Day04 để đỡ RAM. Vướng mắc: RAM chỉ 7,6 GB nên phải chạy model trên Colab và bật CVAT lite; Docker chưa bật lúc đầu; lệnh tạo `.venv` trong shell của Claude Code báo lỗi sao chép `venvlauncher.exe`, tôi tạo lại thành công trong PowerShell; lúc tạo mask SAM, lần đầu img_004 (304 px) và img_032 (không ra mask) phải chọn lại điểm click.
- Dùng cache `data/cache/` thay vì tự chạy? Không, đã tự chạy trên Colab.
- Lỗi / bug script đã gặp (nếu có): không gặp bug trong script của lab; `validate_labels.py` chỉ cảnh báo vật cao dưới 20 px (guideline 4.1.6), không phải lỗi; `threshold_sweep.md` có lỗi chính tả ("báocáo") không ảnh hưởng.
- Một điều bạn làm khác lần sau: chỉ chạy `review_stats.py` một lần sau khi export lần cuối (tôi đã chạy nhiều lần và làm lộ danh sách bẫy lọt, nên số bắt bẫy bị nhiễm), và bấm giờ từng lượt duyệt thay vì ghi một số tổng.
