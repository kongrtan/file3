# Root CA → Star Cert vs Root CA → Intermediate CA → Star Cert 비교 정리

## 1. 구조적 차이

| 항목 | Root → Star | Root → Intermediate → Star |
|------|-------------|----------------------------|
| 인증서 체인 길이 | 짧음 (2단계) | 김 (3단계) |
| 신뢰 구조 | 단일 루트에 직접 의존 | 루트 → 중간 CA가 관리 |
| 발급 유연성 | 낮음 | 높음 |

---

## 2. 보안 측면 차이

### Root → Star
- 루트 CA가 직접 발급하는 구조  
- 루트 CA가 노출되면 전체 보안이 붕괴  
- 실환경에서는 **매우 비권장**

### Root → Intermediate → Star
- 업계 표준, 모든 공인 CA가 사용하는 방식  
- 루트 CA는 오프라인 보관 (air-gapped)  
- 실제 발급 기능은 Intermediate가 수행 → 위험 분산  
- 갱신/폐기/관리 유연성 높음

---

## 3. 운영/확장 측면 차이

| 항목 | Root → Star | Root → Intermediate → Star |
|------|-------------|----------------------------|
| 대규모 운영 | 어려움 | 쉬움 (Intermediate 여러 개 구성 가능) |
| 폐기(Revocation) 영향도 | 매우 큼 (루트 장애 시 전체 무효) | 제한적 (Intermediate만 재발급 가능) |
| 재발급 편의성 | 낮음 | 높음 |

---

## 4. 환경별 인증서 검증

- 두 구조 모두 검증 가능  
- Intermediate 구조가 업계 표준이라 **호환성 최고**

---

## 결론

### ✔ Root → Star
- 단순하지만 보안 리스크 큼  
- 내부 테스트 환경 정도에서만 사용 권장  

### ✔ Root → Intermediate → Star
- 공인 CA와 동일한 실환경 구조  
- 보안, 운영, 확장성 모두 우수  
- 가장 권장되는 PKI 구성

---
