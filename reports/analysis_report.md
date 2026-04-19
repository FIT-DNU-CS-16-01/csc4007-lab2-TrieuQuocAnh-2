# Lab 2 Analysis Report (IMDB)

## 1. Data audit
- Số lượng mẫu đã dùng: ~50,000 reviews (25k train / 25k test chuẩn IMDB)
- Phân bố nhãn positive / negative:positive: ~50% negative: ~50%
- Độ dài review điển hình (median, p95):
  median: ~230–250 tokens
  p95: ~800–1000 tokens
- Không Có missing / empty text 
- Có tồn tại duplicate nhưng tỉ lệ thấp
- 3 quan sát đáng chú ý về dữ liệu IMDB:
  Review thường chứa cảm xúc mạnh (great, terrible, boring, amazing)
  Nhiều câu có phủ định phức tạp (“not bad”, “not really good”)
  Có hiện tượng sarcasm (mỉa mai) → khó cho mô hình tuyến tính
## 2. Preprocessing design
**Các bước làm sạch đã sử dụng:**
- Loại bỏ các ký tự đặc biệt không cần thiết (ngoại trừ dấu câu quan trọng).
- Chuyển toàn bộ văn bản về chữ thường để giảm số lượng từ vựng.
- Loại bỏ khoảng trắng thừa, chuẩn hóa dấu cách.

**Về dấu câu:**
giữ lại các dấu câu cơ bản (., !, ?) vì chúng mang thông tin cảm xúc quan trọng trong review (ví dụ: "!" thể hiện cảm xúc mạnh, "..." thể hiện sự lưỡng lự hoặc mỉa mai). Nếu loại bỏ hoàn toàn dấu câu, mô hình có thể mất tín hiệu về sắc thái cảm xúc.

**Về số:**
không thay thế số bằng `<NUM>`, vì trong dữ liệu IMDB, số lượng xuất hiện không nhiều và thường không mang ý nghĩa cảm xúc rõ rệt. Nếu thay thế, có thể làm mất đi một số tín hiệu đặc biệt (ví dụ: "10/10", "1 star").

**Bước cố tình không làm:**
không loại bỏ các từ mang sắc thái cảm xúc (như "not", "never", "very") và không loại bỏ stopwords, vì các từ này rất quan trọng để mô hình nhận diện cảm xúc, đặc biệt là các phủ định hoặc nhấn mạnh.

## 3. Experiment comparison
| Run | Text version | Vectorizer | Model                 | ngram | Macro-F1 | Accuracy | Ghi chú |
|-----|-------------|------------|-----------------------|-------|----------|----------|--------|
| 1   | cleaned     | TF-IDF     | Linear SVM            | 1–2   | 0.9100   | 0.9100   | Best   |
| 2   | cleaned     | TF-IDF     | Logistic Regression   | 1–2   | 0.9064   | 0.9064   | Stable |
| 3   | cleaned     | BoW        | Logistic Regression   | 1–2   | 0.8956   | 0.8956   | Worst  |

## 4. Error analysis (>= 10 lỗi)
- Chọn ít nhất 10 mẫu trong `outputs/error_analysis/error_analysis.csv`.
- Gom thành 2–4 nhóm lỗi.
- Gợi ý nhóm lỗi trên IMDB:
  - phủ định / tương phản (`not good`, `although ... still`)
  - cảm xúc trộn lẫn
  - mỉa mai / châm biếm
  - review quá dài, nhiều chi tiết không liên quan
  - mô hình dự đoán rất tự tin nhưng sai

### Bảng ghi lỗi
| ID | True | Pred | Nhóm lỗi | Giải thích ngắn |
|---|---|---|---|---|
|31245|negative|positive| Review dài, nhiều chi tiết không liên quan | Review dài, nhiều thông tin phụ, mô hình khó nắm bắt ý chính |
|37061|	negative|	positive|	Cảm xúc mạnh, mô hình nhầm lẫn|	Review khen ngợi, nhiều cảm xúc tích cực nhưng gán nhãn negative|
|8347|	negative|	positive|	Cảm xúc trộn lẫn|	Review trung tính, có cả khen và chê, mô hình khó xác định cảm xúc chính|
|17596|	positive|	negative|	Phủ định/so sánh|	Có nhiều phủ định, so sánh, mô hình dễ nhầm lẫn ý nghĩa thực sự|
|34543|	negative|	positive|	Mỉa mai/châm biếm|	Review có yếu tố mỉa mai, mô hình không nhận diện được sắc thái|
|4648|	negative|	positive|	Review dài, nhiều chi tiết không liên quan|	Nhiều nhận xét phụ, mô hình không tập trung vào ý chính|
|8659|	negative|	positive|	Mỉa mai/châm biếm|Review khen quá mức, có thể là mỉa mai, mô hình hiểu nhầm là tích cực thật
|36707|	negative|	positive|	Cảm xúc trộn lẫn|	Review có nhiều nhận xét tích cực về kỹ thuật nhưng nhãn là negative
|29148|	negative|	positive|	Cảm xúc trộn lẫn|	Review kể lại trải nghiệm cá nhân, cảm xúc lẫn lộn, mô hình khó xác định|
|36218|	negative|	positive|	Mô hình quá tự tin nhưng sai|	Dự đoán xác suất rất cao nhưng lại sai nhãn, có thể do overfitting|
## 5. Reflection
- Pipeline nào tốt nhất trên IMDB? Vì sao?**
Pipeline tốt nhất là TF-IDF kết hợp Linear SVM (macro-F1 = 0.9100, accuracy = 0.9100). Mô hình này tận dụng được thông tin từ n-gram và khả năng phân tách tuyến tính mạnh của SVM, giúp nhận diện cảm xúc tốt hơn so với Logistic Regression hay BoW.

- Trên IMDB, accuracy và macro-F1 có chênh nhau nhiều không?**
Không, hai chỉ số này gần như bằng nhau (chênh lệch < 0.001). Điều này cho thấy dữ liệu IMDB khá cân bằng giữa hai lớp, mô hình không bị lệch về một phía.

- Nếu chuyển sang dữ liệu lệch lớp hơn, bạn kỳ vọng metric nào sẽ phản ánh tốt hơn? Vì sao?**
Khi dữ liệu lệch lớp, macro-F1 sẽ phản ánh tốt hơn vì nó tính trung bình F1-score trên từng lớp, không bị ảnh hưởng bởi tỉ lệ mẫu giữa các lớp như accuracy. Accuracy có thể cao nhưng mô hình bỏ qua lớp thiểu số.

- Một cải tiến bạn muốn thử ở Lab 3 là gì?**
Thử các mô hình phi tuyến tính (như Random Forest, XGBoost) hoặc thử fine-tune các mô hình transformer (BERT, DistilBERT) để cải thiện khả năng nhận diện sắc thái cảm xúc phức tạp, đặc biệt là các trường hợp mỉa mai, phủ định hoặc review dài.
