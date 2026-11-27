# Báo cáo Day 14 — Nguyen Ngoc Dong — 2A202603291

- Mã học viên dùng với `--student`: `2A202603291`
- `seed_tag` trong `frames_A.txt`: `82ac3100fd`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

**Bản nháp do AI soạn để tham khảo.**

Dự đoán tăng `score_thr` từ `0.1` lên `0.3` và giữ `min_points = 0` sẽ giảm tổng công sửa trên tập A. Ở frame `s03f41`, hai hộp `pp_id` 31 và 35 vẽ nhầm lên bụi cây, với điểm tin cậy lần lượt là 0,2791 và 0,2410. Ngưỡng `0.3` sẽ lọc hai hộp đó. Giữ `min_points = 0` giúp tránh áp thêm bộ lọc số điểm lên các vật thể thưa điểm như người đi bộ và xe hai bánh ở xa. Sweep sẽ kiểm tra liệu công xóa tiết kiệm được có bù phần công thêm nếu mô hình bỏ sót vật thật hay không.

## 2. Tập A: sửa tay và tune

Có dùng AI để hỗ trợ rà ảnh camera và LiDAR, loại hộp thừa, chỉnh một số hộp và thêm vật bị sót. Nhãn đã được học viên rà lại trong CVAT.

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | 71 | 6 | 1 | 27,8 |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_2A202603291/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_2A202603291/frames_B.txt --config d14_2A202603291/best_config.json --gt d14_2A202603291/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- Nêu ít nhất 2 giới hạn (ví dụ: chỉ 3 frame tune; model KITTI không có Truck/Bus; chi phí là công thức, chưa phải thời gian sửa thật): ...

