# Face & Eye Detection

업로드한 정지 이미지에서 Haar Cascade로 얼굴과 눈을 검출하고, 검출 위치에 사각형을 그려 다시 보여주는 Django + OpenCV 앱입니다. 얼굴의 신원을 식별하는 얼굴 인식 서비스가 아닙니다.

## 처리 흐름

1. `ImageUploadForm`으로 설명과 이미지를 받습니다.
2. `ImageUploadModel`의 ImageField를 통해 `media/images/YYYY/MM/DD/`에 저장합니다.
3. `cv2.imread()`로 읽어 원본 가로가 640을 넘으면 가로 기준 축소하고, 그렇지 않고 세로가 480을 넘으면 세로 기준 축소합니다.
4. 회색조로 바꾸고 얼굴 Cascade를 `detectMultiScale(gray, 1.3, 5)`로 실행합니다.
5. 각 얼굴 ROI 안에서 눈 Cascade를 적용합니다.
6. 얼굴은 파란색, 눈은 초록색 박스로 표시하고 **동일 파일에 덮어씁니다**.

위 조건은 `if/elif`이므로 모든 비율에서 가로 640, 세로 480을 동시에 보장하는 리사이즈는 아닙니다.

## 구성

- `opencv_webapp/cv_functions.py`: 이미지 읽기, 크기 조절, 검출, 저장
- `opencv_webapp/forms.py`: 단순 업로드 폼과 모델 폼
- `opencv_webapp/models.py`: description, document, uploaded_at
- `opencv_webapp/views.py`: 업로드 및 검출 요청
- `media/haarcascade_frontalface_default.xml`: 얼굴 분류기
- `media/haarcascade_eye.xml`: 눈 분류기

원본 XML의 저작권 고지는 유지했습니다. 기존 업로드 사진은 제외했으므로 직접 사용할 권한이 있는 이미지를 준비하세요.

## 실행

```bash
cd face-eye-detection
python manage.py migrate
python manage.py runserver 8000
```

프로젝트 루트에서 실행해야 상대 경로 `MEDIA_ROOT_URL='.'`가 올바르게 해석됩니다.

| 경로 | 설명 |
|---|---|
| `/` | 첫 화면 |
| `/simple_upload/` | 파일 저장 후 URL 표시 |
| `/detect_face/` | 이미지 업로드 및 검출 결과 |
| `/admin/` | 업로드 레코드 관리 |

## 제약

- 웹캠, 동영상, 실시간 스트리밍 기능은 없습니다.
- 원본 이미지 보존 기능이 없고 검출 결과로 업로드 파일을 덮어씁니다.
- 유효하지 않은 POST 폼에서 명시적인 응답을 반환하지 않는 분기가 있습니다.
- 파일 크기 제한, 이용자 인증, 사용자별 접근 제어, 검출 실패 UI가 없습니다.
- XML 누락이나 이미지 처리 오류에 대한 예외 처리가 충분하지 않습니다.
- 테스트 템플릿만 있었으며 실제 인물 사진의 검출 정확도/성능은 측정하지 않았습니다.
