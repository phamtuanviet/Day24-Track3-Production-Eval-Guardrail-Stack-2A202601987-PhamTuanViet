# CI/CD Blueprint: RAG Eval + Guardrail Stack

**Sinh viên:** Pham Tuan Viet  
**Ngày:** 2026-08-26

---

## Guard Stack Architecture

```
User Input
    │
    ▼ (~?ms P95)
[Presidio PII Scan]
    │ block if: VN_CCCD / VN_PHONE / EMAIL detected
    │ action:   return 400 + "PII detected in query"
    ▼ (~?ms P95)
[NeMo Input Rail]
    │ block if: off-topic / jailbreak / prompt injection
    │ action:   return 503 + refuse message
    ▼
[RAG Pipeline (Day 18)]
    │ M1 Chunk → M2 Search → M3 Rerank → GPT-4o-mini
    ▼
[NeMo Output Rail]
    │ flag if:  PII in response / sensitive content
    │ action:   replace with safe response
    ▼
User Response
```

---

## Latency Budget

*(Điền từ kết quả Task 12 — measure_p95_latency())*

| Latency Budget | Phase C (P50) | Phase C (P95) | Phase C (P99) | Budget OK? |
|---|---|---|---|---|
| Presidio PII | 960ms | 2259ms | 2259ms | No |
| NeMo Input Rail | 85ms | 94ms | 94ms | Yes |
| RAG Pipeline | 400ms | 800ms | 1500ms | Yes |
| NeMo Output Rail | 40ms | 100ms | 200ms | Yes |
| **Total Guard** | 1043ms | **2344ms** | 2344ms | **No** |

**Budget OK?** [ ] Yes / [x] No  
**Comment:** Vượt budget (Total P95 = 2344ms > 500ms). Bottleneck chính là **Presidio PII** (~2259ms). Cần tối ưu bằng cách: dùng Regex/Rule-based thay cho mô hình NLP nặng, dùng các model NER nhỏ hơn, hoặc tối ưu infrastructure cho Presidio.

---

## CI/CD Gates (phải pass trước khi merge to main)

```yaml
# .github/workflows/rag_eval.yml
- name: RAGAS Quality Gate
  run: python src/phase_a_ragas.py
  env:
    MIN_FAITHFULNESS: 0.75
    MIN_AVG_SCORE: 0.65

- name: Guardrail Gate
  run: pytest tests/test_phase_c.py -k "test_adversarial_suite_pass_rate"
  # phải ≥ 15/20 (75%)

- name: Latency Gate
  run: python -c "from src.phase_c_guard import measure_p95_latency; ..."
  # P95 total < 500ms
```

---

## Monitoring Dashboard (production)

| Metric | Alert Threshold | Action |
|---|---|---|
| RAGAS faithfulness (daily sample) | < 0.70 | Page on-call |
| Adversarial block rate | < 80% | Review new attack patterns |
| Guard P95 latency | > 600ms | Scale NeMo model |
| PII detected count | spike >10/hour | Security alert |

---

## Kết quả thực tế từ Lab

| | Kết quả |
|---|---|
| RAGAS avg_score (50q) | 0.81 |
| Worst metric | answer_relevancy |
| Dominant failure distribution | factual |
| Cohen's κ | 0.0 |
| Adversarial pass rate | 4 / 20 |
| Guard P95 latency | 2344 ms |

---

## Nhận xét & Cải tiến

> Hệ thống Guardrails đang hoạt động chưa hiệu quả (Adversarial pass rate thấp: 4/20). Cần tinh chỉnh rule của NeMo Guardrails để xử lý jailbreak và prompt injection tốt hơn.
> P95 latency vượt mức an toàn khá nhiều (2344ms > 500ms). Bottleneck nằm ở Presidio (NER). Cần áp dụng giải pháp quét PII nhẹ hơn hoặc regex-based để giảm độ trễ.
> Cần liên tục cập nhật bộ adversarial set để cải thiện khả năng phòng vệ trong tương lai.
