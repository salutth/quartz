---
title: "RiverWatch 개발 일지 — 2026-06-12"
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
| RiverWatch | 생태건강도(EHI)·정책보고서·10개 하천 | **Phase 1 구현 완료** |
| 거시기(Geosigi) | 외국인 유학생 정착·시민과학 연결 앱 | 계획서·명세서 완료 |

---

## 오늘 구현한 것 (2026-06-12)

### 1. 데이터 파이프라인

- `river_monitor.py` — 서울시 열린데이터 API → Supabase 자동 저장
  - 21개 관측소 실시간 수위 수집
  - 위험(80%+) / 주의(50%+) / 안전 3단계 판정
  - 오늘 결과: **4곳 위험** (양산교 88.8%, 계성교 87.3% 등)

- `collectors/species.py` — iNaturalist API → 도림천 생물 관찰 수집
  - 도림천 반경 2km, 최근 30일 관찰 데이터
  - 오늘 결과: **14건** (왕오색나비, 왜가리 등)
  - 15종 생태교란종 자동 판별

### 2. AI Agent 3개 동작

| Agent | 파일 | 기능 |
|-------|------|------|
| Agent 1: 종 식별 | `collectors/species.py` | iNaturalist CV API 연동 |
| Agent 2: EHI 산출 | `collectors/ehi.py` | 생물다양성+수위안정+외래종+관찰빈도 → A~E |
| Agent 3: 외래종 경보 | `collectors/invasive_alert.py` | 15종 DB 매칭 + 3단계 경보 |

**EHI 산출 결과:**
- 도림천: **B등급 (66.0점)** — 생물 데이터 있어 가장 높음
- 나머지 하천: D~E등급 (생물 데이터 축적 필요)

### 3. 웹 대시보드

`dashboard.html` — Supabase 실시간 연동 대시보드
- Leaflet 다크 지도 + 관측소 마커 (빨강/주황/초록)
- 수위 현황 테이블 (위험도순 정렬)
- 도림천 수위 추이 차트
- 생물 관찰 사진 갤러리 (iNaturalist 실제 사진)

### 4. 자동화

- Windows 작업 스케줄러 `RiverWatch_Collector` — 매시간 4개 수집기 자동 실행
- `run_collectors.bat` — 수위 + 생물 + EHI + 외래종 순차 실행

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
| `species_observations` | ✅ | 14건 |
| `invasive_alerts` | ✅ | 0건 (외래종 미감지) |
| `ehi_scores` | ⏳ SQL 실행 필요 | - |

### 프로젝트 구조

```
sakyowon-ai/
├── index.html              ← 메인 사이트
├── dashboard.html          ← RiverWatch 대시보드
├── river_monitor.py        ← 수위 수집 + Supabase
├── collectors/
│   ├── species.py          ← Agent 1: 종 식별
│   ├── ehi.py              ← Agent 2: EHI 산출
│   └── invasive_alert.py   ← Agent 3: 외래종 경보
├── run_collectors.bat      ← 자동 실행 배치
├── supabase_setup.sql      ← DB 스키마
├── .env                    ← API 키 (git 제외)
└── .gitignore
```

---

## 다음 세션 TODO

### 즉시 할 일
- [ ] Supabase에서 `ehi_scores` 테이블 SQL 실행
- [ ] 환경부 TMS API 키 신청 (water.nier.go.kr)
- [ ] 대시보드에 EHI 패널 추가

### Phase 2 구현 목표
- [ ] Agent 4: 정책보고서 — Claude API로 월간 생태 리포트 자동 생성
- [ ] TMS 수질 데이터 연동 (BOD, COD, DO, 총인, 총질소)
- [ ] 10개 하천 Hub & Spoke 확장 (species.py에 좌표 추가)
- [ ] 문화재 방재 Agent 5 — 국가유산청 DB + LSTM 침수예측

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
| 4 | 정책보고서 (LLM) | RiverWatch | ⏳ 다음 |
| 5 | 문화재 침수 선제경보 | 하천ON | ⏳ 계획 |
| 6 | 방호가이드 AI | 하천ON | ⏳ 계획 |
| 7 | 정착안내 챗봇 | 거시기 | ⏳ 계획 |
| 8 | 신뢰검증 | 거시기 | ⏳ 계획 |
| 9 | 단체 매칭 | 거시기 | ⏳ 계획 |
| 10 | 실시간 번역 | 거시기 | ⏳ 계획 |
| 11 | 스토리 AI | 거시기 | ⏳ 계획 |
