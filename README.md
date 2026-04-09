# bug-bounty-schemas

버그바운티 자동화 파이프라인의 모듈 간 인터페이스 스키마 정의 repo.

## 파이프라인 구조

```
Auto-Recon  →  [RR]  →  Auto-Solve  →  [SR]  →  Auto-PoC  →  [PR]
```

## 스키마 목록

| 파일 | Full Name | 생산 모듈 | 소비 모듈 | 상태 |
|------|-----------|-----------|-----------|------|
| `RR.json` | Recon Result | Auto-Recon | Auto-Solve | 확정 |
| `SR.json` | Solve Result | Auto-Solve | Auto-PoC | 확정 |
| `PR.json` | PoC Result | Auto-PoC | - | 확정 |

## 데이터 흐름

```
┌────────────┐     RR.json      ┌────────────┐     SR.json      ┌────────────┐
│ Auto-Recon │────────────────>│ Auto-Solve │────────────────>│  Auto-PoC  │
│            │                  │            │                  │            │
│ h1scout    │  attack_points[] │ 취약점 검증  │  vulnerability   │ 보고서 생성  │
│ select     │  + evidence      │ + payload   │  + reproduction  │ + chaining │
│ review ────┤                  │  생성       │    steps         │            │
│  (AP 선택) │                  │            │                  │            │
└────────────┘                  └────────────┘                  └──────┬─────┘
                                                                      │
                                                                 PR.json
                                                                      │
                                                                      ▼
                                                               HackerOne 보고서
```

## 스키마 설명

### RR (Recon Result)

Auto-Recon (`h1scout select`)의 출력. BBP의 공격 표면 분석 결과.

**생성 과정:**
1. BBOT → 서브도메인 수집
2. httpx → 라이브 호스트 필터링
3. gau/waybackurls → URL 수집
4. linkfinder → JS 엔드포인트 추출
5. nuclei → 알려진 취약점 패턴 탐지
6. LLM → Attack Point 식별 및 분류

**주요 필드:**
- `attack_points[].category` — IDOR, auth_bypass, exposure, secret, injection, takeover, oauth_misconfig, jwt_attack, routing_inference 등
- `attack_points[].priority` — 1(최고) ~ 5(최저)
- `attack_points[].evidence.source` — 어떤 도구에서 발견했는지 (gau, linkfinder, nuclei, llm_inference 등)
- `attack_points[].evidence.llm_reasoning` — LLM의 판단 근거 (null이면 도구 직접 발견)

**Auto-Solve로 전달 시:** `h1scout review`로 AP를 선택하면 `selected_attack_points[]`로 변환되어 SR 입력으로 사용됨.

### SR (Solve Result)

Auto-Solve의 입력이자 Auto-PoC의 입력.

**입력 단계 (Auto-Recon → Auto-Solve):**
- `selected_attack_points[]` — `h1scout review`에서 선택된 AP
- `bbp`, `target`, `scope` — 프로그램/타겟 컨텍스트

**출력 단계 (Auto-Solve → Auto-PoC):**
- `vulnerability` — 검증된 취약점 유형, payload, 재현 단계
- `program_info` — 보고서 작성에 필요한 프로그램 정보

### PR (PoC Result)

Auto-PoC의 출력. HackerOne 보고서 포맷 기반.

**주요 필드:**
- `weakness` — CWE ID/이름
- `description` — 요약, 재현 단계, 권고사항, 참고자료
- `chaining_analysis` — 취약점 체이닝 분석 (status는 항상 `suggested`, 미검증)

## 관련 repo

| repo | 역할 |
|------|------|
| [Auto-Recon](https://github.com/user/Auto-Recon) | BBP 선정 + 공격 표면 탐색 → RR.json |
| Auto-Solve | 취약점 검증 + payload 생성 → SR.json |
| Auto-PoC | PoC 재현 + 보고서 생성 → PR.json |
