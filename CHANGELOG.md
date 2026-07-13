# Changelog

## [1.0.0] - 2026-07-13

초기 공개.

- R&D 동향 조사 에이전트 팀(2026-04 구축)의 내장 검증자(fact-verifier)를 독립 스킬로 승격·일반화
- 3단 검증 프로토콜: ① 4-Tier 출처 분류 ② 교차 일관성(재인용 체인 감지) ③ 실존 확인(URL/DOI/arXiv/NTIS)
- 판정 4단계(VERIFIED / NEEDS_REVIEW / REJECT / NO_SOURCE) + 사유·조치 병기
- 검증 깊이 3단(quick / standard / deep)
- 출력: 마크다운 검증 보고서(기본) + YAML(에이전트 파이프라인 연동용)
- `references/tier-rules.md`: STI 분야 기본 Tier 목록 + 분야 커스터마이즈 지침
