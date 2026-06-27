---
title: "RiverWatch + 하천ON 구축 가이드 (처음부터 끝까지)"
date: 2026-06-28
tags: [riverwatch, 하천ON, guide, supabase, telegram]
---

## 개요

서울 21개 하천의 수위/생물/생태건강도를 실시간 모니터링하고, 수변 문화재 337건의 침수 위험을 자동 감지하여 텔레그램으로 경보를 보내는 시민과학 AI 플랫폼.

**완성된 서비스 URL:**
- 플랫폼 (소개): https://sakyowon-ai.pages.dev/platform
- 대시보드 (실시간 데이터): https://sakyowon-ai.pages.dev/dashboard
- 하천ON (문화재 방재 지도): https://sakyowon-ai.pages.dev/river-on

---

## Phase 1: 인프라 준비

### 1-1. GitHub 저장소 생성

```
GitHub에서 새 저장소 생성: sakyowon-ai
git clone https://github.com/salutth/sakyowon-ai.git
```

### 1-2. Cloudflare Pages 연결

1. Cloudflare 대시보드 → Pages → "Create a project"
2. GitHub 저장소 연결 (sakyowon-ai)
3. Build settings: 빌드 명령 없음 (정적 HTML)
4. 배포 완료 → `sakyowon-ai.pages.dev`에서 접속 가능
5. **주의:** Cloudflare는 URL에서 `.html` 확장자를 자동 제거 (`/dashboard.html` → `/dashboard`)

### 1-3. Supabase 프로젝트 생성

1. https://supabase.com 가입/로그인
2. "New Project" → 프로젝트명: `riverwatch` → 리전: Northeast Asia (Seoul)
3. 생성 후 Settings → API에서 **Project URL**과 **anon public key** 복사

### 1-4. 환경변수 파일 (.env) 생성

프로젝트 루트에 `.env` 파일 생성 (절대 git에 push하지 않는다):

```
SEOUL_API_KEY=서울열린데이터광장에서_발급받은_키
SUPABASE_URL=https://프로젝트ID.supabase.co
SUPABASE_KEY=anon_public_키
TELEGRAM_BOT_TOKEN=텔레그램봇토큰 (Phase 5에서 생성)
TELEGRAM_CHAT_ID=텔레그램채팅ID (Phase 5에서 생성)
```

`.gitignore`에 `.env` 추가 필수.

### 1-5. Python 환경 확인

- Python 3.12 설치 (python.org에서 다운로드)
- Windows에서는 `python` 명령이 Microsoft Store로 연결될 수 있음
- 전체 경로 사용: `C:\Users\사용자\AppData\Local\Programs\Python\Python312\python.exe`

---

## Phase 2: 데이터베이스 테이블 생성

Supabase SQL Editor에서 아래 SQL 전체를 한번에 실행한다.

