---
title: "RiverWatch v2.0 — 보안 강화 + UPSERT + URL 통합"
date: 2026-07-04
draft: false
---

## 작업 요약

RiverWatch 시스템 v2.0 업그레이드 — 10개 항목 정리 + 보안 강화 + 데이터 신뢰성 확보.

## 완료 항목

### 시스템 정리 (10항목)
1. 제목 통일: `RiverWatch — 서울하천 AI 모니터링` (9개 HTML)
2. URL 통합: `dorimchun-ai.pages.dev` (새 Cloudflare Pages 프로젝트)
3. 네비게이션 재구성: RiverWatch 상위, 하천ON/Geosigi 하위 서비스
4. 보안 감사 + RLS 강화
5. 망남클린워치/서남해그랜드 제외
6. 크레딧 통일: 건강한도림천을만드는주민모임 1999-2026, 도깨비3.0, 사회혁신교육원
7. UI/UX 일관성
8. 사용자 매뉴얼 v2.0
9. 시스템개발계획서 v2.0
10. 보안점검보고서 v1.0

### 보안 강화
- **RLS INSERT 차단**: RiverWatch 9개 테이블 anon INSERT → `WITH CHECK (false)`
- **Geosigi INSERT 인증**: `auth.uid() IS NOT NULL` 조건
- **UPSERT 중복방지**: 5개 테이블 UNIQUE 제약조건 + `on_conflict` 파라미터
  - species_observations: 833 → 400건 (중복 제거)
  - cultural_assets: 674 → 337건 (중복 제거)

### 데이터 현황 (중복 제거 후)
| 테이블 | 건수 |
|--------|------|
| river_readings | 147 |
| species_observations | 400 |
| ehi_scores | 123 |
| cultural_assets | 337 |
| weather_forecasts | 2,635 |
| flood_alerts | 86 |
| flood_predictions | 756 |
| collector_health | 20 |
| **총계** | **4,894** |

### GitHub Actions 수정
- `model.onnx.data` 누락 → git 추가 (flood_predict 실패 원인 해결)
- `continue-on-error` 추가 예정 (workflow scope 문제로 보류)

## 미완료
- [ ] GitHub OAuth workflow scope 해결 (salutth vs salutth0529-wq 계정 불일치)
- [ ] HTTP → HTTPS 전환 (서울시/문화재청 API)
- [ ] innerHTML → textContent 전환
- [ ] CSP 헤더 설정

## 서비스 URL
- https://dorimchun-ai.pages.dev (메인)
- https://dorimchun-ai.pages.dev/dashboard (대시보드)
- https://dorimchun-ai.pages.dev/river-on (하천ON)
- https://geosigi.pages.dev (거시기 PWA)
