---
title: "RiverWatch 개발 일지 — 2026-06-18"
tags:
  - riverwatch
  - 개발일지
  - 비공개
---

# RiverWatch 개발 일지

> 이 페이지는 비공개 URL입니다. 링크를 아는 사람만 접근할 수 있습니다.

## 프로젝트 개요

**하천ON + RiverWatch + 거시기** — 도림천 Hub 기반 3개 프로젝트 통합 플랫폼

| 프로젝트 | 핵심 | 상태 |
|---------|------|------|
| 하천ON | 복합재난 경보·취약가구·AR 역사탐방 | 계획서 완료 |
| RiverWatch | 생태건강도(EHI)·정책보고서·10개 하천 | **Phase 2 확장 완료** |
| 거시기(Geosigi) | 외국인 유학생 정착·시민과학 연결 앱 | 계획서·명세서 완료 |

---

## Phase 1 구현 (2026-06-12)

### 데이터 파이프라인
- `river_monitor.py` — 서울시 열린데이터 API → Supabase 자동 저장 (21개 관측소)
- `collectors/species.py` — iNaturalist API → 도림천 생물 관찰 수집 (14건)
- `collectors/ehi.py` — EHI 생태건강지수 A~E 산출
- `collectors/invasive_alert.py` — 15종 생태교란종 경보

### 웹 대시보드
- `dashboard.html` — Leaflet 다크 지도 + 수위 테이블 + 생물 갤러리

---

## Phase 2 확장 (2026-06-18)

### 10개 하천 Hub & Spoke 확장
`species.py`를 도림천 전용에서 서울 10개 하천으로 확장:

| 하천 | 위도 | 경도 | 관찰 건수 |
|------|------|------|----------|
| 도림천 | 37.4838 | 126.9295 | 14건 |
| 안양천 | 37.4750 | 126.8870 | 1건 |
| 중랑천 | 37.5950 | 127.0500 | 14건 |
| 탄천 | 37.5050 | 127.0780 | 50건 |
| 불광천 | 37.5900 | 126.9200 | 13건 |
| 홍제천 | 37.5750 | 126.9450 | 18건 |
| 방학천 | 37.6550 | 127.0280 | 2건 |
| 우이천 | 37.6500 | 127.0130 | 11건 |
| 정릉천 | 37.6050 | 127.0050 | 6건 |
| 청계천 | 37.5700 | 127.0100 | 50건 |

**총 179건 관찰, 119종 확인, 외래종 0건**

### EHI 생태건강도 결과

| 등급 | 하천 | 점수 |
|------|------|------|
| 🟢 A | 청계천, 탄천 | 85.0 |
| 🟢 B | 홍제천(74.5), 도림천(70.0), 불광천(66.0), 중랑천(66.0), 우이천(60.0), 정릉천(60.0) | |
| 🟡 C | 방학천(49.0), 안양천(44.5) | |

EHI = 생물다양성 30% + 수위안정성 30% + 외래종부재 20% + 관찰빈도 20%

### Agent 4: 정책보고서 자동 생성
- `collectors/policy_report.py` — 데이터 집계 → 마크다운 보고서
- `reports/report_2026-06.md` 첫 보고서 생성
- 내용: 요약 + 하천별 EHI + 수위 위험 + 외래종 + 생물다양성 + 정책 제언

### 대시보드 EHI 패널
- `dashboard.html`에 EHI 생태건강도 카드 추가
- 하천별 등급(A~E) + 점수 바 시각화
- Beta 시범운영 배지 추가

### 산출물
- **QR코드**: `qr-dashboard.png` — 대시보드 접속용
- **발표자료**: `RiverWatch_Beta_발표자료.pptx` (8슬라이드, 다크테마)

---

## 인프라 현황

```
GitHub:     https://github.com/salutth/sakyowon-ai
사이트:      https://sakyowon-ai.pages.dev
대시보드:    https://sakyowon-ai.pages.dev/dashboard.html
위키:        https://quartz-33c.pages.dev
Supabase:   https://gypkipqlkaeocrvzueek.supabase.co
```

### Supabase 테이블

| 테이블 | 상태 | 데이터 |
|--------|------|--------|
| `river_readings` | ✅ | 21건/회 |
| `species_observations` | ✅ | 179건+ (10개 하천) |
| `invasive_alerts` | ✅ | 0건 (외래종 미감지) |
| `ehi_scores` | ✅ | 21건 (10개 하천) |
| `collector_health` | ⏳ SQL 준비됨 | - |

### 프로젝트 구조

```
sakyowon-ai/
├── index.html              ← 메인 사이트
├── dashboard.html          ← RiverWatch 대시보드 (Beta)
├── river_monitor.py        ← 수위 수집 + Supabase
├── collectors/
│   ├── species.py          ← Agent 1: 종 식별 (10개 하천)
│   ├── ehi.py              ← Agent 2: EHI 산출
│   ├── invasive_alert.py   ← Agent 3: 외래종 경보
│   └── policy_report.py    ← Agent 4: 정책보고서
├── reports/
│   └── report_2026-06.md   ← 첫 월간 보고서
├── run_collectors.bat      ← Agent 1~4 자동 실행
├── qr-dashboard.png        ← QR코드
├── RiverWatch_Beta_발표자료.pptx ← 발표 자료
├── supabase_setup.sql      ← DB 스키마
├── .env                    ← API 키 (git 제외)
└── .gitignore
```

---

## 다음 세션 TODO

### 즉시 할 일
- [ ] Supabase `collector_health` 테이블 SQL 실행
- [ ] 환경부 TMS API 키 신청 (water.nier.go.kr)
- [ ] Supabase service_role key 발급 → .env 추가 → INSERT 정책 전환

### Phase 3 구현 목표
- [ ] TMS 수질 데이터 연동 (BOD, COD, DO, 총인, 총질소)
- [ ] Agent 4 → LLM (Claude API) 분석 보고서로 업그레이드
- [ ] 문화재 방재 Agent 5 — 국가유산청 DB + LSTM 침수예측
- [ ] 방호가이드 AI Agent 6

### 거시기(Geosigi) 시작 조건
- [ ] 개인정보보호위원회 사전 자문 신청
- [ ] React Native 프로젝트 초기화
- [ ] 정착안내 챗봇 (Agent 7) — LLM + RAG 구현

---

## 통합 아키텍처 (11개 AI Agent)

| # | Agent | 프로젝트 | 상태 |
|---|-------|---------|------|
| 1 | 종 식별 (iNaturalist CV) | RiverWatch | ✅ 동작 |
| 2 | EHI 생태건강지수 | RiverWatch | ✅ 동작 |
| 3 | 외래종 경보 | RiverWatch | ✅ 동작 |
| 4 | 정책보고서 (템플릿) | RiverWatch | ✅ 동작 |
| 5 | 문화재 침수 선제경보 | 하천ON | ⏳ 계획 |
| 6 | 방호가이드 AI | 하천ON | ⏳ 계획 |
| 7 | 정착안내 챗봇 | 거시기 | ⏳ 계획 |
| 8 | 신뢰검증 | 거시기 | ⏳ 계획 |
| 9 | 단체 매칭 | 거시기 | ⏳ 계획 |
| 10 | 실시간 번역 | 거시기 | ⏳ 계획 |
| 11 | 스토리 AI | 거시기 | ⏳ 계획 |
