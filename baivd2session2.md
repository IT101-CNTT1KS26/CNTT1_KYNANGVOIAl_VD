# PHÂN TÍCH CƠ CHẾ IN-CONTEXT LEARNING VÀ THIẾT KẾ PROMPT FEW-SHOT — TECHSHOP



## PHẦN 1: PHÂN TÍCH CƠ CHẾ IN-CONTEXT LEARNING (HỌC TRONG NGỮ CẢNH)

### 1. Nguyên nhân thất bại của phương pháp Zero-shot

Trong bài toán phân loại cảm xúc đánh giá sản phẩm của TechShop, câu lệnh **Zero-shot** (đưa thẳng câu review vào yêu cầu phân loại mà không kèm ví dụ mẫu) đã đưa ra kết quả sai lệch hoàn toàn (`TÍCH CỰC` cho một phản hồi chê máy quá nóng 98°C). Nguyên nhân cốt lõi là:

* **Bám vào nghĩa hiển ngôn (Surface Lexical Meaning):** Mô hình ở chế độ Zero-shot chỉ quét qua các từ khóa đơn lẻ có giá trị biểu đạt cảm xúc tích cực ở bề mặt (như *"đẹp mê ly"*, *"đầm tay"*, *"khỏi cần bật lò sưởi"*).
* **Thiếu khả năng suy luận ngữ dụng (Pragmatic Inference):** AI không tự nhận diện được tầng nghĩa ẩn ý, sự mỉa mai (sarcasm) hay bối cảnh kỹ thuật cực đoan (nhiệt độ CPU 98°C là ngưỡng nguy hiểm cho thiết bị).

### 2. Bản chất hoạt động của In-Context Learning (ICL)

**In-Context Learning** là khả năng mô hình ngôn ngữ lớn (LLM) hiểu và thực hiện nhiệm vụ mới thông qua các ví dụ được cung cấp trực tiếp bên trong cửa sổ ngữ cảnh (Context Window) của câu lệnh, **hoàn toàn không cần cập nhật trọng số mô hình (No Weight Updates / No Fine-tuning)**.

* **Cơ chế Dự đoán Token tiếp theo (Next-Token Prediction):** LLM hoạt động bằng cách dự đoán từ hoặc chuỗi token tiếp theo có xác suất xuất hiện cao nhất dựa trên toàn bộ ngữ cảnh đầu vào. Khi ta cung cấp các cặp ví dụ mẫu (Input ➔ Output), mô hình sử dụng chính các ví dụ đó làm "khuôn mẫu suy luận" (pattern matching).
* **Kích hoạt tri thức tiềm ẩn (Semantic Priming):** Bản thân LLM đã được huấn luyện với lượng dữ liệu khổng lồ trên Internet nên nó có sẵn khả năng hiểu văn phong mỉa mai. Tuy nhiên, ở chế độ Zero-shot, xác suất cho kết quả bề mặt cao hơn. Khi có các ví dụ Few-shot, cơ chế tự chú ý (**Attention Mechanism**) của AI sẽ lập tức chuyển đổi trạng thái, kích hoạt các biểu diễn không gian tiềm ẩn (latent space) liên quan đến sự mỉa mai, từ đó hiểu đúng ý nghĩa sâu xa của câu đánh giá mà không cần can thiệp huấn luyện lại mô hình.

---

## PHẦN 2: THIẾT KẾ PROMPT FEW-SHOT HOÀN CHỈNH CHO TECHSHOP

Dưới đây là một Prompt Few-shot hoàn chỉnh, tích hợp **3 cặp ví dụ mẫu** nhằm huấn luyện AI nhận diện chính xác các đánh giá mang sắc thái mỉa mai, khen đểu đặc thù trong lĩnh vực bán lẻ thiết bị công nghệ.

