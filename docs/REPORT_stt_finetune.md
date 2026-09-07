# STT 파인튜닝 실험 기록

> 자동 생성 · 2026-07-27 09:40

## 요약

- 실험 횟수: **11회**
- 최고 성능: **exp8_capacity_r64_at_lr2e-4** — CER 11.7% → **5.43%** (53.6% 개선)

## 실험 비교

| # | 실험 | 변경점 | CER 전 | CER 후 | 개선 | 시간 |
|---|---|---|---|---|---|---|
| 1 | exp1_lora_r32_ep3 | lora_r=32, target=q_proj,v_proj, lr=0.001, epochs=3, train_n=8000, eval=없음 | 11.7% | **6.92%** | 40.8% | 0.0분 |
| 2 | exp2_eval_bestckpt | lora_r=32, target=q_proj,v_proj, lr=0.001, epochs=3, train_n=8000, eval=epoch+best_ckpt | 11.7% | **6.72%** | 42.6% | 0.0분 |
| 3 | exp3_beam5 | lora_r=32, target=q_proj,v_proj, lr=0.001, epochs=3, train_n=8000, eval=epoch+best_ckpt, num_beams=5 | 11.7% | **6.3%** | 46.2% | 0.0분 |
| 4 | exp4_capacity_r64_qkvo | lora_r=64, target=q,k,v,out, lr=0.001, epochs=3, train_n=8000, eval=epoch+best_ckpt, num_beams=5 | 11.7% | **8.09%** | 30.8% | 0.0분 |
| 5 | exp5_lr2e-4_cosine_ep6 | lora_r=32, target=q_proj,v_proj, lr=0.0002, epochs=6, train_n=8000, eval=epoch+best_ckpt+warmup+cosine, num_beams=5 | 11.7% | **6.1%** | 47.8% | 0.0분 |
| 6 | exp9_decoding_params | num_beams=5/10 | 11.7% | **6.08%** | 48.0% | 0.0분 |
| 7 | exp7_data16k | lora_r=32, target=q_proj,v_proj, lr=0.0002, epochs=3, train_n=16000, eval=epoch+best_ckpt+warmup+cosine, num_beams=5 | 11.7% | **6.02%** | 48.6% | 0.0분 |
| 8 | exp8_capacity_r64_at_lr2e-4 | lora_r=64, target=q,k,v,out, lr=0.0002, epochs=3, train_n=16000, eval=epoch+best_ckpt+warmup+cosine, num_beams=5 | 11.7% | **5.43%** | 53.6% | 0.0분 |
| 9 | exp11_capacity_r128 | lora_r=128, target=q,k,v,out, lr=0.0002, epochs=2, train_n=16000, num_beams=5 | 11.7% | **5.74%** | 50.9% | 49분 |
| 10 | exp6_lr1e-4_cosine_ep4 | lora_r=32, target=q_proj,v_proj, lr=0.0001, epochs=4, train_n=8000, eval=epoch+best_ckpt+warmup+cosine, num_beams=5 | 11.7% | **6.47%** | 44.7% | 45분 |
| 11 | exp10_final_500sample_reeval | num_beams=5 | 11.7% | **5.56%** | 52.5% | 20분 |

## 오류 유형 분해 (최고 성능 실험)

| 오류 유형 | 파인튜닝 전 | 파인튜닝 후 |
|---|---|---|
| 구두점 표기 | 1.65%p | 0.33%p |
| 숫자 표기 | 1.72%p | -0.16%p |
| 띄어쓰기 | 0.71%p | 0.41%p |
| **순수 인식 오류** | **7.61%** | **4.85%** |

> CER을 그대로 읽지 않고 요소별로 분해했다. 전체 오류 중 표기 규칙(구두점·숫자) 차이가 차지하는 비중과 실제 음성 인식 실패를 분리해야 개선 방향을 정할 수 있기 때문이다.

## 실험별 메모

**exp1_lora_r32_ep3** (2026-07-23 14:55)  
첫 LoRA 실험. 숫자표기·구두점 크게 개선. 일부 샘플 퇴화로 과적합 의심  
개선 112건 / 퇴화 35건