```sql
-- 1. 수위 관측 데이터
create table river_readings (
  id bigint generated always as identity primary key,
  station text,
  river text,
  gu text,
  water_level real,
  embankment_height real,
  level_ratio real,
  status text,
  measured_at timestamptz,
  created_at timestamptz default now()
);

-- 2. 생물 관찰 데이터
create table species_observations (
  id bigint generated always as identity primary key,
  inaturalist_id bigint,
  taxon_name text,
  common_name text,
  taxon_id bigint,
  river text,
  species text,
  korean_name text,
  taxon text,
  observed_at timestamptz,
  latitude real,
  longitude real,
  photo_url text,
  observer text,
  source text,
  is_invasive boolean,
  created_at timestamptz default now()
);

-- 3. 생태건강지수 (EHI)
create table ehi_scores (
  id bigint generated always as identity primary key,
  river text,
  biodiversity_score real,
  water_stability_score real,
  non_invasive_score real,
  observation_freq_score real,
  ehi_score real,
  ehi_grade text,
  grade text,
  total_species int,
  native_ratio real,
  taxon_diversity real,
  indicator_score real,
  species_count int,
  reading_count int,
  created_at timestamptz default now()
);

-- 4. 외래종 경보
create table invasive_alerts (
  id bigint generated always as identity primary key,
  river text,
  species text,
  count int,
  risk_level text,
  created_at timestamptz default now()
);

-- 5. 수집기 상태
create table collector_health (
  id bigint generated always as identity primary key,
  collector text,
  status text,
  records_count int,
  error_message text,
  created_at timestamptz default now()
);

-- 6. 수변 문화재
create table cultural_assets (
  id bigint generated always as identity primary key,
  name text,
  name_hanja text,
  grade text,
  grade_code text,
  category text,
  address text,
  district text,
  latitude real,
  longitude real,
  nearest_river text,
  distance_km real,
  risk_priority int,
  asset_code text,
  collected_at timestamptz,
  created_at timestamptz default now()
);

-- 7. 기상 예보
create table weather_forecasts (
  id bigint generated always as identity primary key,
  zone_code text,
  zone_name text,
  river text,
  forecast_date text,
  forecast_hour text,
  temperature real,
  rain_probability int,
  rain_amount text,
  sky_status text,
  humidity real,
  wind_speed real,
  collected_at timestamptz,
  created_at timestamptz default now()
);

-- 8. 침수 경보 이력
create table flood_alerts (
  id bigint generated always as identity primary key,
  station text,
  river text,
  alert_level text,
  water_ratio real,
  rain_probability int,
  nearby_assets int,
  high_grade_assets int,
  reasons text,
  notified boolean default false,
  created_at timestamptz default now()
);

-- RLS 활성화 + 공개 읽기/쓰기 정책 (8개 테이블 일괄)
do $$
declare t text;
begin
  for t in select unnest(array[
    'river_readings','species_observations','ehi_scores',
    'invasive_alerts','collector_health','cultural_assets',
    'weather_forecasts','flood_alerts'
  ]) loop
    execute format('alter table %I enable row level security', t);
    execute format('create policy %I on %I for select using (true)', t || '_read', t);
    execute format('create policy %I on %I for insert with check (true)', t || '_insert', t);
  end loop;
end $$;
```

---

## Phase 3: 데이터 수집기 (Python)

### 3-1. 수위 모니터링 (`river_monitor.py`)

- **데이터 소스:** 서울열린데이터광장 하천 수위 API
- **URL:** `http://openAPI.seoul.go.kr:8088/` (HTTP만 동작, HTTPS는 SSL 오류)
- **수집 대상:** 21개 관측소 (도림천, 안양천, 중랑천, 탄천 등)
- **수집 내용:** 수위, 제방높이, 수위비율, 상태(위험/주의/안전)
- **실행:** `python river_monitor.py`

### 3-2. 생물 관찰 (`collectors/species.py`)

- **데이터 소스:** iNaturalist API (무료, API 키 불필요)
- **수집 방법:** 각 하천 좌표 반경 2km 내 관찰 데이터
- **수집 내용:** 종명, 관찰일, 위치, 사진 URL, 관찰자
- **결과:** 457건, 254종

### 3-3. 생태건강지수 (`collectors/ehi.py`)

- **산출 기준:**
  - 생물다양성 30% + 수위안정성 30% + 외래종부재 20% + 관찰빈도 20%
- **등급:** A(매우좋음) ~ E(매우나쁨)
- **전제 조건:** species.py와 river_monitor.py 먼저 실행 필요

### 3-4. 문화재 수집 (`collectors/cultural_assets.py`)

- **데이터 소스:** 문화재청 공공 API (XML 형식)
- **수집 방법:** 서울 전체 문화재 → 21개 하천 반경 1.5km 필터링
- **분류:** 국보, 보물, 사적, 천연기념물 등 등급별 위험 우선순위 부여
- **결과:** 서울 1,184건 중 수변 337건

### 3-5. 기상 예보 (`collectors/weather.py`)

