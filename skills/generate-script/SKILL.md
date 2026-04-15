---
name: generate-script
description: 완성된 씬 데이터(프롬프트, 내레이션, scene_meta)를 마스터 대본 JSON, 마크다운 대본, CSV 파일로 저장한다. JSON은 FlowGenie/TTS/SceneWeaver가 소비하는 마스터 포맷이다. 모든 씬 상세가 확정된 후 자동으로 실행되거나, '파일 생성해줘', '저장해줘'라고 요청하면 사용한다.
---

# 출력 파일 생성

완성된 씬 데이터를 3가지 형식으로 저장한다.

## 사전 조건

- prompt-craft에서 모든 씬의 상세가 확정되어야 한다
- 시리즈 가이드의 video_meta 정보 (있는 경우)

## 생성 절차

### Step 1: 출력 폴더 생성

`output/ch{NN}/` 폴더를 생성한다 (없으면).

### Step 2: JSON 생성 — 마스터 대본

`output/ch{NN}/ch{NN}_script.json`

CLAUDE.md에 정의된 마스터 대본 스키마를 따른다:

```json
{
  "version": "1.0",
  "chapter": 3,
  "title": "기계는 어떻게 가르치는 존재가 되었나",
  "genre": "classic-documentary",
  "total_duration_seconds": 120,
  "scenes": [ ... ],
  "video_meta": {
    "aspect_ratio": "16:9",
    "opening_title": "3장. 기계가 가르치기 시작하다",
    "closing_text": "다음 장에서 계속...",
    "default_transition": "crossfade",
    "bgm_track": null
  }
}
```

**video_meta 규칙:**
- `aspect_ratio`: 시리즈 가이드 또는 장르 템플릿 참조
- `opening_title`: "N장. [챕터 제목]"
- `closing_text`: 마지막 장이면 "시리즈 완결", 아니면 "다음 장에서 계속..."
- `default_transition`: 시리즈 가이드 또는 장르 템플릿 참조
- `total_duration_seconds`: 모든 씬의 narration_seconds 합계

### Step 3: MD 생성 — 사람이 읽는 대본

`output/ch{NN}/ch{NN}_script.md`

```markdown
# N장. [챕터 제목]

> 장르: [장르 이름] | 씬: [N]개 | 총 시간: [M]분 [S]초

---

## Scene 1: [씬 제목] (N초)

**내레이션:**
> [내레이션 텍스트]

**이미지 프롬프트:**
[영문 프롬프트]

**비주얼 메모:** [visual_description]

**파일명:** [image_filename]
**모델:** [model]
**자막:** [subtitle]
**분위기:** [mood] | **전환:** [transition_hint] | **BGM:** [bgm_hint]

---

## Scene 2: [씬 제목] (N초)
...

---

## 영상 정보

- 화면비: [aspect_ratio]
- 오프닝: [opening_title]
- 엔딩: [closing_text]
- 기본 전환: [default_transition]
```

### Step 4: CSV 생성 — Excel 검토용

`output/ch{NN}/ch{NN}_script.csv`

UTF-8 BOM 인코딩으로 생성하여 Excel에서 바로 열 수 있게 한다.

**컬럼:**

| 컬럼 | 내용 |
|------|------|
| scene | 씬 번호 |
| title | 씬 제목 (한국어) |
| narration_text | 내레이션 텍스트 |
| narration_seconds | 예상 시간 |
| prompt | 영문 이미지 프롬프트 |
| model | 모델 (nano_banana/imagen) |
| image_filename | 이미지 파일명 |
| visual_description | 비주얼 메모 |
| era | 시대 |
| mood | 분위기 |
| transition | 전환 효과 |
| subtitle | 자막 |
| bgm_hint | BGM 힌트 |

### Step 5: 매니페스트 업데이트

`output/_manifest.json`을 업데이트 (또는 신규 생성):

```json
{
  "book_title": "시리즈 제목 (시리즈 가이드 참조)",
  "genre": "classic-documentary",
  "chapters": [
    {
      "chapter": 3,
      "title": "기계가 가르치기 시작하다",
      "scene_count": 5,
      "total_seconds": 120,
      "generated_at": "2026-04-15T12:00:00+09:00",
      "files": {
        "json": "ch03/ch03_script.json",
        "md": "ch03/ch03_script.md",
        "csv": "ch03/ch03_script.csv"
      }
    }
  ]
}
```

기존 매니페스트가 있으면 해당 챕터 항목만 추가/업데이트.

### Step 6: 결과 보고

```
✅ 대본 생성 완료

📁 output/ch03/
   ├── ch03_script.json  (FlowGenie/TTS/SceneWeaver용)
   ├── ch03_script.md    (사람이 읽는 대본)
   └── ch03_script.csv   (Excel 검토용)

📊 요약:
   씬: 5개 | 총 시간: 2분 0초
   모델: nano_banana ×5
   이미지 파일: ch03_01~ ch03_05

💡 다음 단계:
   - JSON을 FlowGenie에 드래그앤드롭하면 이미지 생성 시작
   - /forge-review로 수정 가능
   - /forge-batch로 나머지 챕터 일괄 처리 가능
```
