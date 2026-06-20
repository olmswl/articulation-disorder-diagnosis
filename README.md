# articulation-disorder
# trigger rebuild
# 🗣️ 조음장애 자가 진단 웹 서비스
**MediaPipe 기반 실시간 입모양 분석으로 발음 정확도를 수치화하는 컴퓨터 비전 프로젝트**

[![Streamlit App](https://img.shields.io/badge/Streamlit-Live%20Demo-FF4B4B?logo=streamlit)](https://github.com/olmswl/articulation-disorder-diagnosis)
[![Python](https://img.shields.io/badge/Python-100%25-blue?logo=python)](https://github.com/olmswl/articulation-disorder-diagnosis)

> 졸업 프로젝트 · 개발기간: (학기 입력) · 컴퓨터 비전 / 풀스택 단독 구현

## 👋 Intro

조음장애 환자, 보호자, 언어치료사 모두를 위한 발음 자가 진단 시스템을 만들었습니다.
OpenCV와 MediaPipe로 입모양을 분석하고, 정답 영상과의 유사도를 수치화해
"내 발음이 얼마나 정확한지" 즉시 피드백을 받을 수 있도록 설계했습니다.

## 📌 프로젝트 소개

언어 장애 등록 인구는 매년 증가하고 있지만, 기존 치료법(대면 치료, 화상통화 치료)은 **지리적·경제적·시간적 접근성**에 한계가 있습니다.

특히 지방 거주자의 경우 조음음운 장애 전문가 수가 수도권 대비 현저히 적고(예: 포항 1명 vs 서울 송파 7명), 재활 치료 특성상 비용 부담이 크며, 학생·직장인은 정기적인 대면/화상 치료를 지속하기 어렵습니다.

본 프로젝트는 **사용자가 업로드한 발음 영상을 컴퓨터 비전으로 분석하여 조음 정확도를 수치화하고, 즉각적인 피드백을 제공하는 시스템**을 통해 이러한 한계를 보완하고자 합니다.

## 🎯 목표

- 입모양 좌표 기반의 **수치화된 발음 정확도 평가** 제공
- 시간·공간 제약 없이 **언제 어디서든 접근 가능한** 진단 환경 구축
- 전문가 대면 치료 대비 **치료 비용 절감**
- 조음장애 환자뿐 아니라 **언어치료사, 보호자**도 활용 가능한 도구 제공

## 🧑‍⚕️ 주요 사용자

| 사용자 | 활용 방식 |
|---|---|
| 조음장애 환자 | 스스로 발음 연습 및 정확도 확인 |
| 언어치료사 | 환자 진단 보조 및 객관적 수치 자료 확보 |
| 보호자 | 자녀/가족의 발음 발달 추적 |

---

## 🛠️ 기술 스택

**Frontend**
- `Streamlit` — 영상 업로드, 결과 시각화, 사용자 인터페이스

**Backend / Computer Vision**
- `OpenCV` — 영상 입출력 및 프레임 전처리
- `MediaPipe FaceMesh` — 얼굴 468개 랜드마크 추출 (입술 좌표 추출)
- `NumPy` — 좌표 정규화 및 유클리드 거리 기반 유사도 계산

**Speech**
- `STT (Speech-to-Text)` — 사용자 발화 텍스트 인식
- `Google TTS` — 정답 문장 음성 합성 (base64 인코딩 후 재생)
- `difflib.SequenceMatcher` — STT 결과와 정답 문장 간 텍스트 유사도 계산

**Data**
- `Pandas` — 사용자별 분석 기록 저장 및 조회 (CSV)

**Deployment**
- Streamlit Community Cloud (`apt.txt`, `packages.txt`, `runtime.txt` 기반 환경 구성)

---

## ⚙️ 시스템 아키텍처

```
[사용자 영상 업로드]
        │
        ▼
[OpenCV: 프레임 읽기 + 명암 대비 보정 (CLAHE, LAB 색공간)]
        │
        ▼
[MediaPipe FaceMesh: 얼굴 검출 → 입술 랜드마크 추출]
        │
        ▼
[좌표 정규화 (위치/크기 무관하게 형태만 비교 가능하도록 0~1 스케일링)]
        │
        ▼
[기준 영상 좌표와 유클리드 거리 기반 유사도 계산] ──► 조음 정확도 (30~100점)
        │
[STT 결과 vs 정답 문장 텍스트 유사도 계산] ──► 발화 정확도 (0~100%)
        │
        ▼
[종합 피드백 생성 + TTS 정답 음성 제공]
        │
        ▼
[분석 결과 저장 (CSV) → 사용자별 기록 조회/삭제 가능]
```

## 🧩 핵심 모듈

| 모듈 | 설명 |
|---|---|
| **영상 처리 모듈** | OpenCV로 영상을 읽고 MediaPipe로 입술 위치 감지 |
| **좌표 분석 모듈** | 추출된 입술 랜드마크를 NumPy로 수치 분석 |
| **유사도 계산 모듈** | 사용자/정답 영상의 입술 좌표를 유클리드 거리로 비교 |
| **데이터 관리 모듈** | 발음 결과 및 기록을 사용자별로 저장·관리 |
| **웹 인터페이스 모듈** | Streamlit 기반 직관적 UI 제공 |

## 🔍 핵심 기술 상세

### 1. 얼굴 랜드마크 추출 (MediaPipe FaceMesh)
- 얼굴 1명 기준, 검출 신뢰도 0.7 / 추적 신뢰도 0.5로 설정
- 영상 프레임마다 468개 얼굴 랜드마크 중 입술 영역 좌표만 선별 추출
- BGR → LAB 색공간 변환 후 CLAHE(Contrast Limited Adaptive Histogram Equalization)를 적용해 저조도·노이즈 환경에서도 안정적인 랜드마크 인식 확보

### 2. 좌표 정규화
- 사용자마다 다른 얼굴 위치·크기를 보정하기 위해 좌표를 0~1 범위로 정규화
- 분모가 0이 되는 경우(가로/세로 길이 0)에 대비해 `1e-6`으로 대체하여 연산 안정성 확보

### 3. 유사도 계산
- 사용자 영상과 기준 영상의 입술 좌표를 프레임 단위로 비교 (최소 길이 기준 정렬)
- `np.linalg.norm`으로 유클리드 거리를 계산, `100 - 거리 × 65` 공식으로 점수화
- 점수는 30~100점으로 클리핑, 평균 90점 이상 시 100점 부여
- 좌표 비교가 불가능한 경우 0점 반환으로 실패 케이스 명시적 처리

---

## 💻 주요 기능

- ✅ 닉네임/이메일 기반 간편 로그인 (세션 상태 관리)
- ✅ 진단 문장 선택 + 문장별 유의 음소 안내
- ✅ 모바일 환경 즉시 촬영 업로드 지원 (mp4, mov)
- ✅ 시간 기반 파일명 자동 생성으로 중복 방지
- ✅ 조음 정확도 / 발화 정확도 분리 시각화 (`st.metric`)
- ✅ 기준 발음 TTS 음성 재생
- ✅ 사용자별 분석 기록 자동 저장 및 최신순 조회
- ✅ 분석 기록 삭제 기능

---

## 📂 프로젝트 구조

```
articulation-disorder/
├── .devcontainer/                 # 개발 환경 설정
├── articulation-disorder/         # 메인 애플리케이션 코드
├── generate_reference_coords.py   # 기준 영상 좌표 생성 스크립트
├── requirements.txt                # Python 패키지 의존성
├── apt.txt                         # 시스템 레벨 의존성 (Streamlit Cloud 배포용)
├── packages.txt                    # 추가 패키지 정의
├── runtime.txt                     # Python 런타임 버전
└── README.md
```

## 🚀 실행 방법

```bash
# 저장소 클론
git clone https://github.com/olmswl/articulation-disorder-diagnosis.git
cd articulation-disorder

# 의존성 설치
pip install -r requirements.txt

# 기준 좌표 생성 (최초 1회)
python generate_reference_coords.py

# 앱 실행
streamlit run app.py
```

---

## 👤 담당 역할

> 핵심 역할: 단독 개발, 컴퓨터 비전 파이프라인 설계 및 풀스택 구현

- 🎥 컴퓨터 비전 파이프라인 설계 및 구현 (OpenCV, MediaPipe FaceMesh)
- 📐 입술 좌표 정규화 및 유클리드 거리 기반 유사도 알고리즘 개발
- 🖥️ Streamlit 기반 웹 인터페이스 및 사용자 데이터 관리 시스템 구축
- 🔊 STT/TTS 연동 및 점수 구간별 종합 피드백 로직 설계

---

## 📈 향후 개선 방향

- 입술뿐 아니라 혀/턱 등 추가 조음기관 좌표 분석 확장
- 음소 단위 세부 피드백 고도화
- 다국어 문장 데이터셋 확장

---

## 📞 Contact

- 이메일 : 
- 깃허브 : [github.com/olmswl/articulation-disorder-diagnosis](https://github.com/olmswl/articulation-disorder-diagnosis)
- 링크드인 :
