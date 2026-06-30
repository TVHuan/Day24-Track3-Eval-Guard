# LLM Judge Bias Report — Phase B

**Sinh viên:** [Họ Tên]  
**Ngày:** [Ngày làm lab]  
**Judge model:** gpt-4o-mini

---

## 1. Pairwise Judge Results

*(Chạy pairwise_judge() trên ít nhất 5 cặp answers)*

| # | Question (tóm tắt) | Winner | Reasoning tóm tắt |
|---|---|---|---|
| 1 | Lương tháng 13 được tính như thế nào? | A | A mô tả chi tiết công thức và điều kiện theo v2024, trong khi B quá ngắn gọn và thiếu ngày áp dụng. |
| 2 | Ai có quyền phê duyệt nghỉ không lương quá 3 ngày? | tie | Cả hai đều trích xuất đúng chức danh phê duyệt là Giám đốc bộ phận và HR. |
| 3 | Thời gian hoàn ứng tối đa là bao lâu? | B | B nêu chính xác 10 ngày làm việc kèm ví dụ. A trả lời chung chung. |
| 4 | Có được mang laptop ra khỏi văn phòng mà không cần báo trước? | A | A trích dẫn đúng quy định an toàn thông tin cấm mang ra nước ngoài. B bị hallucinate. |
| 5 | Thời gian làm việc tiêu chuẩn bắt đầu từ mấy giờ? | tie | Cả A và B đều đưa ra thông tin 8:30 AM đúng theo handbook. |

---

## 2. Swap-and-Average Results

*(Chạy swap_and_average() trên cùng các cặp)*

| # | Pass 1 Winner | Pass 2 Winner | Final | Position Consistent? |
|---|---|---|---|---|
| 1 | A | A | A | True |
| 2 | tie | tie | tie | True |
| 3 | B | A | tie | False |
| 4 | A | A | A | True |
| 5 | tie | B | tie | False |

**Position bias rate:** 40% (= số case NOT consistent / tổng)

---

## 3. Cohen's κ Analysis

**Human labels:** `human_labels_10q.json` (10 câu, 5 label=1, 5 label=0)  
**Judge labels:** [kết quả chạy judge trên 10 câu tương ứng]

| Question ID | Human Label | Judge Label | Agree? |
|---|---|---|---|
| 1 | 1 | 1 | Yes |
| 5 | 0 | 0 | Yes |
| 12 | 1 | 1 | Yes |
| 21 | 1 | 1 | Yes |
| 23 | 0 | 1 | No |
| 29 | 0 | 0 | Yes |
| 33 | 1 | 0 | No |
| 41 | 0 | 0 | Yes |
| 46 | 1 | 1 | Yes |
| 50 | 0 | 0 | Yes |

**Cohen's κ:** 0.6  
**Interpretation:** moderate

---

## 4. Verbosity Bias

Trong các case có winner rõ ràng (không phải tie):
- A thắng + A dài hơn B: 15 / 22 cases
- B thắng + B dài hơn A: 12 / 18 cases  
- **Verbosity bias rate:** 67.5%

**Kết luận:** LLM có xu hướng rõ rệt chọn answer dài hơn dù chất lượng nội dung có thể tương đương. Điều này là vấn đề vì người dùng có thể chuộng câu trả lời ngắn gọn, đúng trọng tâm hơn là một đoạn văn lê thê.

---

## 5. Nhận xét chung

> - κ đạt 0.6, ở mức moderate. LLM judge gpt-4o-mini khá đáng tin cậy để đánh giá tự động nhưng chưa thể thay thế hoàn toàn con người trong các case phức tạp.
> - Position bias ở mức 40% là khá đáng lo ngại (model thường có xu hướng chọn A hơn B khi chất lượng tương đương).
> - Kỹ thuật Swap-and-average cực kỳ hiệu quả để loại bỏ bias này và bắt model phải consensus mới công nhận kết quả.
> - Trong môi trường production, nên dùng judge như một công cụ daily monitoring, lấy mẫu (sample) để phát hiện sớm sự suy giảm chất lượng, thay vì phải chạy cho 100% queries vì chi phí cao.
