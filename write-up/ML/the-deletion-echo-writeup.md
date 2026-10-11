# the-deletion-echo — 풀이

**Flag:** `isfcr{763c8479fac095a9e471ad3d}`

`unlock.py`에 아래 정답 리스트를 넣어 직접 실행해 확인했습니다.

## 풀이 과정

### 1. 스냅샷별 모델 복원

스냅샷마다 쿼리가 240개씩 있습니다. 모든 입력 벡터는 contract의 `permutation`, `center`, `scale`로 정규화한 뒤 사용했습니다.

- `predictive_variance = σ²(1 + qᵀH⁻¹q)`는 qqᵀ의 상삼각 78개 항과 상수항에 대해 선형입니다. 최소제곱으로 σ²와 H⁻¹를 구하고, 역행렬을 취해 H를 얻었습니다.
- `mean = qᵀθ`도 최소제곱으로 풀어 θ를 구했습니다.
- 이것으로 b = Hθ = Σxy 를 복원했습니다.

### 2. 버전 순서 복원

두 스냅샷의 H 차이에서 배치의 Σxxᵀ를 뺀 값이 PSD가 되는지 모든 조합에 대해 검사했습니다. PSD라면 남는 것은 삭제분과 ridge 변화뿐입니다.

```
b5c2 ─(ingest-5798d74d)→ e21c ─(ingest-e0449f2a + 삭제)→ 720a ─(ingest-578b7698)→ 83c2
```

- `d170`은 `e21c`와 데이터가 같고 ridge만 다른 stale redeploy입니다.

### 3. 삭제된 레코드 찾기

잔차 R = G_batch − ΔH 를 후보 레코드들의 xxᵀ와 ±I(ridge 변화분)로 NNLS 분해했습니다. 정확히 26개 후보의 가중치가 1.000으로 나왔고, 잔차는 약 0.3이었습니다.

### 4. 라벨 복원

Δb = Σ_batch x·y − Σ_deleted x·y 는 미지수가 36개(배치 10개, 삭제 26개)이고 식은 12개뿐입니다. 하지만 라벨이 {0,1,2,3} 정수라는 제약이 있어서, 격자 CVP(Kannan embedding과 BKZ-20)로 유일해를 구했습니다. 오차는 0.004 미만이었습니다.

- 같은 방법을 나머지 두 ingest 배치에 적용하면 라벨이 정확히 정수로 떨어집니다. 이걸로 복원 정밀도를 검증했습니다.

## 정답 (삭제된 26개 레코드 : 라벨)

| Record | Label |
|---|---|
| record-0730706150 | 0 |
| record-28c0bbf813 | 2 |
| record-2e6662ecc6 | 3 |
| record-3de105d25d | 3 |
| record-3ee76f33c8 | 2 |
| record-46126dd779 | 0 |
| record-473e486afd | 0 |
| record-49aa7c51a4 | 2 |
| record-507645d5d9 | 2 |
| record-58fad90382 | 2 |
| record-5bc7d8554c | 1 |
| record-66f65f5026 | 0 |
| record-71fb3e266c | 2 |
| record-7796490f27 | 1 |
| record-77a4509144 | 1 |
| record-8f4b4073f9 | 1 |
| record-90769aad7a | 1 |
| record-978b124800 | 1 |
| record-9cddf7a4fb | 0 |
| record-9cfdc74744 | 3 |
| record-b9e67b6811 | 2 |
| record-cf44aec78a | 2 |
| record-cf4fee237a | 0 |
| record-d32fd7d5d5 | 1 |
| record-ebfb2ad09e | 0 |
| record-ee9e7ec4d8 | 1 |

### unlock.py 입력

```json
["record-0730706150:0","record-28c0bbf813:2","record-2e6662ecc6:3","record-3de105d25d:3","record-3ee76f33c8:2","record-46126dd779:0","record-473e486afd:0","record-49aa7c51a4:2","record-507645d5d9:2","record-58fad90382:2","record-5bc7d8554c:1","record-66f65f5026:0","record-71fb3e266c:2","record-7796490f27:1","record-77a4509144:1","record-8f4b4073f9:1","record-90769aad7a:1","record-978b124800:1","record-9cddf7a4fb:0","record-9cfdc74744:3","record-b9e67b6811:2","record-cf44aec78a:2","record-cf4fee237a:0","record-d32fd7d5d5:1","record-ebfb2ad09e:0","record-ee9e7ec4d8:1"]
```

```
$ python3 unlock.py '<위 JSON>'
isfcr{763c8479fac095a9e471ad3d}
```
