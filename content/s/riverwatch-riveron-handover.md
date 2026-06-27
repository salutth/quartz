---
title: "RiverWatch + 하천ON 핸드오버 (2026-06-28 세션2)"
date: 2026-06-28
tags: [riverwatch, 하천ON, handover, supabase, telegram]
---

## 이번 세션 완료 작업

### 1. Supabase 3차 프로젝트 생성 + 전체 마이그레이션
- 이전 프로젝트 접근 불가 문제 → 새 프로젝트 생성 (서울 리전)
- 프로젝트 ID: `nsuefjoovtbxlohwnigq`
- 8개 테이블 한번에 생성 (RLS + 공개 정책 포함)
- .env + 7개 HTML 파일 자격증명 업데이트 → Cloudflare 배포

### 2. 전체 데이터 수집 완료
| 수집기 | 결과 |
|--------|------|
| 수위 (river_monitor.py) | 21건, 위험 4개 |
| 생물 (species.py) | 457건, 254종 |
| EHI (ehi.py) | 22개 하천 |
| 문화재 (cultural_assets.py) | 337건 |
| 기상 (weather.py) | 1,008건 |
| 침수 경보 (flood_alert.py) | 11건 (긴급 2 + 위험 9) |

### 3. 텔레그램 봇 연동 완료
- 봇: @riverwatch_hacheon_bot
- 토큰 + Chat ID → .env에 저장
- 침수 경보 텔레그램 자동 전송 테스트 성공

### 4. 버그 수정
- EHI 대시보드: `calculated_at` → `created_at` 컬럼명 수정
- 하천ON 지도: 기상 필터 쿼리 오류 수정
- 플랫폼: RiverWatch 카드에 대시보드 링크 추가, 생물종 수 업데이트

### 5. 단계별 구축 가이드 문서 작성
- `/s/riverwatch-setup-guide` — Phase 1~7 전체 과정 + 트러블슈팅

## 배포 URL
- 플랫폼: https://sakyowon-ai.pages.dev/platform
- 대시보드: https://sakyowon-ai.pages.dev/dashboard
- 하천ON: https://sakyowon-ai.pages.dev/river-on

## 미완료 — 다음 세션
1. Phase 4: LSTM 침수 예측 AI (수위 데이터 3개월 축적 후)
