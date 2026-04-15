---
name: prompt-craft
description: 확정된 씬 라인업의 각 씬에 대해 영문 이미지 프롬프트, 한국어 내레이션 텍스트, scene_meta를 작성한다. 시리즈 가이드의 장르 프로필과 knowledge/prompt-style-guide.md를 참조하여 일관된 스타일을 유지한다. 씬 라인업이 확정된 후 자동으로 실행되거나, '프롬프트 만들어줘'라고 요청하면 사용한다.
---

# 씬별 상세 설계

확정된 씬 라인업의 각 씬에 대해 이미지 프롬프트, 내레이션, 메타데이터를 작성한다.

## 사전 조건

- analyze-chapter에서 씬 라인업이 확정되어야 한다
- 시리즈 가이드 또는 장르 지정이 있어야 한다

## 참조 문서

- `knowledge/prompt-style-guide.md` — 프롬프트 작성 원칙
- 장르 템플릿 JSON — image_style, narration_style, scene_structure
- 시리즈 가이드 — visual_consistency (시대별 톤 진행, 오프닝/엔딩)

## 작성 절차 (씬당)

### 1. 원문 시각 요소 추출

해당 씬의 출처 섹션에서:
- 묘사된 물리적 환경 (장소, 물건, 조명)
- 인물의 외모/행동/표정
- 시대적 단서 (연도, 기술, 의복)
- 분위기/감정

### 2. 영문 이미지 프롬프트 작성

prompt-style-guide.md의 5단계 구조를 따른다:

```
[주요 피사체/배경] + [시대적 디테일] + [구도] + [색조/톤] + [스타일 태그]
```

**규칙:**
- 30-80단어
- 시리즈 가이드의 color_progression에 맞는 톤 적용
- 장르 템플릿의 base_suffix로 마무리
- era_overrides 참조하여 시대 톤 적용
- 텍스트 렌더링 의존 금지 (자막은 scene_meta.subtitle에)
- 실존 인물 이름 대신 외모/직업으로 묘사

### 3. 한국어 visual_description 작성

프롬프트의 핵심 시각 요소를 한국어 1줄로 요약.
- 빠른 참조용 메모
- 예: "1950년대 교실, 칠판, 학생들, 흑백 톤"

### 4. 모델 선택

prompt-style-guide.md의 모델 선택 가이드 참조:
- 시리즈 가이드의 default_model이 기본
- 특정 씬에서 다른 모델이 더 적합하면 override 가능

### 5. 내레이션 텍스트 작성

한국어로 2-4문장:
- 해당 씬의 핵심 내용을 시청자에게 전달
- 장르 템플릿의 narration_style (tone, person, tempo) 준수
- 원문의 인상적인 표현을 활용하되, 영상 내레이션에 맞게 재구성
- 너무 학술적이지 않게, 듣기 편한 문장으로

### 6. 내레이션 시간 추정

```
narration_seconds = ceil(narration_text 글자수 / chars_per_second)
```
- chars_per_second: 시리즈 가이드 또는 장르 템플릿 참조 (기본 3.5)
- 범위: 15-30초/씬

### 7. scene_meta 작성

```json
{
  "era": "1950s",           // 10년 단위 시대
  "mood": "tension",        // tension, discovery, sadness, hope, neutral 등
  "transition_hint": "fade_in",  // fade_in, crossfade, cut, wipe, glitch 등
  "text_overlay": null,     // 필요 시 자막 텍스트
  "subtitle": "1953년 11월, 하버드 대학교",  // 하단 자막
  "bgm_hint": "suspenseful_piano"  // BGM 분위기 힌트
}
```

**transition_hint 규칙:**
- 첫 씬: `fade_in`
- 같은 시대 연속: `crossfade`
- 시대 전환: `wipe` 또는 장르 템플릿의 default_transition
- 극적 전환: `fade_to_black`
- 마지막 씬: `fade_out`

### 8. 파일명 생성

`ch{NN}_{SS}_{영문슬러그}.png`
- NN: 장 번호 (2자리, 0패딩)
- SS: 씬 번호 (2자리, 0패딩)
- 영문슬러그: 씬 내용을 영문 소문자+언더스코어로 (3-5단어)
- 예: `ch03_01_1953_classroom.png`

## 출력 형식

2-3씬씩 묶어서 사용자에게 보여준다:

```
━━━ Scene 1: [씬 제목] ━━━━━━━━━━━━━━━━

📸 이미지 프롬프트:
"A 1950s elementary school classroom scene..."

🎙️ 내레이션 (25초):
"1953년 하버드, 심리학 교수 스키너는..."

📋 메타:
모델: nano_banana | 파일: ch03_01_1953_classroom.png
시대: 1950s | 분위기: tension | 전환: fade_in
자막: 1953년 11월, 하버드 대학교

━━━ Scene 2: [씬 제목] ━━━━━━━━━━━━━━━━
...
```

사용자 확인 후 다음 2-3씬 진행. 전체 완료되면 generate-script로 넘어간다.

## 사용자 수정 대응

- "프롬프트 수정: [새 내용]" → 해당 씬 프롬프트 재작성
- "내레이션 톤 바꿔줘" → narration_style 조정
- "이 씬 모델 바꿔" → model override
- "자막 추가/변경" → scene_meta.subtitle 수정
