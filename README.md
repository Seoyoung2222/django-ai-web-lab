# Django AI Web Lab

Django의 폼, ORM, 템플릿 기반 웹 개발에서 이미지 처리와 한국어 감성 분석까지 연결한 네 개의 독립 웹 프로젝트입니다. 원본 ZIP에 누적된 동일 프로젝트는 한 번씩만 정리하고, 실제 기능에 맞춰 폴더 이름을 지정했습니다.

## 프로젝트 한눈에 보기

| 프로젝트 | 핵심 구현 | 접속 경로 | 상세 설명 |
|---|---|---|---|
| 랜덤 번호 생성 | 1~45 사이 중복 없는 숫자 6개를 기본 5세트 생성하고 저장 | `/lotto/` | [random-number-generator](random-number-generator/README.md) |
| 질문과 투표 | 질문 목록, 선택지 투표, 득표 결과, 관리자 화면 | `/polls/` | [polling-app](polling-app/README.md) |
| 얼굴과 눈 검출 | 이미지 업로드, Haar Cascade 검출, 검출 영역 표시 | `/detect_face/` | [face-eye-detection](face-eye-detection/README.md) |
| 한국어 감성 분석 | KoBERT 기반 이진 감성 분류 모델 구성과 추론 흐름 | `/predict_sentiment/bert_input/` | [korean-sentiment-analysis](korean-sentiment-analysis/README.md) |

각 폴더에 별도의 `manage.py`, 설정 모듈, SQLite DB가 있습니다. 하나의 서버에서 네 앱이 함께 실행되는 구조는 아닙니다. 폴더 이름은 기능 중심으로 바꾸고 Django 내부 패키지 이름(`site_1`, `site_2`, `cv_project`, `bert_project`)은 import 호환성을 위해 유지했습니다.

## 구조

```text
django-ai-web-lab/
├── random-number-generator/   # ModelForm과 랜덤 번호 저장
├── polling-app/               # Question-Choice 관계와 투표 처리
├── face-eye-detection/        # OpenCV 이미지 처리
├── korean-sentiment-analysis/ # KoBERT 모델 로딩과 추론
├── requirements.txt          # 번호 생성 / 투표 / 이미지 검출 검증 환경
├── requirements-original.txt # 원본 공통 환경 기록, 완전한 NLP 환경은 아님
├── .env.example
└── docs/VERIFICATION.md
```

## 실행 환경과 설치

아래 공통 환경은 Python 3.12.4에서 번호 생성, 투표, 이미지 검출 프로젝트를 검증하기 위한 환경입니다. 원본의 Django 5.1.2를 유지했으며, 서버에서 GUI를 사용하지 않으므로 OpenCV는 headless 4.10.0.84를 사용합니다. 역사적 개발 환경을 기록한 것이며 최신 보안 패치가 적용된 운영 환경을 의미하지 않습니다.

```bash
git clone https://github.com/Seoyoung2222/django-ai-web-lab.git
cd django-ai-web-lab
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
export DJANGO_SECRET_KEY="$(python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())')"
```

Windows에서는 가상환경을 `.venv\Scripts\activate`로 활성화하고 환경변수도 해당 셸 문법으로 지정합니다. `.env.example`은 참고 파일이며 자동으로 읽히지 않습니다.

```bash
cd polling-app                 # 실행할 프로젝트 선택
python manage.py migrate
python manage.py createsuperuser  # 관리자 기능이 필요한 경우
python manage.py runserver 8000
```

이후 `http://127.0.0.1:8000/polls/`에 접속합니다. 질문이 없으면 `/admin/`에서 질문과 선택지를 등록합니다. 여러 프로젝트를 동시에 열려면 프로젝트마다 8000, 8001 등 서로 다른 포트를 사용하세요.

**한국어 감성 분석은 추가 모델, 데이터, ML 의존성이 필요합니다.** 공통 requirements만 설치해서 추론까지 완료할 수 없습니다. [별도 준비 및 제약](korean-sentiment-analysis/README.md)을 먼저 확인하세요.

## 공통 요청 처리 구조

```text
브라우저 → URL 라우팅 → View → Form / Model / 처리 함수 → Template → 응답
                                   ↓
                            프로젝트별 SQLite
```

- 번호 생성: ModelForm 검증 후 모델의 `generate()`에서 숫자와 생성 시각을 저장합니다.
- 투표: 질문에 속한 Choice를 찾고 득표 수를 증가시킨 후 결과 화면으로 이동합니다.
- 이미지 검출: ImageField에 파일을 저장한 뒤 OpenCV로 같은 파일에 검출 박스를 그립니다.
- 감성 분석: 전처리와 토큰화 후 TensorFlow 모델로 부정/긍정 클래스와 softmax 최대값을 계산합니다.

## 안전한 저장소 정리

원본 압축파일과 기존 작업 폴더는 변경하지 않았습니다. 새 저장소에 필요한 코드, 템플릿, 정적 파일, 마이그레이션과 Cascade XML만 복사했습니다.

제외한 항목:
- 가상환경, 설치 패키지, 캐시, macOS 메타데이터
- SQLite DB와 기존 계정/세션/업로드 내역
- 업로드된 인물 사진 및 생성된 이미지
- 약 369 MB의 KoBERT 가중치와 약 20.6 MB의 리뷰 데이터
- 중복 collectstatic 출력

설정의 고정 Django SECRET_KEY를 제거하고 환경변수 또는 프로세스별 임시 키로 대체했습니다. 키를 지정하지 않으면 서버 재시작 때 세션이 무효화될 수 있습니다. 기존 DB는 커밋하지 않으며 `migrate`로 새 DB를 만듭니다.

## 원본과 달라진 부분

1. 기능별 최상위 폴더 이름 및 README 추가
2. 가상환경, 실행 데이터와 비밀 설정 분리
3. 로컬 실행용 DEBUG/ALLOWED_HOSTS 환경변수 설정 추가
4. 감성 분석 view의 누락된 `bert_load` import 추가
5. 검증용 공통 의존성 파일과 검증 결과 기록

기능을 확장하거나 완성된 운영 서비스로 재작성하지 않았습니다. 예외 처리, 동시 투표, 중복 투표 방지, 모델 캐싱 등 개선이 필요한 부분은 각 README에 명시했습니다. [검증 결과](docs/VERIFICATION.md)는 수행한 검사와 미수행 항목을 구분합니다.

## 운영 및 라이선스 주의

로컬 학습/포트폴리오용 코드입니다. 공개 서버에서는 최신 지원 버전 검토, HTTPS, 비밀키 관리, DEBUG 비활성화, 허용 호스트, 업로드 검증과 접근 통제를 별도로 갖춰야 합니다. 번호 생성 기능은 무작위 샘플링 예제이며 예측 능력이나 당첨 가능성 향상을 제공하지 않습니다.

`bert_tokenizer.py`의 Apache 2.0 고지와 OpenCV Cascade XML의 원저작권 고지를 유지했습니다. 원본 데이터, 이미지, 모델의 재배포 권한은 별도 확인이 필요하므로 리뷰 데이터와 가중치는 저장소에 포함하지 않았습니다. 저장소 전체에 임의의 오픈소스 라이선스를 부여하지 않았습니다.
