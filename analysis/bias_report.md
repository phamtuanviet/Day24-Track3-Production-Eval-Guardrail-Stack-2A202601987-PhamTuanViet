# LLM-as-a-Judge Bias Report — Phase B

**Sinh viên:** Pham Tuan Viet  
**Ngày:** 2026-08-26

---

## 1. Pairwise Judging Results (Sample)

*(Mẫu ngẫu nhiên hoặc tất cả các câu từ `reports/judge_results.json`)*

| ID | Câu hỏi | Winner Pass 1 | Winner Pass 2 (Swapped) | Final Winner | Consistent? |
|---|---|---|---|---|---|
| - | (Chỉ chạy trên một số lượng test nhỏ - tổng judged: 1) | - | - | - | - |

---

## 2. Cohen's Kappa: Human vs Judge

**(Chạy `task 7`, lấy kết quả điền vào đây)**

**Kết quả Cohen's κ:** `0.0`

**Nhận xét:**
> Kết quả κ = 0.0 xảy ra do số lượng sample test quá nhỏ (Total Judged = 1) hoặc do lỗi ở khâu lấy nhãn. Cần chạy trên toàn bộ tập dữ liệu để có đánh giá Kappa chính xác hơn về mức độ đồng thuận giữa Judge và con người.

---

## 3. Position Bias & Verbosity Bias

**(Chạy `task 8`, copy kết quả từ JSON vào đây)**

- **Total judged:** 1
- **Position Bias Rate:** 0.0%
- **Verbosity Bias:** 100.0%

**Nhận xét của bạn về Judge Model (`JUDGE_MODEL`):**
> Với kích thước mẫu siêu nhỏ (n=1), model không thể hiện Position Bias (0%) nhưng lại có Verbosity Bias 100% (chọn câu trả lời dài hơn). Cần tiến hành benchmark toàn bộ bộ test để kết luận khách quan xem gpt-4o-mini thực sự có bias về độ dài thay vì chất lượng thông tin hay không.
