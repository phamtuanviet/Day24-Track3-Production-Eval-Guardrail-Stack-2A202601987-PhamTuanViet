# Failure Cluster Analysis — Phase A

**Sinh viên:** Pham Tuan Viet  
**Ngày:** 2026-08-26

---

## 1. Aggregate RAGAS Scores theo Distribution

| Metric | factual | multi_hop | adversarial |
|---|---|---|---|
| faithfulness | 0.91 | 0.50 | 0.80 |
| answer_relevancy | 0.79 | 0.68 | 0.60 |
| context_precision | 0.98 | 0.94 | 1.00 |
| context_recall | 0.95 | 0.81 | 0.68 |
| **avg_score** | **0.91** | **0.73** | **0.77** |

---

## 2. Bottom 10 Questions

| Rank | Distribution | Question | avg_score | worst_metric |
|---|---|---|---|---|
| 1 | multi_hop | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | 0.375 | faithfulness |
| 2 | multi_hop | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | 0.375 | faithfulness |
| 3 | adversarial | Bao lâu phải đổi mật khẩu một lần? | 0.417 | faithfulness |
| 4 | adversarial | Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không? | 0.417 | faithfulness |
| 5 | factual | Nam nhân viên được nghỉ bao nhiêu ngày khi vợ sinh con? | 0.458 | faithfulness |
| 6 | multi_hop | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | 0.482 | context_precision |
| 7 | multi_hop | Lương thử việc của nhân viên Junior mức cao nhất là bao nhiêu? | 0.500 | faithfulness |
| 8 | multi_hop | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | 0.542 | answer_relevancy |
| 9 | multi_hop | Nhân viên đi công tác trong nước 2 ngày, ở khách sạn giá 1.500.000 VNĐ/đêm. Công ty thanh toán tối đa bao nhiêu cho tiền khách sạn? | 0.622 | faithfulness |
| 10 | adversarial | Nhân viên thử việc có được hưởng bảo hiểm sức khỏe PVI không? | 0.667 | answer_relevancy |

---

## 3. Failure Cluster Matrix

*(Mỗi ô = số câu có worst_metric = row, thuộc distribution = col)*

| worst_metric | factual | multi_hop | adversarial | Total |
|---|---|---|---|---|
| faithfulness | 3 | 13 | 2 | 18 |
| answer_relevancy | 15 | 2 | 2 | 19 |
| context_precision | 0 | 1 | 0 | 1 |
| context_recall | 2 | 4 | 6 | 12 |

---

## 4. Dominant Failure Analysis

**Dominant distribution:** factual  
**Dominant metric:** answer_relevancy

**Lý do phân tích:**

> Distribution 'factual' có nhiều failure nhất về answer_relevancy. Điều này cho thấy mặc dù retrieve đúng context (thể hiện ở precision và recall đều cao > 0.95), câu trả lời được sinh ra bởi LLM chưa trả lời trực tiếp hoặc dư thừa thông tin so với câu hỏi thực tế.
> Ngoài ra, 'multi_hop' có điểm faithfulness rất thấp (0.50), chứng tỏ khi cần tổng hợp từ nhiều nguồn, LLM rất dễ bị ảo giác (hallucination) thay vì bám sát vào context.

---

## 5. Suggested Fixes

| Metric yếu | Root cause | Suggested fix |
|---|---|---|
| faithfulness | LLM hallucinating khi tổng hợp nhiều tài liệu (multi_hop) | Sử dụng prompt strict hơn, yêu cầu LLM trích dẫn nguyên văn trước khi trả lời. |
| context_recall | Missing relevant chunks ở adversarial | Tăng số lượng top_k hoặc cải thiện embedding có semantic tốt hơn để chịu nhiễu. |
| context_precision | Too many irrelevant chunks | Áp dụng re-ranking model mạnh hơn. (Đã tốt trong lab này). |
| answer_relevancy | Answer không sát trọng tâm câu hỏi (factual) | Thêm instruction để LLM trả lời ngắn gọn, đúng trọng tâm câu hỏi, loại bỏ râu ria. |

---

## 6. Nhận xét về Adversarial Distribution

> Điểm avg_score của adversarial (0.77) tuy thấp hơn factual (0.91) nhưng cao hơn multi_hop (0.73).
> Điều này có thể do test set chưa đủ khó hoặc pipeline có khả năng chịu lỗi (noise) nhất định ở mặt retrieval (context_precision = 1.0). Tuy nhiên, context_recall thấp (0.68) cho thấy các prompt lắt léo vẫn làm hệ thống bị miss một số chunk quan trọng.
