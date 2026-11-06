# Báo cáo cá nhân — Day 01

SEED 3164. Nguồn UCI, 5,171 tin sau khử trùng. Chia 3,621 train, 775 validation và 775 test. Độ trễ đo trên CPU Windows từ SMS thô đến nhãn.

## 1. Bằng chứng

- [Notebook đã chạy](day01_lab.ipynb)
- [Kết quả JSON](lab1_report.json)
- [Bản trình bày 5 phép tính](hand_calc.jpg) (đánh máy).

## 2. Kết quả chính

| Mô hình | F1 test | F1 SUB 40 | p95 ms/tin | USD/1.000 tin |
| --- | ---: | ---: | ---: | ---: |
| BoW + LogReg | 0.873 | 0.788 | 0.052 | 0.000 |
| BoW + MLP | 0.958 | 0.974 | 0.275 | 0.000 |
| Encoder (MiniLM) + kNN | 0.914 | 0.895 | 27.966 | 0.000 |
| Encoder (MiniLM) + LogReg | 0.963 | 0.919 | 21.034 | 0.000 |
| Encoder (MiniLM) + MLP head | 0.968 | 0.919 | 32.730 | 0.000 |
| LLM nhỏ zero-shot | — | 0.667 | 4189.542 | 0.000 |
| LLM lớn zero-shot | — | 0.655 | 2333.051 | 0.000 |
| LLM nhỏ few-shot | — | 0.000 | 3645.591 | 0.000 |

F1 SUB 40 là 0.667 với model nhỏ zero-shot, 0.655 với model lớn zero-shot và 0.000 với model nhỏ few-shot. Đây là kết quả của hai model SmolLM2 chạy trên máy này. Số câu trả lời không hợp lệ theo thứ tự trên là 5, 0 và 39 trên 40 tin. Chi phí API cục bộ là 0. Chi phí phần cứng và điện chưa được đo.

## 3. Dự đoán và thực tế

- Dự đoán few-shot tăng F1 khoảng 0,10. Thực tế giảm từ 0.667 xuống 0.000. Có 39/40 câu trả lời few-shot không đọc được thành nhãn.
- Dự đoán BoW và Encoder + LogReg đúng 3/6 và 5/6 trên sáu tin ngụy trang. Thực tế cả hai đúng 5/6. Cả hai bỏ sót tin hối thúc xác nhận thông tin để giữ điểm thưởng.
- Dự đoán LLM lớn tăng F1 khoảng 0,10 và có p95 cao hơn. F1 thực tế là 0.655 so với 0.667. p95 của model lớn là 2333 ms, thấp hơn 4190 ms của model nhỏ trong lần chạy này.

## 4. Phân tích 50 ca khó

2. BoW + MLP giảm nhiều nhất, từ 0.974 trên SUB xuống 0.359 ở 50 ca khó. Ở nhóm prompt injection, accuracy của prompt thường là 33.3%, prompt có thẻ `<sms>` là 16.7%. Các nhóm giảm khi phòng thủ gồm Thao túng / lừa đảo, Spam giọng tự nhiên, Né từ khoá.

- Ca #8 có nhãn ham, mô hình đoán spam. 'Cash prize' nhưng là sự kiện nội bộ
- Ca #11 có nhãn spam, mô hình đoán ham. Giọng thân mật, giả làm người quen, thực chất là khuyến mãi

## 5. Thử thách cá nhân

Giữ cùng dữ liệu, seed và 30 epoch, rồi thay độ rộng lớp ẩn. Mạng 4 nơ-ron có 8,009 tham số, F1 0.973 và train trong 1.85 s. Mạng 64 nơ-ron có 128,129 tham số, F1 0.958 và train trong 2.26 s. Mạng 512 nơ-ron có 1,025,025 tham số, F1 0.959 và train trong 9.68 s. Trong lần chạy này, mạng 4 nơ-ron nhanh nhất và có F1 cao nhất. Tăng độ rộng lên 512 làm tăng chi phí mà F1 thấp hơn. Cần thử thêm seed trước khi kết luận về độ rộng tối ưu.

## 6. Phương án cho sản phẩm

Chọn BoW + MLP cho luồng chính. F1 toàn test là 0.958, p95 là 0.27 ms/tin và thông lượng theo lô là 54581 tin/s trên máy thử. Encoder (MiniLM) + MLP head có F1 cao hơn (0.968) nhưng xử lý 103 tin/s, dưới nhu cầu trung bình khoảng 116 tin/s. Độ trễ và thông lượng đo riêng, nên cần kiểm tra cả hai khi hệ thống chịu tải. Với giả định 100 token vào và 1 token ra như bài tính tay, giá Luna trong đề cho 10 triệu tin/ngày là khoảng 3.150 USD/30 ngày. Router có thể dùng BoW + MLP cho tin chắc chắn và chuyển ca gần ngưỡng sang embedding. Cách này tập trung chi phí vào tin khó, nơi BoW + MLP mất nhiều F1 nhất. Các cách qua ngưỡng p95 50 ms trong lần chạy gồm BoW + LogReg, BoW + MLP, Encoder (MiniLM) + kNN, Encoder (MiniLM) + LogReg, Encoder (MiniLM) + MLP head.

## 7. AI log và điều đã học

Bản làm lại dùng Codex để đọc đề, điền TODO, ghi dự đoán trước khi chạy và phân tích kết quả thực nghiệm. Mã được kiểm tra qua các ô có sẵn trong notebook. Sai lệch lớn nhất là few-shot. Tôi dự đoán F1 tăng khoảng 0,10 nhưng kết quả giảm từ 0.667 xuống 0.000. Với mô hình nhỏ, cần kiểm tra tỷ lệ đầu ra hợp lệ trước khi kết luận ví dụ mẫu giúp phân loại tốt hơn.
