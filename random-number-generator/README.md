# Random Number Generator

Django ModelForm으로 제목과 설명을 입력받아 랜덤 번호 묶음을 생성하고 SQLite에 저장하는 웹 앱입니다. Django 앱 이름은 `lotto`입니다.

## 기능과 동작

1. `/lotto/new/`에서 `name`과 `text`를 입력합니다.
2. `PostForm.is_valid()`를 통과하면 `GuessNumbers.generate()`를 호출합니다.
3. 1~45 목록을 섞고 앞의 6개를 정렬합니다. 기본 `num_lotto=5`이므로 다섯 묶음을 생성합니다.
4. 결과를 줄바꿈으로 연결한 문자열과 생성 시각을 저장합니다.
5. 목록으로 이동하고 개별 상세 페이지에서 결과를 확인합니다.

한 묶음 안에서는 중복 숫자가 없지만 서로 다른 묶음 사이 중복은 가능합니다. 입력 폼에는 묶음 수 필드가 없으므로 웹 화면에서 개수를 지정하는 기능은 없습니다.

## 데이터 모델

| 필드 | 의미 |
|---|---|
| name | 제목, 최대 24자 |
| text | 설명, 최대 255자 |
| lottos | 생성 결과 문자열, 최대 255자로 선언 |
| num_lotto | 생성할 6개 숫자 묶음 수, 기본 5 |
| update_date | 마지막 생성 시각 |

## 주요 파일

- `lotto/models.py`: 숫자 생성과 저장
- `lotto/forms.py`: 제목/설명 ModelForm
- `lotto/views.py`: 목록, 생성, 상세 화면
- `lotto/templates/lotto/`: HTML 템플릿
- `site_1/urls.py`: 라우팅
- `lotto/tests.py`: 생성 결과 문자열 길이 테스트

## 실행

저장소 루트에서 공통 requirements 설치 후 실행합니다.

```bash
cd random-number-generator
python manage.py migrate
python manage.py runserver 8000
```

| 경로 | 기능 |
|---|---|
| `/lotto/` | 전체 목록 |
| `/lotto/new/` | GET 입력 폼 / POST 생성 |
| `/lotto/<id>/detail/` | 개별 결과 |
| `/admin/` | 관리자 |

## 제약

- `/hello/`는 조건 없는 `objects.get()`을 포함한 미완성 보조 경로이며 일반 사용 경로가 아닙니다.
- 없는 ID의 상세 조회에 404 처리가 없습니다.
- 잘못된 생성 폼은 기존 오류/입력값을 보존하지 않고 빈 폼으로 다시 표시됩니다.
- 로그인, 사용자별 소유권, 수정/삭제 UI는 구현하지 않았습니다.
- 포함된 단위 테스트는 결과 문자열 길이를 확인하며 숫자 개수, 범위, 중복 검사를 포함하지 않습니다.
- 실제 복권 구매나 당첨 예측 기능이 없는 무작위 숫자 생성 예제입니다.
