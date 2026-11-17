# Thông tin học viên

- Họ và tên: Nguyễn Văn Quang
- MSHV: 2A202603200
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: ngyenquang
- Repo bài nộp (URL): https://github.com/ngyenquang/K5-L23-DAY08-2A202603200-NguyenVanQuang-AIPrelabel
- Commit chốt bài (hash): 17ce57f
- Máy: Windows 11, GPU NVIDIA RTX 3050 Ti 4 GB (CUDA) nhưng RAM chỉ 7,6 GB nên model chạy trên Google Colab (T4)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 463 / 22 / 37 (mẫu số: 522 box máy); added: 23 (mẫu số: 508 đối tượng)
- So với gold: precision / recall của **máy** 68.9% / 61.4% (mẫu số: 83 đối tượng gold, 4 ảnh)
- Bắt bẫy: 5/12 = 41.7%
- Thời gian: gán tay 2.5 phút/ảnh (10 phút / 4 ảnh); duyệt có pre-label 0.625 phút/ảnh (15 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/107) ở vòng 2 (vòng 1: 40.0% theo ảnh = 2/5, 2.0% theo đối tượng = 2/102)
- **Quyết định:** Rework — lý do một dòng: bắt bẫy 5/12 = 41.7% dưới ngưỡng 80% nên lô duyệt chưa đáng tin, dù lỗi QC vòng 2 là 0/5 ảnh (mẫu nhỏ, cận trên 95% là 43%)
