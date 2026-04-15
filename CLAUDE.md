# ScriptForge — 영상 대본 생성 스킬

## 프로젝트 개요

확정된 영상 스타일(장르)과 챕터 마크다운을 입력받아, 하류 프로세스(이미지 생성, 음성 생성, 영상 조립)가 모두 소비할 수 있는 **마스터 대본 JSON**을 생성하는 스킬 세트.

## 파이프라인 위치

```
StoryLens → [ScriptForge] → FlowGenie + TTS → SceneWeaver
(상담/기획)   (대본 생성)     (이미지+음성)     (CapCut 조립)
```

- **상류**: StoryLens(260418)의 `_series_guide.json` 또는 사용자 직접 장르 지정
- **하류**:
  - FlowGenie(260415): `scenes[].prompt/model/image_filename` 소비
  - TTS: `scenes[].narration_text/voice_style` 소비
  - SceneWeaver(260417): `scenes[].scene_meta` + `video_meta` 소비

## 핵심 규칙

1. **원고 경로**: `D:\00work\260416-scriptforge\sample\` 의 챕터 MD 파일
2. **시리즈 가이드**: `output\_series_guide.json`이 있으면 자동 로드하여 스타일 일관성 유지
3. **언어 구분**:
   - 이미지 프롬프트(`prompt`): 항상 **영어**
   - 내레이션/제목/설명: 항상 **한국어**
4. **FlowGenie 하위 호환**: 기존 FlowGenie가 읽을 수 없는 필드가 추가되더라도, 기존 필드(prompt, model, image_filename 등)는 변경하지 않는다
5. **단계별 확인**: `/forge` 실행 시 각 단계마다 사용자 확인을 받는다. 한 번에 전체를 생성하지 않는다
6. **본문 범위**: 챕터 MD에서 `## 참고문헌`과 `# 인물부록` 이전까지가 씬 추출 대상 본문

## 마스터 대본 JSON 스키마

```jsonc
{
  "version": "1.0",
  "chapter": 3,                          // 장 번호 (정수)
  "title": "기계는 어떻게 가르치는 존재가 되었나",  // 한국어 제목
  "genre": "classic-documentary",         // genre_id
  "total_duration_seconds": 120,          // 전체 예상 시간

  "scenes": [
    {
      "scene": 1,                         // 씬 번호 (1-based)
      "title": "1953년 하버드 교실",       // 한국어 씬 제목

      // ── FlowGenie 호환 ─────────────
      "image_filename": "ch03_01_1953_classroom.png",
      "prompt": "A 1950s elementary school classroom scene...",
      "model": "nano_banana",             // nano_banana | imagen
      "visual_description": "1950년대 교실, 칠판, 학생들, 흑백",
      "reference_image": null,

      // ── TTS용 ──────────────────────
      "narration_text": "1953년 하버드, 심리학 교수 스키너는...",
      "narration_seconds": 25,
      "voice_style": "calm_narrator",

      // ── SceneWeaver용 ──────────────
      "scene_meta": {
        "era": "1950s",
        "mood": "tension",
        "transition_hint": "fade_in",
        "text_overlay": null,
        "subtitle": "1953년 11월, 하버드 대학교",
        "bgm_hint": "suspenseful_piano"
      }
    }
  ],

  "video_meta": {
    "aspect_ratio": "16:9",
    "opening_title": "3장. 기계가 가르치기 시작하다",
    "closing_text": "다음 장에서 계속...",
    "default_transition": "crossfade",
    "bgm_track": null
  }
}
```

## 파일명 규칙

- 이미지: `ch{NN}_{SS}_{영문슬러그}.png` (예: ch03_01_1953_classroom.png)
- JSON: `output/ch{NN}/ch{NN}_script.json`
- MD: `output/ch{NN}/ch{NN}_script.md`
- CSV: `output/ch{NN}/ch{NN}_script.csv`

## 내레이션 시간 추정

- 한국어 내레이션 속도: 시리즈 가이드의 `chars_per_second` 참조 (기본 3.5자/초)
- `narration_seconds = ceil(narration_text 글자수 / chars_per_second)`
- 씬당 목표 범위: 15-30초
- 에피소드 전체 목표: 시리즈 가이드의 `episode_length_seconds` 참조
