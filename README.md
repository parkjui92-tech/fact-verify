# fact-verify

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

AI가 수집한 조사 결과·동향 브리핑·보고서 초안의 **출처를 믿기 전에 검증**하는 [Claude Code](https://claude.com/claude-code) 스킬입니다.

일반 "팩트체커"가 아닙니다 — 연구·정책 브리핑의 **출처 규율** 도구입니다. 출처가 실존하는지, 주장을 실제로 뒷받침하는지, 여러 출처처럼 보이는 것이 사실은 한 보도자료의 재인용인지를 체계적으로 가려냅니다.

> **English**: A Claude Code skill that audits the sources behind AI-gathered research findings before you trust them. It classifies every source into a 4-tier reliability scheme (government/academic/press/informal), detects re-quotation chains ("fake cross-validation" where multiple articles all trace back to one press release), verifies that URLs, DOIs, arXiv IDs and Korean NTIS project numbers actually exist, and checks numeric/date consistency — then returns a verdict table (VERIFIED / NEEDS_REVIEW / REJECT / NO_SOURCE).

## 왜 필요한가

AI 리서치의 대표적 실패는 **그럴듯한 출처 조작**입니다:

- 존재하지 않는 arXiv ID·DOI를 단 인용 (할루시네이션)
- 실존하는 URL이지만 그 페이지에 없는 내용의 인용 (출처-주장 불일치)
- 언론 3곳 보도처럼 보이지만 전부 같은 보도자료를 받아쓴 **가짜 교차검증**
- 블로그 글이 정부 통계처럼 인용되는 신뢰도 세탁

이 스킬은 위 네 가지를 각각 잡아내도록 설계된 3단 프로토콜(Tier 분류 → 교차 일관성 → 실존 확인)을 실행합니다.

## 주요 기능

- **4-Tier 출처 분류** — 정부·기관 공식 / 학술·국책연구 / 주요 언론 / 비공식. Tier 4는 단독 근거 불가, 원출처 추적 후 승격
- **재인용 체인 감지** — "○○부에 따르면"류 공통 원천 표지를 추적해 독립 출처 수를 셈
- **실존 확인** — URL fetch(404·내용 불일치), DOI/arXiv resolve, NTIS 과제번호 형식
- **수치·날짜 정합성** — 인용 수치가 원문과 일치하는지, 발표일이 맞는지
- **판정 4단계** — ✅ VERIFIED / ⚠️ NEEDS_REVIEW / ❌ REJECT / 🚫 NO_SOURCE, 모든 판정에 사유·조치 병기
- **검증 깊이 3단** — quick(오프라인) / standard(발행 전) / deep(대외 보고서·투고)

## 설치

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92-tech/fact-verify.git
```

설치 후 Claude Code를 재시작하면 자동으로 인식됩니다.

## 사용법

```
이 브리핑 초안 출처 검증해줘
조사 결과에서 할루시네이션 걸러줘
보고서에 실을 근거들 교차 검증해줘 (deep)
이 자료 믿을 만한지 출처 확인해줘
```

### 출력 예시

| # | 주장(요약) | 출처 | Tier | 판정 | 사유·조치 |
|---|-----------|------|------|------|-----------|
| 1 | 정부 AI R&D 예산 20% 증액 | msit.go.kr 보도자료 | 1 | ✅ | 원문 확인, 수치 일치 |
| 2 | 신규 아키텍처 성능 2배 | arXiv 2404.99999 | 2 | ❌ | 해당 ID 실존하지 않음 — 삭제·재조사 |
| 3 | 업계 투자 급증 | 매경·한경 기사 2건 | 3 | ⚠️ | 둘 다 동일 보도자료 재인용 — 독립 출처 1개 |
| 4 | 해외 사례 성과 | 개인 블로그 | 4 | ❌ | Tier 4 단독 근거 불가 — 원출처(보고서) 확인 필요 |

전체 예시는 [examples/verification-report-example.md](examples/verification-report-example.md) 참고.

## 에이전트 파이프라인의 게이트로

이 스킬은 원래 R&D 동향 조사 에이전트 팀의 내장 검증자로 설계되어, 3명의 조사 에이전트가 수집한 주간 브리핑 재료를 집필 전에 걸러내는 관문으로 실전 사용된 것을 독립 스킬로 승격한 것입니다. `조사 → fact-verify → 집필` 순서로 배치하면 어떤 리서치 파이프라인에도 검증 게이트를 넣을 수 있습니다 (에이전트 정의 예시는 [SKILL.md](SKILL.md) 참고).

## 한계

- URL·DOI 실존 확인에는 웹 접근이 필요합니다 (오프라인에서는 `quick` 깊이만).
- 페이월·로그인 뒤 문서는 확인 불가 시 ⚠️로 정직하게 보류합니다 (오류로 단정하지 않음).
- 출처의 실존·일치를 검증하는 도구이지, 주장의 학술적 타당성을 최종 보증하지 않습니다.

## 커스터마이즈

기본 Tier 목록은 과학기술정책(STI) 분야 기준입니다. `references/tier-rules.md`에 자기 분야의 기관·학술지·전문지를 추가해 쓰세요 (파일 말미에 분야별 보강 지침).

## 연구자용 스킬 시리즈

- **fact-verify** (이 저장소) — 출처 신뢰도 검증 게이트
- **[paper-proofread](https://github.com/parkjui92-tech/paper-proofread)** — 한국어 학술 원고 교정교열 (본문↔참고문헌 대조·서지 실재 검증 포함)

## 라이선스

[Apache License 2.0](LICENSE)
