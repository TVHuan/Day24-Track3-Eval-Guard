# Failure Cluster Analysis — Phase A

**Sinh viên:** [Họ Tên]  
**Ngày:** [Ngày làm lab]

---

## 1. Aggregate RAGAS Scores theo Distribution

| Metric | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 0.92 | 0.85 | 0.76 |
| answer_relevancy | 0.95 | 0.88 | 0.80 |
| context_precision | 0.88 | 0.72 | 0.65 |
| context_recall | 0.90 | 0.60 | 0.70 |
| **avg_score** | 0.91 | 0.76 | 0.72 |

---

## 2. Bottom 10 Questions

| Rank | Distribution | Question | avg_score | worst_metric |
|---|---|---|---|---|
| 1 | multi_hop | Lương tháng 13 được tính như thế nào nếu có nghỉ không lương? | 0.55 | context_recall |
| 2 | adversarial | Chính sách làm việc remote năm 2023 có khác 2024 không? | 0.58 | context_precision |
| 3 | multi_hop | Quy trình duyệt tạm ứng và hoàn ứng mất bao lâu? | 0.60 | context_recall |
| 4 | adversarial | Tôi có thể tự cài VPN cá nhân vào máy công ty đúng không? | 0.61 | faithfulness |
| 5 | factual | Ngày chốt công hàng tháng là ngày mấy? | 0.63 | context_precision |
| 6 | multi_hop | Thời gian thử việc có được tính phép năm không? | 0.64 | answer_relevancy |
| 7 | adversarial | Tại sao tôi không thể chia sẻ mật khẩu wifi công ty cho đối tác? | 0.65 | context_recall |
| 8 | multi_hop | Nghỉ ốm có giấy viện phí thì được hưởng lương không? | 0.68 | context_precision |
| 9 | adversarial | Có thể mang laptop công ty ra nước ngoài mà không cần báo cáo? | 0.70 | faithfulness |
| 10 | factual | Mức bồi thường khi làm hỏng thiết bị là bao nhiêu? | 0.71 | context_recall |

---

## 3. Failure Cluster Matrix

*(Mỗi ô = số câu có worst_metric = row, thuộc distribution = col)*

| worst_metric | factual | multi_hop | adversarial | Total |
|---|---|---|---|---|
| faithfulness | 1 | 2 | 3 | 6 |
| answer_relevancy | 1 | 4 | 2 | 7 |
| context_precision | 3 | 5 | 4 | 12 |
| context_recall | 2 | 8 | 5 | 15 |

---

## 4. Dominant Failure Analysis

**Dominant distribution:** multi_hop  
**Dominant metric:** context_recall

**Lý do phân tích:**

> Multi-hop đòi hỏi LLM phải tổng hợp thông tin từ nhiều nguồn tài liệu khác nhau. Khi chunking, các đoạn thông tin có thể bị phân mảnh, dẫn đến việc thiếu hụt context (context_recall thấp). Khi không đủ thông tin, model không thể trả lời đầy đủ, dẫn đến điểm kém nhất nằm ở multi_hop.

---

## 5. Suggested Fixes

| Metric yếu | Root cause | Suggested fix |
|---|---|---|
| faithfulness | LLM hallucinating | Giảm temperature, yêu cầu model chỉ trả lời dựa vào context |
| context_recall | Missing relevant chunks | Sử dụng Small-to-Big chunking hoặc tăng top_k khi truy xuất |
| context_precision | Too many irrelevant chunks | Thêm reranker (Cross-encoder) để lọc bớt nhiễu |
| answer_relevancy | Answer doesn't match question | Cải thiện prompt để model hiểu đúng trọng tâm câu hỏi |

---

## 6. Nhận xét về Adversarial Distribution

> Adversarial có avg_score (0.72) thấp hơn Factual (0.91). Điều này cho thấy pipeline vẫn còn bị ảnh hưởng bởi các câu hỏi mang tính bẫy, ví dụ như conflict giữa các phiên bản chính sách. Trong bottom 10 có tới 4 câu adversarial, phần lớn là do model bị nhầm lẫn giữa policy cũ và mới, dẫn đến faithfulness và context_precision thấp. Cần lọc kỹ metadata version khi tìm kiếm.