- **데이터 소스:** Open-Meteo API (무료, API 키 불필요)
- **수집 내용:** 48시간 예보 (기온, 강수확률, 강수량, 풍속, 습도)
- **결과:** 1,008건 (21개 하천 x 48시간)

### 3-6. 침수 경보 (`collectors/flood_alert.py`)

- **판정 기준:**
  - 긴급: 수위비율 80% 이상 AND 국보/보물/사적 1km 이내
  - 위험: 수위비율 80% 이상 OR (수위 60%+ 이면서 강수확률 70%+)
  - 주의: 수위비율 60% 이상 OR 강수확률 80% 이상
- **텔레그램 알림:** 경보 발생 시 자동 전송
- **전제 조건:** 수위 + 기상 + 문화재 데이터가 Supabase에 있어야 함

### 수집기 실행 순서

```bash
python river_monitor.py        # 1. 수위 (21건)
python collectors/species.py   # 2. 생물 (457건)
python collectors/ehi.py       # 3. EHI (species + readings 필요)
python collectors/cultural_assets.py  # 4. 문화재 (337건)
python collectors/weather.py   # 5. 기상 (1,008건)
python collectors/flood_alert.py     # 6. 침수 경보 (수위+기상+문화재 필요)
```

`run_collectors.bat`로 전체를 순차 실행할 수 있다. Windows 작업 스케줄러로 매시간 자동 실행 설정 가능.

---

## Phase 4: 웹 프론트엔드 (HTML)

### 4-1. 플랫폼 랜딩 (`platform.html`)

- 프로젝트 소개 페이지
- RiverWatch → `/dashboard` 링크
- 하천ON → `/river-on` 링크

### 4-2. 대시보드 (`dashboard.html`)

- **수위 지도:** Leaflet.js로 21개 관측소 위치 + 상태 표시
- **생물 갤러리:** 하천별 관찰 종 태그 표시
- **EHI 카드:** 하천별 생태건강도 등급 (A~E) + 점수 바 차트
- **외래종 경보:** 생태교란종 감지 시 경고
- **Supabase 연동:** REST API로 실시간 데이터 조회

### 4-3. 하천ON 방재 지도 (`river-on.html`)

- **Leaflet.js 다크 테마 지도** (CARTO 타일)
- **문화재 마커:** 등급별 색상 (국보=빨강, 보물=주황, 사적=파랑)
- **관측소 마커:** 수위 상태별 색상 (위험/주의/안전)
- **기상 데이터:** 관측소 팝업에 강수확률 표시
- **필터 버튼:** 전체, 위험구역, 국보, 500m 이내

### 4-4. 기타 페이지

- `today.html` — 오늘의 하천 현황
- `play.html` — 하천 탐험 기능
- `mission.html` — 미션 시스템
- `report.html` — 리포트

### HTML 파일의 Supabase 연동 패턴

모든 HTML 파일에서 동일한 패턴을 사용한다:

```javascript
const SUPABASE_URL = 'https://프로젝트ID.supabase.co';
const SUPABASE_KEY = 'anon_public_키';

async function query(table, params) {
  const res = await fetch(`${SUPABASE_URL}/rest/v1/${table}?${params}`, {
    headers: { 'apikey': SUPABASE_KEY }
  });
  const data = await res.json();
  return Array.isArray(data) ? data : [];  // 방어 처리 필수
}
```

**`Array.isArray` 방어 처리:** Supabase가 에러를 반환하면 객체가 오므로, 반드시 배열인지 확인 후 사용.

---

## Phase 5: 텔레그램 봇 연동

### 5-1. 봇 생성

1. 텔레그램 앱에서 **@BotFather** 검색 → 대화 시작
2. `/newbot` 입력
3. 봇 이름 입력: `RiverWatch 하천ON 경보`
4. 봇 username 입력: `riverwatch_hacheon_bot` (끝에 `_bot` 또는 `Bot` 필수)
5. BotFather가 **HTTP API 토큰**을 발급 → 복사

