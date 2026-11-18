# Django AI Web Lab

**웹 애플리케이션에 데이터 처리와 AI 추론을 연결한 Django 프로젝트 모음**

사용자 입력을 받아 데이터를 저장하는 웹 서비스부터 이미지 검출, 한국어 감성 분석까지 구현한 네 개의 프로젝트입니다. Django의 URL 라우팅, 폼 검증, ORM, 템플릿을 공통 기반으로 사용하며 각 프로젝트는 독립적으로 실행됩니다.

## 프로젝트

| 프로젝트 | 주요 기능 | 구현 기술 |
|---|---|---|
| [Random Number Generator](random-number-generator/README.md) | 랜덤 번호 생성, 결과 저장, 목록 및 상세 조회 | ModelForm, ORM, SQLite |
| [Polling App](polling-app/README.md) | 질문별 투표, 득표 결과, 관리자 검색과 필터 | 관계형 모델, POST 처리, Django Admin |
| [Face & Eye Detection](face-eye-detection/README.md) | 이미지 업로드, 얼굴과 눈 검출, 결과 이미지 표시 | ImageField, OpenCV, Haar Cascade |
| [Korean Sentiment Analysis](korean-sentiment-analysis/README.md) | 리뷰 데이터 미리보기, 한국어 문장 전처리, 감성 분류 추론 코드 | pandas, KoBERT, TensorFlow, SentencePiece |

## 기술 구성

- **Backend:** Python, Django
- **Data:** SQLite, Django ORM, pandas, NumPy
- **Computer Vision:** OpenCV Haar Cascade
- **NLP:** KoBERT, Hugging Face Transformers, TensorFlow/Keras, SentencePiece
- **Frontend:** Django Templates, HTML, CSS

## 구현 구조

```text
브라우저 → URL 라우팅 → View → Form / Model / 처리 함수 → Template → 응답
                                   ↓
                       프로젝트별 데이터베이스 또는 파일
```

### 입력부터 저장까지

번호 생성 앱은 ModelForm으로 제목과 설명을 검증한 후 모델 메서드에서 번호와 생성 시각을 저장합니다. 투표 앱은 Question과 Choice의 일대다 관계를 이용해 질문별 선택지와 득표 수를 관리합니다.

### 이미지 처리와 웹 화면 연결

이미지 검출 앱은 ImageField에 저장한 파일을 OpenCV로 읽습니다. 얼굴 영역과 해당 영역 안의 눈을 검출하고, 검출 위치를 표시한 이미지를 웹 화면에 반환합니다.

### 한국어 텍스트의 추론 흐름

감성 분석 앱은 문장 전처리, 토큰화, 길이 조정, 모델 입력 배열 구성, 분류 결과 표시로 이어지는 흐름을 구성합니다. KoBERT 출력에 이진 분류 head를 연결하며 별도 학습 가중치가 필요합니다. 실제 모델 실행 환경과 준비 사항은 프로젝트 README에 설명되어 있습니다.

## 디렉터리

```text
django-ai-web-lab/
├── random-number-generator/
│   ├── lotto/                # 번호 생성, 폼, 화면
│   ├── site_1/               # Django 설정과 URL
│   └── manage.py
├── polling-app/
│   ├── polls/                # 질문, 선택지, 투표
│   ├── site_2/
│   └── manage.py
├── face-eye-detection/
│   ├── opencv_webapp/        # 업로드와 검출 처리
│   ├── media/                # Haar Cascade XML
│   ├── cv_project/
│   └── manage.py
├── korean-sentiment-analysis/
│   ├── main/                 # 토큰화, 모델 로딩, 추론
│   ├── bert_project/
│   └── manage.py
├── requirements.txt
└── .env.example
```

## 로컬 실행

번호 생성, 투표, 이미지 검출 앱의 공통 환경은 Python 3.12.4, Django 5.1.2, NumPy 1.26.4, OpenCV headless 4.10.0.84, Pillow 10.4.0입니다. 감성 분석에는 별도 ML 의존성과 모델 파일이 필요합니다.

```bash
git clone https://github.com/Seoyoung2222/django-ai-web-lab.git
cd django-ai-web-lab
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
export DJANGO_SECRET_KEY="$(python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())')"

# 실행할 프로젝트를 선택합니다.
cd polling-app
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 8000
```

Windows에서는 `.venv\Scripts\activate`로 가상환경을 활성화하고 환경변수는 사용 중인 셸 문법에 맞춰 지정합니다. `.env.example`은 설정 예시이며 자동으로 로드되지 않습니다.

| 프로젝트 | 기본 접속 경로 | 시작 전 준비 |
|---|---|---|
| 번호 생성 | `http://127.0.0.1:8000/lotto/` | migration 실행 |
| 투표 | `http://127.0.0.1:8000/polls/` | `/admin/`에서 질문과 선택지 등록 |
| 얼굴과 눈 검출 | `http://127.0.0.1:8000/detect_face/` | 사용 권한이 있는 이미지 준비 |
| 감성 분석 | `http://127.0.0.1:8000/predict_sentiment/bert_input/` | [모델, 데이터, ML 환경 준비](korean-sentiment-analysis/README.md#실행-준비) |

각 프로젝트에서 migration을 실행하면 독립된 SQLite DB가 생성됩니다. 여러 프로젝트를 동시에 실행할 때는 서로 다른 포트를 지정합니다.

### 환경변수

| 변수 | 용도 | 기본 동작 |
|---|---|---|
| `DJANGO_SECRET_KEY` | 세션 등 Django 서명 키 | 미설정 시 프로세스별 임시 키 생성 |
| `DJANGO_DEBUG` | 개발 모드 | `1` |
| `DJANGO_ALLOWED_HOSTS` | 쉼표로 구분한 허용 호스트 | localhost, 127.0.0.1, [::1] |

지속적인 세션 사용을 위해 SECRET_KEY를 지정해야 합니다. 공개 서버에 배포할 경우 지원되는 최신 의존성, HTTPS, DEBUG 비활성화, 업로드 검증 및 접근 제어를 별도로 구성해야 합니다.

## 프로젝트 범위

웹 개발과 AI 기능 연동을 다루는 학습 프로젝트입니다. 인증 기반 중복 투표 방지, 동시성 제어, 모델 캐싱 등은 구현 범위에 포함하지 않았으며 각 README에 제한 사항을 설명했습니다. 감성 분석의 실제 모델 로딩과 end-to-end 추론은 검증되지 않았습니다.

KoBERT 가중치와 리뷰 데이터는 저장소에 포함되어 있지 않습니다. 모델과 데이터 사용 시 각각의 이용 조건을 확인해야 합니다. `bert_tokenizer.py`와 OpenCV Cascade XML에는 해당 구성 요소의 라이선스 고지가 포함되어 있습니다.
