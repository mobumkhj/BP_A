# BP_A — MagicSquare 4×4 TDD 프로젝트 KPT 회고

| 항목 | 내용 |
|------|------|
| **프로젝트** | [MagicSquare_1](https://github.com/mobumkhj/MagicSquare_1) / MagicSquare_xx |
| **회고 대상 기간** | RED 설계(Report/13~14) → GREEN G-01~G-07(Report/17) → Golden Master·REFACTOR·QA·Export(Report/19~24) |
| **작성자** | 강희주 |
| **리뷰어** | 권유리, 권태현, 김경민, 김석범, 김용준, 김주현 |
| **작성일** | 2026-05-29 |
| **기준 브랜치** | `refactor/refactor` |
| **기준선 (Step 0)** | pytest **71 passed** · Golden Master **6/6** · 커버리지 Domain **95%** / Boundary **88%** / 전역 **90%** |

---

## 작업 요약 (회고 맥락)

4×4 Magic Square를 **알고리즘 구현**이 아니라 **불변식·입출력 계약 고정 + Dual-Track TDD + ECB(boundary → control → entity)** 훈련으로 진행했다.

| Phase | 핵심 성과 |
|-------|-----------|
| **RED** | Dual-Track 스켈레톤 49건 → G-01~G-07에서 assertion 전환 완료 |
| **GREEN** | Track B(D-LOC~D-SOL) → Track A(U-IN~U-OUT) 순 최소 구현 · 53→71 tests |
| **회귀** | Golden Master GM-TC-01~05 · `golden_master_expected.txt` baseline 고정 |
| **REFACTOR** | RF-01 `ValidationResult` 분리(C1) · R-01~R-08 백로그·README §7.3 SSOT |
| **QA** | NFR gate 충족 · RED 스텁 0건 확인 |

상세 산출: 프로젝트 `Report/17`~`Report/24`, `README.md` §7.3.

---

## Keep — 잘했고 계속할 것

### TDD·품질 문화

- **RED 확인 후 GREEN**: `pytest.fail` 스텁을 assertion으로 바꾸기 전에 실패 원인을 확인하는 흐름을 지켰고, REFACTOR 착수 전 **RED gate(스텁 0건)** 를 명시적으로 검증했다.
- **Dual-Track TDD**: Track A(Boundary 계약·U-IN/U-OUT)와 Track B(Entity 규칙·D-VAL/D-SOL)를 병렬로 설계·구현해, UI 계약과 도메인 규칙을 분리해 학습할 수 있었다.
- **최소 구현(GREEN)**: G-01~G-07에서 “통과에 필요한 최소 코드”만 넣고, 구조 개선은 REFACTOR 단계로 미룬 판단이 회귀 테스트와 맞았다.

### 아키텍처·추적성

- **ECB 의존 방향**(`boundary → control → entity`)을 Cursor rules·Report에 SSOT로 고정하고, 역방향 import·`print()`·bare `except:` 를 피했다.
- **Concept-to-Code Traceability**: `Scenario → AC → RED Test ID → 구현` 체인과 Report/01~24 문서 체인으로 “왜 이 테스트가 있는지”를 추적할 수 있게 했다.
- **도메인 상수 SSOT**: `GRID_SIZE`, `MAGIC_CONSTANT` 등 매직 넘버를 entity 상수로 두어 규칙 변경 추적이 쉬웠다.

### REFACTOR 전 방어선

- **Golden Master 회귀 스위트**(Report/19): REFACTOR 전에 approval 시나리오 6건과 baseline을 고정해, RF-01 이후에도 **GM 6/6·전체 71/71** 불변을 확인할 수 있었다.
- **REFACTOR 계획 선행**(Report/20): code-reviewer 기반 ECB/SRP 분석 → README §7.3 실행 계획 → **기능 변경 없이** RF-01만 착수한 순서가 안전했다.
- **RF-01 (ValidationResult)**: Boundary 검증 결과를 SRP에 맞게 분리하고 `FailureResponse` dead path를 제거했으며, **계약 테스트·GM 모두 GREEN** 을 유지했다.

### 측정·문서화

- **NFR 커버리지 gate** (Step 0 실측): Domain 95%, Boundary 88%, 전역 90% — 프로젝트 목표(80%+/85%+)를 충족했다.
- **8섹션 Export·Session Summary**(Report/23~24): Step 0 재실측·미완료 항목·이슈表를 한곳에 모아, 다음 스프린트 입력으로 쓸 수 있게 정리했다.

---

## Problem — 문제·아쉬운 점

### 구조·테스트 갭

- **`tests/control/` 미착수 (P0-0)**: ECB 권장 트리와 달리 Control 레이어 전용 테스트 디렉터리가 없어, `D-SOL-*` 가 entity 쪽에만 있고 **Control 오케스트레이션**은 직접 검증이 약하다.
- **`solver.py`의 Control·Entity 혼재 (R-06)**: 알고리즘 핵심이 `src/control/solver.py`에 남아 ECB 관점의 **High 이슈(H4)** 가 해소되지 않았다. REFACTOR Wave 2의 가장 큰 기술 부채다.
- **Screen 레이어 ECB 이슈 (B1~B2)**: `screen` → `entity`/`control` 직접 wiring은 REFACTOR P1 대상으로만 계획되어 있고 아직 미해결이다.

### 계약·커버리지

- **이중 SSOT (H1)**: `E001_*` 와 `INVALID_SIZE_*` 가 공존해, R-01 통합 시 AC-FR-01-01·GM 회귀에 주의가 필요하다(DEF-H1).
- **`magic_square_validator.py` 79%**: D-VAL 방어 분기 5줄이 미커버 — NFR은 통과했으나 **strict 100%** 는 아니다.
- **PyQt boot 경로**: `screen/app.py`, `__main__.py` 등은 선택 smoke 없이 0%에 가깝다(기능 리스크는 낮으나 관측성 부족).

### 프로세스·도구

- **REFACTOR 백로그 대부분 “계획만”**: R-01~R-08 중 실제 코드 변경은 **RF-01(C1)뿐** — 계획 대비 실행 속도가 느렸다.
- **GM 실행 경로 혼선 (ISS-24-01)**: 프롬프트/CI 템플릿의 `test_gm_01_*` 경로는 존재하지 않고, SSOT는 `tests/golden_master/` — 문서·자동화 간 **경로 불일치**가 반복될 여지가 있다.
- **로컬 CI 도구**: `gh` CLI 미설치 등 환경 차이로 원격 작업 시 추가 단계가 필요했다(본 회고 작성 시에도 `git push`로 대체).

---

## Try — 다음에 시도할 것

### 우선순위 (Report/20 Phase 0-A)

1. **`tests/control/` P0-0**: Control 유스케이스 characterization RED를 먼저 추가하고, `pytest tests/entity/ tests/control/` 를 REFACTOR SSOT 명령으로 고정한다.
2. **R-06 — `solver.py` → Entity**: 커밋 단위를 작게 쪼개고, 매번 `pytest -q` + Golden Master 6건으로 회귀를 확인한 뒤 README §7.3 체크리스트를 갱신한다.
3. **R-01 — 오류 코드 통합**: `E001` ↔ `INVALID_SIZE` 단일 SSOT화 시 AC-FR-01-01·GM-TC를 반드시 함께 돌린다.

### 품질·자동화

- **validator 5줄 보강**: D-VAL-02~05 방어 분기용 테스트를 추가하거나 R-05와 병행해 Domain 레이어를 **100%에 가깝게** 맞춘다.
- **GM 경로 SSOT 정합**: CI·프롬프트·Report 모두 `tests/golden_master/test_golden_master_magic_square.py` 로 통일해 exit 4 혼선을 없앤다.
- **REFACTOR 브랜치 정책 유지**: `refactor/*` 에서만 구조 변경, G-* GREEN 커밋과 분리 — PR 단위를 “한 REFACTOR ID = 한 PR”로 줄인다.

### 협업·회고

- **KPT 주기화**: GREEN 마일스톤·REFACTOR Wave 종료마다 BP_A README 또는 Issue에 KPT를 남겨, Keep/Problem이 Report에만 묻히지 않게 한다.
- **리뷰어 피드백 루프**: 권유리·권태현·김경민·김석범·김용준·김주현 리뷰 시 ECB 위반·테스트 약화 여부를 체크리스트화해 code-reviewer 산출과 대조한다.

---

## 참고 링크

| 리소스 | URL / 경로 |
|--------|------------|
| 구현 저장소 | https://github.com/mobumkhj/MagicSquare_1 |
| 본 회고 저장소 | https://github.com/mobumkhj/BP_A |
| Session Summary | MagicSquare_xx `Report/24. MagicSquare_Session_Summary_Report.md` |
| REFACTOR 계획 | MagicSquare_xx `README.md` §7.3 · `Report/20` |

---

## 문서 이력

| 버전 | 일자 | 작성자 | 내용 |
|------|------|--------|------|
| 1.0 | 2026-05-29 | 강희주 | MagicSquare TDD 프로젝트 KPT 회고 최초 작성 |
