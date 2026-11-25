# Nộp bài Day 14 — Pre-label 3D bằng PointPillars

Bài **cá nhân**. Mỗi học viên nộp **link tới commit bài nộp** trong repo GitHub của mình lên LMS.

## 1. Repo nộp

- Tên repo: `K5-L23-DAY14-HoVaTen-MSSV-PointPillarsPrelabel`, viết không dấu, không khoảng trắng.
  Ví dụ: `K5-L23-DAY14-NguyenVanAn-2A20260000-PointPillarsPrelabel`.
- Tạo repo mới trên GitHub với tên trên, rồi đẩy thư mục lab đã làm lên đó:

  ```bash
  git remote set-url origin https://github.com/<tài-khoản>/K5-L23-DAY14-HoVaTen-MSSV-PointPillarsPrelabel.git
  git add d14_<ID> report
  git commit -m "Day 14: bài nộp"
  git branch -M main
  git push -u origin main
  git rev-parse HEAD   # SHA của commit bài nộp
  ```
- Nếu để repo ở chế độ Private, mời tài khoản GitHub của coach (công bố trên lớp) vào repo trước khi nộp.

## 2. Các file phải có trong repo

| File | Sinh ra ở bước | Dùng để chấm |
|---|---|---|
| `d14_<ID>/frames_A.txt`, `d14_<ID>/frames_B.txt` | 1 | Đúng SEED, đúng frame (có dòng `seed_tag=`). **Không sửa tay** |
| `d14_<ID>/pred_A.json` | 2 | Pre-label gốc của tập A |
| `d14_<ID>/ref_A.json` | 5 | Nhãn tham chiếu bạn đã sửa trên CVAT |
| `d14_<ID>/sweep.csv`, `d14_<ID>/best_config.json` | 6 | Kết quả tune |
| `d14_<ID>/pred_B.json` | 7 | Pre-label tập B bằng cấu hình đã tune |
| `d14_<ID>/compare/*.jpg` | 8 | Ảnh trước/sau, ít nhất ảnh bạn dùng trong báo cáo |
| `d14_<ID>/ref_B.json` | 9 | **Nhãn 12 frame tập B đã sửa xong** (sản phẩm chính) |
| `report/individual/<MSSV>_<HoTen>.md` | 5b, 8 | Báo cáo cá nhân, theo template [`report/individual/MSSV_HoTen.md`](report/individual/MSSV_HoTen.md) |

Không commit: thư mục dữ liệu `Data-VF`, `third_party/`, mật khẩu CVAT, file `.env`. Thư mục ảnh `d14_<ID>/png_*` đã được bỏ qua sẵn trong `.gitignore`.

## 3. Nơi nộp và deadline

- Nộp lên LMS, bằng tài khoản của chính bạn, **link commit** dạng `https://github.com/<tài-khoản>/<repo>/commit/<SHA>` (SHA lấy từ `git rev-parse HEAD` sau khi push). Coach chấm đúng commit này, không chấm nhánh `main` lúc mở repo.
- Sửa bài trước deadline: push commit mới rồi cập nhật link commit trên LMS. Không force-push, không xóa commit đã nộp.
- Deadline: **12:00 trưa ngày hôm sau ngày lab** (giờ Việt Nam, GMT+7).
- Nộp muộn và sửa bài sau deadline: xem [RULES.md](RULES.md).

## 4. Tự kiểm trước khi nộp

- [ ] Tên repo đúng mẫu ở mục 1, link mở được (Private thì đã mời coach).
- [ ] Link trên LMS là link commit có SHA, mở ra đúng commit cuối của bạn.
- [ ] Mở repo trên GitHub, thấy đủ các file ở mục 2.
- [ ] `frames_A.txt` và `frames_B.txt` vẫn có dòng `seed_tag=` do lệnh bước 1 sinh ra.
- [ ] `ref_B.json` có đủ 12 frame của `frames_B.txt`, được kéo về **sau khi** đã Save trên CVAT.
- [ ] Đã chạy `python pp_prelabel.py --data <Data-VF> --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --gt d14_<ID>/ref_B.json` và ghi số xóa/thêm/sửa vào báo cáo.
- [ ] Đã commit và push mục 1 báo cáo (dự đoán 5b) **trước** khi chạy sweep, ví dụ `git commit -m "5b: dự đoán"`.
- [ ] Báo cáo có 1 ảnh compare và phần đối chiếu.
