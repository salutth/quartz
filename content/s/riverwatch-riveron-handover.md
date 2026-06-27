---
title: "RiverWatch + 하천ON 핸드오버 (2026-06-28)"
date: 2026-06-28
tags: [riverwatch, 하천ON, handover, supabase]
---

## 이번 세션 완료 작업

### 1. Supabase 신규 프로젝트 전환
- 이전 프로젝트(gypkipqlkaeocrvzueek) 무료 티어 자동 삭제
- 신규: `tpnikydhphhpwtksxanm.supabase.co`
- 7개 HTML 파일 + .env 자격증명 교체 완료
- 모든 API 응답에 `Array.isArray` 방어처리 추가

### 2. 21개 하천 데이터 수집 완료
- 생물: 439건, 244종 (iNaturalist)
- EHI: 20개 하천 (A등급 7, B등급 10, C등급 3)
- 수위: 21개 관측소 (위험 4, 주의 10, 안전 7)

### 3. 하천ON 문화재 방재 시스템 구축
| Phase | 내용 | 상태 |
|-------|------|------|
| 1-1 | 문화재청 API → 수변 문화재 337건 수집 | ✅ |
| 1-2 | Open-Meteo 기상 예보 연동 (48시간) | ✅ |
| 2 | Leaflet 통합 지도 (`river-on.html`) | ✅ |
| 3 | 침수 경보 시스템 (`flood_alert.py`) | ✅ |

### 4. 경보 시스템 테스트 결과
- 긴급 2건: 신의교(중랑천, 보물·사적 5건), 모래말옆(방학천, 연산군묘)
- 위험 2건: 계성교(우이천 87.4%), 양산교(도림천 88.8%)
- 주의 7건: 정릉천, 우이천, 탄천, 홍제천 등

## 미완료 — 다음 세션에서 처리

1. **flood_alerts 테이블 생성** — Supabase SQL Editor에서 실행 필요 (SQL은 `flood_alert.py` 상단에 있음)
2. **텔레그램 봇 연동** — @BotFather에서 봇 생성 → `.env`에 `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` 추가
3. **Phase 4 LSTM** — 수위 데이터 3개월 축적 후 시계열 예측 모델

## 파일 구조

```
sakyowon-ai/
├── dashboard.html       # RiverWatch 대시보드
├── river-on.html        # 하천ON 방재 지도 (NEW)
├── platform.html        # 플랫폼 랜딩
├── today.html / play.html / mission.html / report.html
├── .env                 # API 키 (push 금지)
├── collectors/
│   ├── species.py       # Agent 1: 생물 수집
│   ├── ehi.py           # Agent 2: EHI 산출
│   ├── invasive_alert.py # Agent 3: 외래종
│   ├── policy_report.py  # Agent 4: 정책보고서
│   ├── cultural_assets.py # 문화재 수집 (NEW)
│   ├── weather.py        # 기상 예보 (NEW)
│   └── flood_alert.py    # 침수 경보 (NEW)
└── river_monitor.py     # 수위 모니터링
```

## 배포 URL
- 대시보드: https://sakyowon-ai.pages.dev/dashboard
- 하천ON: https://sakyowon-ai.pages.dev/river-on
- 플랫폼: https://sakyowon-ai.pages.dev/platform
