# Korean Sentiment Analysis

Django 화면에서 한국어 문장을 입력하고 KoBERT 기반 이진 분류 모델로 부정/긍정 결과를 표시하도록 구성한 프로젝트입니다. **모델 가중치와 데이터가 별도로 필요하며, 이 저장소에서 실제 추론까지 검증된 상태는 아닙니다.**

## 코드에 구현된 흐름

### 입력 화면

`main/views.py`의 `bert_input()`이 `data/naver_shopping.txt`를 탭 구분 ratings/reviews로 읽습니다. 리뷰 중복을 제거하고 평점이 3을 초과하면 1, 아니면 0으로 라벨을 붙인 뒤 처음 5행을 HTML로 표시합니다. 이 처리는 데이터 미리보기이며 학습 실행이 아닙니다.

### 모델 구성

`main/bert_load.py`는 `TFBertModel.from_pretrained("monologg/kobert", from_pt=True)`로 기본 모델을 불러옵니다. 최대 길이 128의 token IDs, attention masks, token type IDs를 입력으로 사용하고 pooled output에 Dropout(0.2)와 Dense(2, softmax)를 연결합니다. Adam 학습률 1e-5, sparse categorical cross entropy로 compile한 후 로컬 H5 가중치를 로드합니다. 저장소에 모델을 학습하는 코드나 재현 가능한 학습 결과는 없습니다.

### 추론

`main/inference_bert.py`에서 문장에 정규식을 적용해 문자 범위를 정리한 후 KoBERT tokenizer로 padding/truncation을 수행합니다. 세 입력을 `(1, 128)` 배열로 만들고 `model.predict()`를 호출합니다. argmax 0은 부정, 1은 긍정으로 매핑하고 softmax 최대값을 백분율로 표시합니다. 표시 숫자는 검증된 정확도나 보정된 신뢰도를 의미하지 않습니다.

## 주요 파일

| 파일 | 역할 |
|---|---|
| `main/bert_tokenizer.py` | SentencePiece 기반 KoBERT tokenizer, 원저작권/Apache 2.0 고지 유지 |
| `main/bert_load.py` | 기본 모델과 tokenizer 로딩, 분류 head 구성, H5 가중치 로딩 |
| `main/inference_bert.py` | 전처리, 토큰화, 추론 결과 문자열 |
| `main/views.py` | 데이터 미리보기와 HTTP 입력/결과 |
| `main/templates/main/` | 입력 및 결과 화면 |
| `bert_project/settings.py` | DATA_ROOT, MODEL_ROOT, 웹 설정 |

## 필요한 로컬 파일

```text
korean-sentiment-analysis/
├── data/naver_shopping.txt
└── models/best_kobert_weights.h5
```

원본 ZIP에는 두 파일이 있지만, 가중치가 약 369 MB이고 리뷰 데이터의 재배포 권한을 확인하지 않아 Git 커밋에서는 제외했습니다. `.gitignore`도 두 경로의 데이터와 가중치를 제외합니다. 사용 권한을 확인한 원본을 위 경로에 별도로 배치해야 합니다. 잘못된 가중치로 대체하거나 빈 파일을 만들면 실행되지 않습니다.

## 의존성과 실행 준비

공통 `requirements.txt` 외에 코드상 `pandas`, `tensorflow`, `transformers`(TFBertModel 지원 버전), `sentencepiece`, `torch`가 필요합니다. `from_pt=True` 변환에는 PyTorch가 사용됩니다. 원본 공통 requirements에는 이 ML 환경 버전이 빠져 있어 TensorFlow/Keras/Transformers 간 호환되는 정확한 조합은 확정하지 않았습니다. 환경 구축 시 별도 가상환경을 권장합니다.

준비 순서:
1. TensorFlow와 해당 KoBERT 코드가 호환되는 Python/ML 패키지 조합을 구성합니다.
2. 사용 권한이 있는 데이터와 가중치를 위 위치에 둡니다.
3. 모델과 tokenizer 최초 다운로드에 네트워크와 저장 공간이 필요합니다.
4. `python manage.py check`, `python manage.py migrate`를 실행합니다.
5. `python manage.py runserver 8000`으로 시작하고 `/predict_sentiment/bert_input/`을 엽니다.
6. 입력 문장을 `target_sentence` 필드로 `/predict_sentiment/bert_predict/`에 POST합니다.

## 저장소 정리 시 수정

원본 view에서 호출하던 `bert_load`의 import가 빠져 있어 `from . import inference_bert, bert_load`로 보완했습니다. 고정 SECRET_KEY를 제거하고 로컬 호스트와 DEBUG를 환경변수로 설정하도록 정리했습니다. 모델 구조나 예측 로직은 바꾸지 않았습니다.

## 알려진 한계

- 요청할 때마다 tokenizer와 모델을 로드합니다. 모델 캐싱이나 비동기 추론은 없습니다.
- 데이터 미리보기도 요청마다 전체 파일을 읽습니다.
- 빈 입력/GET 요청, 모델/데이터 누락에 대한 사용자 오류 화면은 없습니다.
- 원본 tokenizer가 사용하는 API 및 다운로드 주소의 현재 호환성은 별도 확인이 필요합니다.
- 데이터 전처리 코드만으로 모델 학습 방법, 데이터 분할, 정확도 등의 실험 결과를 증명할 수 없습니다.
- 실제 모델 다운로드, 대용량 가중치 로딩, end-to-end 추론은 이번 정리에서 실행하지 않았습니다.