### 5-2. Chat ID 확인

1. 텔레그램에서 생성한 봇 검색 → **시작(Start)** 클릭
2. 아무 메시지 전송 (예: "hello")
3. 브라우저에서 아래 URL 접속:
   ```
   https://api.telegram.org/bot토큰/getUpdates
   ```
4. JSON 응답에서 `"chat":{"id":숫자}` 부분이 Chat ID

### 5-3. .env에 추가

```
TELEGRAM_BOT_TOKEN=봇토큰
TELEGRAM_CHAT_ID=Chat_ID_숫자
```

### 5-4. 테스트

```bash
python collectors/flood_alert.py
```

경보가 있으면 텔레그램으로 자동 전송된다. 경보 내용:
- 긴급/위험/주의 단계별 구분
- 관측소명, 하천명, 수위비율, 강수확률
- 위험 문화재 목록
- 대시보드/지도 링크

---

## Phase 6: 배포

### 6-1. Git Push

```bash
git add dashboard.html platform.html river-on.html (등 변경 파일)
git commit -m "설명"
git push origin master
```

**주의:** `.env` 파일은 절대 push하지 않는다.

### 6-2. Cloudflare 자동 배포

- `git push` 후 1~2분 내 자동 빌드 및 배포
- URL: `https://sakyowon-ai.pages.dev/페이지명` (.html 확장자 없이)

---

## Phase 7: 자동화 (Windows 작업 스케줄러)

1. `run_collectors.bat` 파일에 전체 수집기 실행 명령 작성
2. Windows 작업 스케줄러 → 새 작업 만들기
3. 트리거: 매 1시간마다
4. 동작: `run_collectors.bat` 실행
5. 결과: 매시간 수위/기상 데이터 자동 갱신 + 침수 경보 자동 발송

---

## 트러블슈팅

| 문제 | 원인 | 해결 |
|------|------|------|
| `species.forEach is not a function` | Supabase 에러 객체 반환 | `Array.isArray(data) ? data : []` 방어 추가 |
| 서울 API SSL 오류 | HTTPS 미지원 | URL을 `http://` 로 변경 |
| 기상청 RSS 실패 | API 서비스 중단 | Open-Meteo 무료 API로 대체 |
| Supabase 저장 400 에러 | 테이블 컬럼 불일치 | ALTER TABLE로 누락 컬럼 추가 |
| EHI 데이터 없음 | 쿼리에 없는 컬럼 요청 | `calculated_at` → `created_at` 수정 |
| Supabase 프로젝트 접근 불가 | 다른 계정/조직으로 생성됨 | 접근 가능한 조직에 새 프로젝트 생성 |
| 브라우저 확장 프로그램 오류 | 번역 도구 충돌 | 시크릿 모드 사용 또는 확장 비활성화 |

---

## 현재 상태 (2026-06-28)

| 항목 | 상태 | 비고 |
|------|------|------|
| Supabase 프로젝트 | ✅ 3차 (서울 리전) | nsuefjoovtbxlohwnigq |
| 테이블 8개 | ✅ 모두 생성 | RLS + 공개 정책 적용 |
| 수위 모니터링 | ✅ 21개 관측소 | 위험 4개, 주의 10개 |
| 생물 관찰 | ✅ 457건 / 254종 | iNaturalist |
| EHI 생태건강도 | ✅ 22개 하천 | A~E 등급 |
| 문화재 | ✅ 337건 | 국보 28, 보물 115, 사적 30 |
| 기상 예보 | ✅ 1,008건 | 48시간 예보 |
| 침수 경보 | ✅ 11건 | 긴급 2 + 위험 9 |
| 텔레그램 봇 | ✅ 연동 완료 | @riverwatch_hacheon_bot |
| Cloudflare 배포 | ✅ 자동 배포 | platform, dashboard, river-on |

**다음 단계:**
- Phase 4 (LSTM): 수위 데이터 3개월 축적 후 시계열 예측 모델 개발
