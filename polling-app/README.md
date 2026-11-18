# Polling App

질문과 선택지를 등록하고 사용자가 투표 결과를 확인할 수 있는 Django 웹 앱입니다.

## 기능

- 게시일 내림차순으로 최근 질문 5개 표시
- 질문 상세와 선택지 라디오 버튼 표시
- 선택한 항목의 득표 수를 1 증가시킨 후 결과 페이지로 redirect
- 선택하지 않은 요청에는 오류 메시지 표시
- Django Admin에서 질문과 선택지를 함께 등록
- 관리자 검색, 게시일 필터, 최근 하루 게시 여부 표시

## 데이터 구조

`Question(question_text, pub_date)`와 `Choice(question, choice_text, votes)`의 일대다 관계입니다. 질문 삭제 시 연결된 선택지도 CASCADE 삭제합니다. `votes`의 기본값은 0입니다. `was_published_recently()`는 현재부터 하루 전까지의 게시 시각인지 판단합니다.

## 요청 흐름

```text
/polls/ → 질문 선택 → /polls/<id>/
                      ↓ choice_select POST
                /polls/<id>/vote/
                      ↓ 득표 수 저장
                /polls/<id>/results/
```

## 주요 파일

- `polls/models.py`: Question, Choice 및 최근 게시 여부
- `polls/views.py`: 목록/상세/결과/투표 처리
- `polls/admin.py`: 선택지 inline, 검색과 필터
- `polls/templates/polls/`: 화면
- `polls/static/polls/`: CSS와 배경 이미지

## 실행

```bash
cd polling-app
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 8000
```

`/admin/`에서 Question과 Choice를 만든 뒤 `/polls/`에서 투표합니다. 초기 데이터가 없으므로 첫 실행 시 목록은 비어 있습니다.

## 제약과 개선 지점

- 같은 사용자의 반복 투표를 막지 않습니다. 선거/공식 설문에 사용할 수 있는 인증 시스템이 아닙니다.
- `votes += 1` 후 저장하는 방식이라 동시 요청 시 갱신 손실 가능성이 있습니다. 원자적 `F()` 갱신은 아직 구현하지 않았습니다.
- 미래 게시일 질문도 목록에 포함될 수 있습니다.
- 투표 예외 처리가 광범위한 `except`로 되어 있으며 POST 전용 데코레이터는 없습니다.
- `tests.py`에는 자동화된 테스트가 구현되어 있지 않습니다.
