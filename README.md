# ScriptForge

확정된 영상 스타일과 챕터 마크다운을 입력받아, 하류 프로세스(이미지/음성/영상)가 모두 소비할 수 있는 **마스터 대본 JSON**을 생성하는 Claude Code 스킬 세트.

## 파이프라인 위치

```
StoryLens-27 → [ScriptForge] → FlowGenie + TTS → SceneWeaver
(상담/기획)    (대본 생성)      (이미지+음성)     (CapCut 조립)
```

- **상류**: [StoryLens-27](https://github.com/leedonwoo2827-ship-it/storylens-27)이 생성한 `_series_guide.json` 또는 사용자 직접 장르 지정
- **하류**:
  - [FlowGenie](https://github.com/leedonwoo2827-ship-it/auto-flowgenie): `scenes[].prompt/model/image_filename` 소비
  - TTS: `scenes[].narration_text/voice_style` 소비
  - SceneWeaver (CapCut 자동화): `scenes[].scene_meta` + `video_meta` 소비

## 마스터 대본 JSON 스키마

```jsonc
{
  "version": "1.0",
  "chapter": 3,
  "title": "기계는 어떻게 가르치는 존재가 되었나",
  "genre": "classic-documentary-full",
  "total_duration_seconds": 900,

  "scenes": [
    {
      "scene": 1,
      "title": "1953년 하버드 교실",

      // FlowGenie 호환
      "image_filename": "ch03_01_1953_classroom.png",
      "prompt": "A 1950s elementary school classroom scene...",
      "model": "nano_banana",
      "visual_description": "1950년대 교실, 칠판, 학생들, 흑백",
      "reference_image": null,

      // TTS용
      "narration_text": "1953년 하버드, 심리학 교수 스키너는...",
      "narration_seconds": 25,
      "voice_style": "calm_narrator",

      // SceneWeaver용
      "scene_meta": {
        "era": "1950s",
        "mood": "tension",
        "transition_hint": "fade_in",
        "subtitle": "1953년 11월, 하버드 대학교",
        "bgm_hint": "suspenseful_piano"
      }
    }
  ],

  "video_meta": {
    "aspect_ratio": "16:9",
    "opening_title": "3장. 기계가 가르치기 시작하다",
    "closing_text": "다음 장에서 계속...",
    "default_transition": "crossfade"
  }
}
```

## 사용법

### 메인 — 챕터별 대본 생성
```
/forge
```
챕터 분석 → 씬 라인업 → 프롬프트 작성 → JSON/MD/CSV 저장. 단계별 사용자 확인.

### 일괄 처리
```
/forge-batch
```
여러 챕터를 시리즈 가이드 기반으로 일괄 생성.

### 기존 대본 수정
```
/forge-review
```
생성된 대본의 씬 추가/삭제/수정.

## 출력 형식 (3종)

| 형식 | 용도 | 위치 |
|------|------|------|
| JSON | FlowGenie + TTS + SceneWeaver 입력 | `output/ch{NN}/ch{NN}_script.json` |
| MD | 사람이 읽는 대본 | `output/ch{NN}/ch{NN}_script.md` |
| CSV | Excel 검토용 (UTF-8 BOM) | `output/ch{NN}/ch{NN}_script.csv` |

## 프로젝트 구조

```
scriptforge/
├── .claude-plugin/plugin.json
├── CLAUDE.md
├── skills/
│   ├── analyze-chapter/         # 챕터 분석 + 씬 선별
│   ├── prompt-craft/            # 영문 이미지 프롬프트 작성
│   └── generate-script/         # JSON/MD/CSV 출력 생성
├── commands/
│   ├── forge.md                 # 메인 오케스트레이션
│   ├── forge-batch.md           # 일괄 처리
│   └── forge-review.md          # 기존 대본 수정
└── knowledge/
    ├── scene-design-guide.md    # 씬 선별 6대 기준
    └── prompt-style-guide.md    # 이미지 프롬프트 작성 가이드
```

## 라이센스

MIT