```text
[Context / Character]
Bạn là Trợ lý AI chuyên phân tích cảm xúc đánh giá khách hàng (Sentiment Analysis) tại hệ thống thương mại điện tử TechShop. Khách hàng của TechShop thường xuyên sử dụng văn phong mỉa mai, "khen đểu" (sarcasm) để phản ánh các lỗi kỹ thuật nghiêm trọng. Bạn cần nhìn thấu nghĩa ẩn ý thay vì chỉ đánh giá dựa trên từ ngữ tích cực ở bề mặt.

[Examples / Few-shot Learning]
Dưới đây là các ví dụ mẫu phân tích cảm xúc thực tế tại TechShop:

---
Ví dụ 1 (Lỗi Nhiệt độ / Quá nhiệt):
- Input: "Máy thiết kế đẹp mê ly, cầm rất đầm tay, mùa đông ôm máy chơi game thì khỏi cần bật lò sưởi vì nhiệt độ CPU lên hẳn 98 độ C!"
- Output: {
    "sentiment": "TIÊU CỰC",
    "hidden_issue": "Quá nhiệt nghiêm trọng (Overheating)",
    "reason": "Sử dụng từ ngữ khen hình thức nhưng thực chất mỉa mai mức nhiệt độ CPU cực kỳ nguy hiểm (98 độ C)."
  }

Ví dụ 2 (Lỗi Pin / Hao nguồn):
- Input: "Pin trâu khủng khiếp thật sự, rút sạc 100% từ sáng đến trưa không đụng vào máy mà đã tự sập nguồn ngắt điện, quá tuyệt vời cho một thiết bị di động!"
- Output: {
    "sentiment": "TIÊU CỰC",
    "hidden_issue": "Lỗi hao pin / Sập nguồn bất thường",
    "reason": "Dùng từ 'pin trâu' và 'tuyệt vời' để mỉa mai việc máy hao nguồn nặng và sập nguồn khi không sử dụng."
  }

Ví dụ 3 (Lỗi Thời gian giao hàng / Chậm trễ):
- Input: "Shop giao hàng nhanh như vận tốc ánh sáng, đặt hàng từ năm ngoái mà năm nay mới nhận được hàng, dịch vụ quá đỉnh cao!"
- Output: {
    "sentiment": "TIÊU CỰC",
    "hidden_issue": "Giao hàng chậm trễ nghiêm trọng",
    "reason": "Dùng cụm từ phóng đại 'nhanh như vận tốc ánh sáng' đi kèm khoảng thời gian vô lý để mỉa mai sự chậm trễ của đơn vị vận chuyển."
  }
---

[Request]
Hãy áp dụng phương pháp phân tích sâu sắc từ các ví dụ mẫu trên để đánh giá cảm xúc của đánh giá khách hàng mới nhất dưới đây.

[Input Data cần xử lý]
"Màn hình hiển thị sắc nét thật đấy, chỉ tiếc là cứ mở ứng dụng lên là đơ máy, khởi động lại 5 lần vẫn đứng hình, đúng là hàng cao cấp có khác!"

[Type of Output & Format]
Kết quả trả về PHẢI ở dạng mã JSON thuần túy (được đặt trong khối ```json ... ```) tuân thủ chính xác cấu trúc như các ví dụ mẫu.

```

---

##  HƯỚNG DẪN NỘP BÀI TRÊN GITHUB

Để nộp bài tập này lên hệ thống của lớp, bạn hãy tạo một Repository trên GitHub với tên gọi tuân thủ cấu trúc chuẩn:

* **Tên Repository:** `[TênLớp]_[MônHọc]_Session02_Ex05`
*(Ví dụ: `CNTT1_NHAPMONCNTT_Session02_Ex05`)*

**Cấu trúc tệp tin bên trong repository:**

```text
[TênLớp]_[MônHọc]_Session02_Ex05/
├── README.md              # Nội dung báo cáo phân tích và prompt chi tiết ở trên
└── solution-prompt.txt    # Tệp chứa nội dung Prompt Few-shot để tiện copy/test

```