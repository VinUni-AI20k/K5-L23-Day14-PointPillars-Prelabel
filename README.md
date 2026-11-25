# Day 14 — Pre-label 3D bằng PointPillars pre-trained (VF LiDAR)

**Bài cá nhân.** Thời lượng: cài đặt ≈10 phút trước buổi; buổi giảng sáng; lab buổi chiều. Nộp link commit GitHub (có SHA) lên LMS trước **12:00 trưa hôm sau** (GMT+7).

Tài liệu: [SUBMISSION.md](SUBMISSION.md) (nộp gì, nộp ở đâu) · [RUBRIC.md](RUBRIC.md) (chấm điểm) · [CHECKPOINTS.md](CHECKPOINTS.md) (tự kiểm từng bước) · [RULES.md](RULES.md) (quy định) · [report/individual/MSSV_HoTen.md](report/individual/MSSV_HoTen.md) (template báo cáo)

Chạy model PointPillars đã huấn luyện trên KITTI ([zhulf0804/PointPillars](https://github.com/zhulf0804/PointPillars), MIT, commit `620e6b0`) trên dữ liệu VF của Day 13, đẩy box lên CVAT làm pre-label, rồi tune cấu hình để **giảm công sửa tay**.
Code chạy bằng PyTorch trên CPU, không cần GPU và không cần biên dịch: hai op CUDA của upstream (voxelize, rotated NMS) được thay bằng bản Python thuần trong `vf_pp/upstream.py`. File của upstream giữ nguyên, không sửa dòng nào.

## 0. Cài trước buổi giảng (≈10 phút)

```bash
git clone <URL repo này> k5-day14-lab && cd k5-day14-lab
python -m venv .venv && source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install -r requirements.txt
python setup_pp.py                    # tải code upstream + weights (~20 MB); không có git: python setup_pp.py --download-zip
python pp_selftest.py --data <Data-VF>
```
`<Data-VF>` là thư mục dữ liệu đã tải ở Day 13 (chứa `sequence_01 … sequence_05`). Selftest in `TẤT CẢ OK` là xong. Nếu máy lỗi, dùng Docker theo hướng dẫn trong `Dockerfile`.

## 1. Buổi giảng: 6 frame chung (mọi người giống nhau)

```bash
python pp_prelabel.py --data <Data-VF> --frames demo/frames_lec.txt --out out/lec.json --plot out/lec_png
python pp_prelabel.py --data <Data-VF> --frames demo/frames_lec.txt --set score_thr=0.4 --set min_points=5
```
Mở `out/lec_png/*.jpg` (7 camera + BEV) để xem model bắt đúng, sót và thừa ở đâu.

## 2. Lab buổi chiều: frame riêng theo mã học viên + SEED

SEED **chỉ được coach công bố lúc bắt đầu lab**. Không có SEED thì lệnh từ chối chạy.

| Bước | Lệnh | Kết quả |
|---|---|---|
| 1. Chọn frame | `python vf_cvat.py day14 --student <ID> --seed <SEED> --data <Data-VF>` | `d14_<ID>/frames_A.txt` (3 frame tune), `frames_B.txt` (12 frame nộp) |
| 2. Pre-label A | `python pp_prelabel.py --data <Data-VF> --frames d14_<ID>/frames_A.txt --out d14_<ID>/pred_A.json --plot d14_<ID>/png_A` | box của cấu hình gốc + ảnh để xem |
| 3. Lên CVAT | `python vf_cvat.py prelabel --student <ID> --task A --data <Data-VF> --boxes d14_<ID>/pred_A.json --host <URL> --user <tên>` | task `D14-<ID>-A` có sẵn cuboid (thuộc tính `pp_id`, `score`, `warn`) |
| 4. Sửa tay A | Trên CVAT: xóa box thừa, thêm box thiếu, chỉnh box lệch theo guideline Day 13. Bấm Save. | nhãn tham chiếu của bạn |
| 5. Lấy tham chiếu | `python vf_cvat.py reference --student <ID> --task A --host <URL> --user <tên>` | `d14_<ID>/ref_A.json` |
| 5b. Đoán trước | Trước khi chạy sweep, ghi vào báo cáo 2 dòng: *tham số nào bạn nghĩ giảm công sửa nhiều nhất, và vì sao* (nhìn ảnh BEV: box thừa nhiều hay sót nhiều?). | dự đoán để đối chiếu ở bước 8 |
| 6. Tune | `python pp_sweep.py --data <Data-VF> --frames d14_<ID>/frames_A.txt --gt d14_<ID>/ref_A.json --grid score_thr=0.1,0.2,0.3,0.4,0.5 --grid min_points=0,5,10 --out d14_<ID>/sweep.csv` | `sweep.csv`, `best_config.json`, top 10, leave-one-out |
| 7. Pre-label B | `python pp_prelabel.py --data <Data-VF> --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --out d14_<ID>/pred_B.json` rồi `vf_cvat.py prelabel --task B --boxes d14_<ID>/pred_B.json ...` | task `D14-<ID>-B` để sửa và nộp |
| 8. Trước/sau | `python pp_compare.py --data <Data-VF> --frames d14_<ID>/frames_A.txt --gt d14_<ID>/ref_A.json --tuned d14_<ID>/best_config.json --out d14_<ID>/compare` | % công sửa giảm được, ảnh BEV gốc ↔ của bạn cho từng frame. Đưa 1 ảnh vào báo cáo và viết: dự đoán ở bước 5b đúng hay sai? |
| 9. Xuất nhãn B, nộp | Sửa xong 12 frame B trên CVAT, bấm Save, rồi `python vf_cvat.py reference --student <ID> --task B --host <URL> --user <tên>`. Commit và push theo [SUBMISSION.md](SUBMISSION.md) | `d14_<ID>/ref_B.json` (sản phẩm chính để chấm) |

Đo lại: `pp_prelabel.py ... --config <cfg> --gt d14_<ID>/ref_A.json` in số box phải **xóa / thêm / sửa** và chi phí `0.3·xóa + 1.0·thêm + 0.5·sửa`. Chỉ tính box có tâm cách LiDAR tối đa 50 m (cố định, không đổi theo tham số `range_m`). Box dự đoán khớp box tham chiếu khi IoU nhìn từ trên xuống (BEV) ≥ 0.3. Một box khớp vẫn bị tính "sửa" khi sai lớp, tâm lệch quá 0.3 m, yaw lệch quá 10° hoặc một cạnh lệch quá 0.2 m.

CVAT ở đây là CVAT cá nhân chạy Docker trên máy bạn (như Day 13). Mật khẩu CVAT đọc từ biến `CVAT_PASSWORD` hoặc được hỏi khi chạy.

## 3. Các tham số (`vf_pp/core.py:Config`)

| Nhóm | Tham số | Ý nghĩa |
|---|---|---|
| A. Đầu vào | `voxel_size` (0.16), `intensity_mode` clip/log/const, `intensity_scale`, `ground_mode` fixed/auto, `delta_z`, `passes` (4) | Đưa điểm VF về miền KITTI. Model chỉ nhìn 90° phía trước nên mỗi frame chạy 4 lượt xoay để phủ 360° |
| B. Đầu ra model | `score_thr` (0.1), `thr_car`/`thr_cyc`/`thr_ped`, `nms_thr`, `nms_pre`, `max_num` | Lọc theo độ tin cậy và NMS |
| C. Hậu xử lý VF | `range_m` (50), `min_points` (0), `merge_nms_thr`, `size_pad_m`, `truck_l_min`/`truck_h_min`, `bus_l_min`/`bus_h_min` | KITTI không có Truck/Bus: box Car dài/cao sẽ được đổi lớp theo luật kích thước |

Ghi đè nhanh: `--set KEY=VALUE`. Lưu cấu hình: sửa `best_config.json` rồi dùng `--config`.

## 4. Hệ trục

VF: x trái, y sau, z lên (phía trước xe là −y), yaw 0…2π tính từ +x, ngược chiều kim đồng hồ. KITTI: x trước, y trái. Phép đổi trục nằm trong `vf_pp/core.py` (`x_kitti = −y_vf`, `y_kitti = x_vf`).
Đã kiểm trên 6 frame của buổi giảng: với box khớp nhãn, trung vị sai lệch là 13 cm (tâm), 2.3° (yaw), dưới 6 cm (kích thước).

## Điểm cộng (tùy chọn)

Sau khi nộp xong phần bắt buộc, xem [`bonus/README.md`](bonus/README.md): đo thao tác sửa nhãn bằng userscript (hướng chính), hoặc thử chạy PointPillars trong CVAT (thử thách mở). Mỗi hướng tối đa 45 phút.

## 5. Lưu ý trung thực khi báo cáo

- Model học trên KITTI (LiDAR 64 tia, chỉ phía trước, 3 lớp). Trên VF nó bỏ sót nhiều Pedestrian/Cyclist và không biết Truck/Bus. Pre-label là để **giảm** công sửa, không thay người gán nhãn.
- 3 frame tune là ít. Hãy đọc phần leave-one-out của `pp_sweep.py` trước khi tin cấu hình "tốt nhất".
- Mọi box nộp phải được người kiểm. Không nộp thẳng box của model.
