# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Ngô Hoàng Thụy Khuê |
| MSSV | 2A202603017 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/kelsingo/K4-L3-DAY21-NgoHoangThuyKhue-2A202603017-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Tôi chọn lần chạy 3 vì `f1_score=0.7149` là kết quả cao nhất. Lần chạy 2 có accuracy cao nhất (`0.8780`) nhưng F1 chỉ đạt `0.7109`; hai kết quả không trùng nhau cho thấy accuracy có thể che khuất chất lượng nhận diện lớp thu nhập cao. Khi tăng từ 50 cây, `learning_rate=0.05` lên 100 cây, `learning_rate=0.1`, F1 tăng mạnh từ `0.6051` lên `0.7109`. Tăng tiếp lên 200 cây và độ sâu 5 chỉ làm F1 tăng khoảng 0.004 trong khi accuracy giảm 0.004. Đây là đánh đổi giữa khả năng học lớp dương với độ phức tạp, thời gian huấn luyện và nguy cơ overfit.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Trong 44.722 mẫu huấn luyện có 11.084 mẫu thu nhập trên 50K (24,78%); holdout có 124/500 mẫu dương (24,8%). Do mất cân bằng, mô hình luôn đoán “thu nhập thấp” vẫn đạt accuracy `376/500 = 0.752`, dù không phát hiện mẫu dương nào và có F1 bằng 0. F1 là trung bình điều hòa của precision và recall, vì vậy phản ánh đồng thời dự đoán dương sai và số mẫu dương bị bỏ sót. Tôi dùng `f1_score` nhị phân mặc định để đo trực tiếp `target=1`. `average="weighted"` sẽ bị lớp đa số chi phối, còn `average="macro"` trung bình hóa hai lớp; cả hai không phù hợp với quality gate tập trung vào lớp thu nhập cao.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Không cài được MLflow | Python 3.14 không tương thích `pyarrow` | Dùng Python 3.12 và pin các dependency. |
| DVC lỗi S3 | Sai bucket và thiếu quyền IAM | Sửa remote, cấp `ListBucket`, `GetObject`, `PutObject`. |
| Train/Release thất bại | `mlruns` chứa path local; cổng 22 chỉ cho IP cá nhân | Bỏ track `mlruns/` và cho GitHub runner truy cập SSH. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7289 | 0.8780 |

**Nhận xét:** Khi tăng từ 22.361 lên 44.722 mẫu, F1 tăng 0.0140 và accuracy tăng 0.0040. Batch mới có phân bố gần giống batch đầu nên cải thiện không lớn, nhưng giúp mô hình khái quát hóa lớp dương tốt hơn.
