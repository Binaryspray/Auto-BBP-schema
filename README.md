# bug-bounty-schemas

버그바운티 자동화 파이프라인의 모듈 간 인터페이스 스키마 정의 repo.

## 파이프라인 구조

```
Auto-Recon  →  [RR]  →  Auto-Solve  →  [SR]  →  Auto-PoC  →  [PR]
```

## 스키마 목록

| 파일 | Full Name | 생산 모듈 | 소비 모듈 | 상태 |
|------|-----------|-----------|-----------|------|
| `RR.json` | Recon Result | Auto-Recon | Auto-Solve | 미정 |
| `SR.json` | Solve Result | Auto-Solve | Auto-PoC | 확정 |
| `PR.json` | PoC Result | Auto-PoC | - | 확정 |

## 스키마 설명

### RR (Recon Result)
Auto-Recon의 출력. SubDomain / AP 쌍 리스트.
> 스키마 미정 — Auto-Recon 설계 시 확정 예정

### SR (Solve Result)
Auto-Solve의 출력이자 Auto-PoC의 입력.
취약점 테스트 결과 및 Payload를 담는다.

### PR (PoC Result)
Auto-PoC의 출력. HackerOne 보고서 포맷 기반.
`chaining_analysis.status`는 항상 `suggested`로 고정 (미검증).

## 관련 repo

- [bug-bounty-auto-recon](#)
- [bug-bounty-auto-solve](#)
- [bug-bounty-auto-poc](#)