**exp2_eval_bestckpt** (2026-07-23 15:46)  
eval 도입. valid loss가 epoch1 0.251 최저 후 0.268/0.321로 상승 -> 과적합 확인, best=epoch1 자동 채택  
개선 125건 / 퇴화 29건

**exp3_beam5** (2026-07-23 16:01)  
exp2 모델 그대로, 추론만 beam search(5). 발화당 0.6초로 준실시간 청크 방식에 적용 가능  
개선 130건 / 퇴화 25건

**exp4_capacity_r64_qkvo** (2026-07-23 16:48)  
용량 병목 검증. rank32->64, 모듈 2->4개. 순수 인식오류가 내려가면 용량 문제, 아니면 도메인 noise floor로 결론  
개선 109건 / 퇴화 45건

**exp5_lr2e-4_cosine_ep6** (2026-07-24 10:29)  
exp2/exp4 valid loss 곡선이 공통으로 LR 과다를 시사 -> LR 5배 인하. valid loss 최저 0.2323(exp2 0.251 대비 개선), CER 6.10%로 최고 기록. under-training 가설 확정. 단 epoch2가 최적이라 6epoch은 과다, 3epoch이면 충분  
개선 127건 / 퇴화 20건

**exp9_decoding_params** (2026-07-24 12:17)  
디코딩 파라미터 탐색. 최적 6.08%(length_penalty 1.2)로 기준 beam5 6.10% 대비 0.02%p. 200샘플 기준 측정 노이즈 수준이라 유의한 개선 아님 -> 디코딩 축 소진. 동시에 200샘플 평가의 해상도 한계 확인 -> 최종 순위는 500샘플 재측정 필요  
개선 127건 / 퇴화 20건

**exp7_data16k** (2026-07-24 13:40)  
데이터 2배(8000->16000). valid 최저 0.2323->0.2263, CER 6.10->6.02%. 데이터 축은 아직 소진되지 않음. 단 epoch1에서 최저 도달 후 바로 과적합 -> 데이터를 늘려도 epoch은 짧게 가야 함. 개선폭 0.08%p는 200샘플 기준 노이즈 가능성 있어 500샘플 재측정 필요  
개선 125건 / 퇴화 20건

**exp8_capacity_r64_at_lr2e-4** (2026-07-24 15:46)  
exp4 재검증 결과 초기 결론이 뒤집힘. 동일 용량(r64,q·k·v·o)인데 LR만 1e-3->2e-4로 바꾸자 valid 0.338->0.2232, CER 8.09%->5.43%. exp4의 실패 원인은 용량이 아니라 LR 과다였음이 확정. 용량 축은 유효하며 아직 소진되지 않음. 단 파라미터 4.71%로 exp7의 4배라 epoch1에서 이미 최저 -> 학습은 짧게  
개선 133건 / 퇴화 15건

**exp11_capacity_r128** (2026-07-24)  
용량 축 한 단계 더. 200샘플 5.38%로 exp8(5.43%)보다 나아 보였으나 500샘플 재측정 시 5.74% vs 5.56%로 역전. 용량은 r64에서 포화하며 r128은 파라미터 2배를 쓰고 오히려 과적합. 표본 부족으로 인한 순위 오판 사례.  

**exp6_lr1e-4_cosine_ep4** (2026-07-24)  
LR 최적점 탐색. 2e-4 -> 1e-4로 인하했으나 CER 6.47%로 악화(exp5 6.10%). valid 최저는 0.2325로 exp5(0.2323)와 거의 동일하나 train loss가 0.156에서 멈춰 학습 부족 상태. 2e-4가 최적점이며 LR 축 소진 확인.  

**exp10_final_500sample_reeval** (2026-07-24)  
최종 후보 4개(exp5/7/8/11)를 valid 전체 500샘플로 재측정. 결과: 6.05% / 5.86% / 5.56% / 5.74%. 200샘플 순위(exp11<exp8)가 500샘플에서 역전(exp8<exp11) -> exp8 최종 채택. 표본 부족이 순위를 오판시킬 수 있음을 확인한 검증 단계.  
