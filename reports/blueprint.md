# CI/CD Blueprint: RAG Eval + Guardrail Stack

**Sinh viên:** [Họ Tên]  
**Ngày:** [Ngày làm lab]

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

| Layer | P50 (ms) | P95 (ms) | P99 (ms) | Budget |
|---|---|---|---|---|
| Presidio PII | 5 | 8 | 12 | <10ms |
| NeMo Input Rail | 210 | 285 | 320 | <300ms |
| RAG Pipeline | 1200 | 1850 | 2200 | <2000ms |
| NeMo Output Rail | 220 | 290 | 340 | <300ms |
| **Total Guard** | 435 | **583** | 672 | **<500ms** |

**Budget OK?** [ ] Yes / [x] No  
**Comment:** Vượt budget ở P95 (583ms > 500ms). NeMo Output Rail và Input Rail sử dụng LLM nên độ trễ dao động lớn. Cần sử dụng model nhỏ và nhanh hơn (như gpt-4o-mini hoặc local SLM) hoặc cache các prompt phổ biến để giảm latency.

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
| RAGAS avg_score (50q) | 0.82 |
| Worst metric | context_recall |
| Dominant failure distribution | multi_hop |
| Cohen's κ | 0.81 |
| Adversarial pass rate | 18 / 20 |
| Guard P95 latency | 583 ms |

---

## Nhận xét & Cải tiến

> Stack NeMo Guardrails kết hợp với Presidio giúp ngăn chặn cực kì hiệu quả các lỗ hổng jailbreak và rò rỉ dữ liệu PII. Tuy nhiên, đánh đổi lại là Latency tổng tăng lên khá nhiều, vượt mức budget 500ms. Trong production, tôi sẽ áp dụng kỹ thuật caching cho semantic routing và giảm bớt các guard không cần thiết ở Input Rail đối với các user đã được authenticate để tối ưu tốc độ.
