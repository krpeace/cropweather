# 🌤️ CropWeather — 농업 기상 분석 도구 (일사·일조, 온도·적산온도)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![R&D](https://img.shields.io/badge/국가R%26D-RS--2025--02305238-orange)](https://www.nrf.re.kr/)

> 기상청 ASOS 66개 지점의 일사·일조·기온·적산온도를 자동 수집·분석하여 엑셀 보고서로 출력하는 도구

---

## 📌 목차

1. [프로젝트 소개](#-프로젝트-소개)
2. [주요 기능](#-주요-기능)
3. [설치](#-설치)
4. [인증키 설정](#-인증키-설정)
5. [사용법](#-사용법)
6. [엑셀 출력 구성](#-엑셀-출력-구성)
7. [지원 작물 목록](#-지원-작물-목록)
8. [디렉터리 구조](#-디렉터리-구조)
9. [문서](#-문서)
10. [관련 저장소](#-관련-저장소)
11. [연구 과제 및 기여](#-연구-과제-및-기여)

---

## 🌱 프로젝트 소개

국가 R&D 과제(RS-2025-02305238, 주관: 주식회사 파모스)로 개발된 **농업 기상 분석 도구**입니다. 기상청 ASOS 공공데이터에서 일사·일조·기온을 자동 수집하고, 권역별·지점별·기간별 비교 분석과 적산온도(GDD) 산출을 수행합니다.

기존에 구글 스프레드시트([기상 데이터 수집, 정리](https://docs.google.com/spreadsheets/d/19eMRA7pvuzxds8cV2gu1YZsHChYwRyzaS-i5KA4-TbY/))로 운영하던 서비스를 Python + 엑셀 출력 방식으로 재개발한 것입니다.

**두 개의 분석 모듈:**

| 모듈 | 입력 | 출력 |
| :--- | :--- | :--- |
| **solar_multi.py** | ASOS 일사·일조 | 7시트 엑셀 (일사량·일조시간·일조율 분석) |
| **temp_multi.py** | ASOS 기온 | 9시트 엑셀 (기온·적산온도·GDD 도달일자 분석) |

두 모듈은 동일한 ASOS API 수집 엔진(`solar_core.py`)과 데이터 캐시(`data/raw/`)를 공유합니다.

---

## ✨ 주요 기능

| 기능 | 설명 |
| :--- | :--- |
| **일사·일조 분석** | 합계 일사량, 일조시간, 가조시간, 일조율 — 권역별·지점별 동기간 비교 |
| **기온 분석** | 평균·최고·최저기온 — 올해/작년/평년 비교, 월별·연도별 추이 |
| **적산온도 (GDD)** | 3가지 계산 방법 지원: (Tmax+Tmin)/2 (기본), avgTa, AquaCrop |
| **41개 작물 기준온도** | Paredes et al. (2025) 기반 Tbase/Tupper 라이브러리 (`crops_gdd.csv`) |
| **적산온도 도달일자** | 500℃~4000℃ 임계값별 도달 날짜, 평년 도달일 자동 산출 |
| **동기간 비교** | 올해 데이터 범위에 맞춰 작년·평년 동일 기간만 자동 추출 |
| **VLOOKUP 동적 시트** | 엑셀에서 "올해" 셀 값만 바꾸면 과거·미래 모두 자동 갱신 |
| **데이터 캐싱** | 확정 연도는 `data/raw/`에 CSV 캐시, 올해분만 API 재조회 |

---

## 🔧 설치

```bash
git clone https://github.com/krpeace/cropweather.git
cd cropweather

# 가상환경 생성 및 활성화
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS/Linux

pip install -r requirements.txt
```

---

## 🔑 인증키 설정

| 키 | 포털 | 용도 |
| :--- | :--- | :--- |
| `DATA_GO_KR` | [공공데이터포털](https://data.go.kr) | ASOS 일자료 조회 |

```bash
cp apikey.txt.example apikey.txt   # 복사 후 실제 키 입력
```

> ⚠️ `apikey.txt`는 `.gitignore`로 관리됩니다. 절대 GitHub에 올리지 마세요.

---

## 🚀 사용법

### solar_multi.py — 일사·일조 분석

```bash
python solar_multi.py                           # 전국 66개 지점, 기본 설정
python solar_multi.py --region 강원영서 강원영동    # 특정 권역만
```

### temp_multi.py — 온도·적산온도 분석

```bash
python temp_multi.py                            # 전국 66개 지점, Tbase=10℃
python temp_multi.py --crop grape               # 포도 기준온도(10℃) 자동 적용
python temp_multi.py --crop apple               # 사과(4℃) + 사과 milestones
python temp_multi.py --tbase 5                  # 기준온도 직접 지정
python temp_multi.py --method avg               # avgTa 방법 사용
python temp_multi.py --region 서울경기 --year 2024   # 권역·연도 지정
```

| 파라미터 | 설명 | 기본값 |
| :--- | :--- | :--- |
| `--crop` | 작물 ID (`crops_gdd.csv` 참조) | 없음 |
| `--tbase` | 기준온도 직접 지정 (℃, `--crop`보다 우선) | 10.0 |
| `--tupper` | 상한온도 직접 지정 (℃) | 35.0 |
| `--method` | GDD 계산 방법: `maxmin`/`avg`/`aquacrop` | `maxmin` |
| `--region` | 분석 대상 권역 (복수 가능) | 전체 |
| `--station` | 분석 대상 지점번호 (복수 가능) | 전체 |
| `--year` | 기준 연도 ("올해") | 올해 |
| `--normal-years` | 평년 산정 기간 | 10 |
| `--no-cache` | CSV 캐시 무시, API 강제 재조회 | 사용 |

---

## 📊 엑셀 출력 구성

### solar_multi.py (7시트)

| 시트 | 내용 |
| :--- | :--- |
| **설정** | 올해·평년·분석기간 (노란 셀 편집 가능) |
| **권역별** | 권역 선택 콤보 + 올해/작년/평년/차이 |
| **지점별** | 지점 선택 콤보 + 동일 비교 |
| **기간별(월별)** | 권역=열, 월별 블록, 동기간 비교 |
| **년도별** | 권역별 × 연도별 슬라이딩 윈도우 |
| **원데이터** | 지점별 일별 전체 관측값 |

### temp_multi.py (9시트)

| 시트 | 내용 |
| :--- | :--- |
| **설정** | 올해·평년·기간 + **Tbase·Tupper·계산방법·작물** |
| **권역별** | 권역 콤보 + 6지표 × 올해/작년/평년/차이 |
| **지점별** | 지점 콤보 + 동일 6지표 |
| **기간별(월별)** | 권역=열, 월별 블록 (solar 서식 통일) |
| **년도별** | 슬라이딩 윈도우 (올해−N년 ~ 올해+5년) |
| **적산온도 도달(권역별)** | 500~4000℃ 도달일자 + 평년 + 일수 차이 |
| **적산온도 도달(지점별)** | 66개 지점별 도달일자 (VLOOKUP 동적) |
| **원데이터** | 일별 avgTa·maxTa·minTa·GDD |

**6개 온도 지표 (전 시트 공통 순서):**

| # | 지표 | 월별 집계 |
| :---: | :--- | :--- |
| 1 | 평균기온 | 일평균의 평균 |
| 2 | 최고기온 | 일최고의 최댓값 |
| 3 | 최저기온 | 일최저의 최솟값 |
| 4 | 최고기온 평균 | 일최고의 평균 |
| 5 | 최저기온 평균 | 일최저의 평균 |
| 6 | 적산온도 | 일 GDD의 합 |

### 권역별 집계 방식

권역 값은 해당 권역에 속한 **ASOS 지점의 단순 산술 평균**입니다. 기상청(KMA) 공식 통계와는 방법론이 다르므로 값이 동일하지 않습니다. 자세한 비교는 [WEATHER_THEORY.md 6장 및 부록 B](docs/WEATHER_THEORY.md#6-다지점-집계-방법론)를 참조하세요.

📖 자세한 해석 방법 → [docs/RESULTS_GUIDE.md](docs/RESULTS_GUIDE.md)

---

## 🌿 지원 작물 목록

`--crop` 파라미터에 `crop_id`를 입력합니다.

| 분류 | crop_id |
| :--- | :--- |
| 과수 | `apple` `pear` `cherry` `citrus` `grape` `peach` `plum` `persimmon` `kiwi` `blueberry` |
| 베리·딸기 | `strawberry` |
| 가지과 | `tomato` `pepper` `eggplant` |
| 박과 | `cucumber` `watermelon` `melon` `pumpkin` |
| 엽채류 | `kimchi_cabbage` `cabbage` `broccoli` `lettuce` `spinach` |
| 근채류·구근 | `potato` `sweet_potato` `radish` `carrot` `onion` `garlic` |
| 두과 | `soybean` `alfalfa` |
| 곡류 | `rice` `barley` `wheat` `maize` `sorghum` |
| 기타 | `cotton` `sunflower` `sesame` `turfgrass_cool` `turfgrass_warm` |

전체 파라미터(Tbase·Tupper·milestones·출처)는 `crops_gdd.csv` 참조.

---

## 📁 디렉터리 구조

```
cropweather/
├── README.md
├── LICENSE
├── requirements.txt
├── apikey.txt.example          ← 인증키 템플릿 (.gitignore에 apikey.txt 등록 필수)
├── .gitignore
│
├── solar_core.py               ← ASOS API 수집·파싱·캐싱 엔진
├── solar_multi.py              ← 일사·일조 종합 분석 → 7시트 Excel
├── temp_core.py                ← 온도·GDD 계산 엔진
├── temp_multi.py               ← 온도·적산온도 종합 분석 → 9시트 Excel
│
├── stations.py                 ← 66개 ASOS 지점·9개 권역 매핑
├── excel_util.py               ← 공통 엑셀 스타일·서식 유틸
├── crops_gdd.csv               ← 41개 작물 Tbase·Tupper (Paredes et al., 2025)
│
├── data/
│   └── raw/                    ← ASOS 수집 CSV 캐시 (.gitignore 처리)
│       └── .gitkeep
│
├── output/                     ← 분석 결과 Excel (.gitignore 처리)
│   └── .gitkeep
│
├── tests/
│   ├── test_solar.py           ← solar_core 핵심 계산 단위 테스트 (pytest)
│   └── test_temp.py            ← GDD 계산·도달일자 단위 테스트 (pytest)
│
└── docs/
    ├── WEATHER_THEORY.md       ← 기상 분석 이론 (일사+온도+적산온도 통합)
    ├── ARCHITECTURE.md         ← 코드 구조·함수 레퍼런스 (개발자용)
    └── RESULTS_GUIDE.md        ← 엑셀 결과 해석 방법
```

---

## 📚 문서

| 문서 | 대상 | 내용 |
| :--- | :--- | :--- |
| [WEATHER_THEORY.md](docs/WEATHER_THEORY.md) | 연구자·개발자 | 일사·기온·GDD 이론, 3가지 GDD 방법 비교, 다지점 집계 방법론 |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | 개발자·연구자 | 함수 레퍼런스·데이터 흐름·모듈 의존 관계·확장 방법 |
| [RESULTS_GUIDE.md](docs/RESULTS_GUIDE.md) | 현장 사용자 | 엑셀 시트별 해석·설정 시트 사용법·주의사항 |

**WEATHER_THEORY.md 주요 구성:**

| 장 | 주제 |
| :---: | :--- |
| 1~2 | 일사·일조 분석 / 온도 분석 |
| 3~4 | 적산온도(GDD) 계산 / 도달 일자 산출 |
| 5 | 작물별 기준온도 라이브러리 (41개 작물) |
| 6 | 다지점 집계 방법론 — 본 시스템 vs KMA |
| 7~8 | 동기간 비교 / 데이터 소스·캐싱 |
| 부록 A | GDD 3가지 방법 실측 비교 (ASOS 258,000건) |
| 부록 B | 권역 집계 방식 비교 — 본 시스템 vs KMA (제주 사례) |

---

## 🔗 관련 저장소

| 저장소 | 설명 |
| :--- | :--- |
| [cropwater](https://github.com/krpeace/cropwater) | FAO-56 기반 노지 작물 물수지 분석 도구 |
| cropweather (본 저장소) | 일사·일조·온도·적산온도 분석 도구 |
| farmos-soil | 토양 분석 (흙토람 API + 센서) — 예정 |
| farmos-market | 도매시장·시세 (aT·KAMIS) — 예정 |

> `crop_id`를 공유 키로 사용하여 cropwater의 `crops_library.csv`와 cropweather의 `crops_gdd.csv`가 연결됩니다.

---

## 🔬 연구 과제 및 기여

| 항목 | 내용 |
| :--- | :--- |
| 과제명 | 노지 작물 수분스트레스 진단·정밀자동관수 패키지기술 산업화 |
| 과제번호 | RS-2025-02305238 |
| 주관기관 | 주식회사 파모스 |
| 지원기관 | 농림축산식품부 |

**참고 문헌**
- Paredes, P. et al. (2025). "Base and upper temperature thresholds…" *Agric. Water Manag.*, 319, 109755.
- McMaster, G.S. & Wilhelm, W.W. (1997). "Growing degree-days…" *Agric. For. Meteorol.*, 87(4).
- Allen, R.G. et al. (1998). *FAO Irrigation and Drainage Paper No. 56.* FAO, Rome.
- KMA ASOS API: https://data.go.kr

새 작물 추가는 `crops_gdd.csv`에 행 하나를 추가하는 것으로 충분합니다. PR과 Issue 모두 환영합니다.

---

## 📄 라이선스

MIT License © 2026 주식회사 파모스 (FarmOS Corp.)
