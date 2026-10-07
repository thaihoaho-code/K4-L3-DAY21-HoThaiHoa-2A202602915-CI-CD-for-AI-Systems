# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Hồ Thái Hòa |
| MSSV | 2A202602915 |
| Lớp / Khóa | K4 |
| Repo GitHub | [K4-L3-DAY21-HoThaiHoa-2A202602915-CI-CD-for-AI-Systems](https://github.com/thaihoaho-code/K4-L3-DAY21-HoThaiHoa-2A202602915-CI-CD-for-AI-Systems) |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 đạt F1 lớp dương cao nhất (0.7149) nên được chọn. Lần chạy 1 có accuracy cao nhất (0.8780), cho thấy thứ hạng theo hai chỉ số khác nhau. So với lần chạy 1, lần chạy 3 dùng nhiều cây và cây sâu hơn; F1 tăng 0.0040 nhưng accuracy giảm 0.0040. Lần chạy 2 có F1 thấp nhất (0.6051). Vì nhiều tham số thay đổi đồng thời, các lần chạy chưa tách riêng ảnh hưởng của từng tham số.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Bộ dữ liệu Adult mất cân bằng: lớp thu nhập trên 50K chiếm khoảng 24,8%, còn lớp dưới hoặc bằng 50K chiếm 75,2%. Mô hình luôn dự đoán thu nhập thấp vẫn đạt accuracy khoảng 75,2% nhưng bỏ sót toàn bộ lớp dương. Vì thế accuracy có thể che khuất hiệu quả kém ở lớp thiểu số. F1 của lớp dương kết hợp precision và recall, phản ánh cả dự đoán dương sai lẫn trường hợp thu nhập cao bị bỏ sót. Quality gate dùng `f1_score` với `pos_label=1`, ngưỡng 0.65. Không dùng `average="weighted"` hoặc `average="macro"` vì cần đo riêng lớp dương, không gộp điểm F1 của hai lớp.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| `dvc pull` lỗi xác thực | Bucket đã bị xóa. | Khôi phục bucket hoặc tạo bucket mới rồi `dvc push` lại dữ liệu. |
| MLflow không tạo run | FileStore thiếu metadata. | Đặt `MLFLOW_TRACKING_URI=sqlite:///mlflow.db` cho job train. |
| API không mở cổng 8080 | VM thiếu tên bucket và dùng scikit-learn khác phiên bản train. | Sửa biến systemd và đồng bộ phiên bản thư viện theo `requirements.txt`. |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Sau khi bổ sung dữ liệu, F1 tăng 0.0205 và accuracy tăng 0.0080. Kết quả cho thấy mô hình trong lần chạy này cải thiện trên cả hai chỉ số; quality gate F1 0.65 vẫn đạt.
