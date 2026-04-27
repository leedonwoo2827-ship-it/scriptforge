# 이미지 프롬프트 작성 가이드

이 문서는 FlowGenie(Google Flow/Imagen)용 영문 이미지 프롬프트 작성의 원칙과 패턴을 정의한다.

## 기본 원칙

1. **항상 영어로 작성** — Google Flow는 영어 프롬프트에 최적화
2. **30-80단어** — 너무 짧으면 모호, 너무 길면 핵심 희석
3. **구체적 시각 요소 중심** — 추상 개념이 아니라 "보이는 것"을 묘사
4. **스타일 태그로 마무리** — 장르 템플릿의 `image_style.base_suffix` 사용

## 프롬프트 구조 (5단계)

```
[주요 피사체/배경] + [시대적 디테일] + [구도/프레이밍] + [색조/톤] + [스타일 태그]
```

### 예시 분해

```
A 1950s elementary school classroom scene.     ← 주요 피사체/배경
Chalkboard with 20 math problems written.      ← 시대적 디테일
30 students sitting at desks.                  ← 디테일 추가
Teacher walking around the classroom.          ← 인물 행동
Black and white photography, nostalgic         ← 색조/톤
1950s atmosphere.                              ← 시대 분위기
Documentary style.                             ← 스타일 태그
```

## 시대별 시각 어휘

### pre-1900 (1840-1899)
- 톤: sepia tone, daguerreotype, hand-tinted photograph
- 배경: Victorian era, gas lamp lighting, wooden furniture
- 의복: period clothing, formal attire, top hats
- 기술: handwritten letters, printing press, telegraph

### 1900-1950
- 톤: black and white photography, vintage, silver gelatin print
- 배경: industrial era, Art Deco architecture, university halls
- 의복: 1920s-1940s fashion, academic robes
- 기술: typewriter, radio equipment, early mechanical devices

### 1950-1980
- 톤: mid-century photography, slightly desaturated, Kodachrome feel
- 배경: modernist architecture, university labs, TV studios
- 의복: 1950s professional attire, lab coats
- 기술: mainframe computers, television sets, teaching machines, tape reels

### 1980-2000
- 톤: warm film photography, slightly grainy, retro
- 배경: computer labs, early internet cafes, offices with CRT monitors
- 의복: 1980s-1990s casual, Silicon Valley style
- 기술: personal computers, early internet, CD-ROMs, dial-up modems

### 2000-현재
- 톤: modern digital photography, clean, high contrast
- 배경: modern classrooms, smartphones, tablets, sleek offices
- 의복: contemporary casual, hoodies, business casual
- 기술: smartphones, tablets, VR headsets, AI interfaces, neural networks

## 구도 유형

| 구도 | 용도 | 프롬프트 키워드 |
|------|------|----------------|
| 와이드 샷 | 배경/시대 설정 | wide shot, establishing shot, panoramic view |
| 인물 초상 | 인물 소개/감정 | portrait, close-up, head and shoulders |
| 극단적 클로즈업 | 물건/디테일 강조 | extreme close-up, macro detail, focus on |
| 분할 화면 | 비교/대비 | split-screen composition, left/right comparison |
| 다이어그램 | 기술/메커니즘 설명 | technical diagram, schematic, blueprint style |
| 오버헤드 | 작업 공간/책상 | overhead view, bird's eye, flat lay |
| 몽타주 | 시간 경과/변화 | montage composition, collage, multiple frames |

## 모델 선택 가이드

| 장면 유형 | 추천 모델 | 이유 |
|-----------|-----------|------|
| 역사적 장면 (pre-1980) | nano_banana | 예술적/회화적 표현 강점 |
| 현대적 장면 (post-2000) | imagen | 사실적/선명 표현 강점 |
| 다이어그램/도표 | nano_banana | 일러스트 스타일 적합 |
| 인물 초상 | nano_banana | 초상화 느낌 연출 |
| 기술 장비 | imagen | 디테일 정확도 |

시리즈 가이드의 `default_model`이 있으면 기본으로 따르되, 특정 씬에서 다른 모델이 더 적합하면 override 가능.

## 프롬프트 안티패턴

### 피해야 할 것

1. **텍스트 렌더링 의존 금지**
   - 나쁜: "Image with text saying 'B.F. Skinner, 1953'"
   - 좋은: "Portrait of a 1950s psychology professor" + subtitle은 scene_meta에

2. **너무 많은 피사체 금지**
   - 나쁜: "A classroom with 30 students, teacher, chalkboard, desks, books, pencils, windows, clock, globe"
   - 좋은: "A 1950s classroom scene. Teacher at chalkboard, students at desks"

3. **추상적 개념 금지**
   - 나쁜: "The concept of behavioral reinforcement in education"
   - 좋은: "A laboratory rat pressing a lever in a Skinner Box, food pellet dispensing"

4. **부정형 지시 금지**
   - 나쁜: "Not a modern classroom, no computers"
   - 좋은: "A 1950s classroom, wooden desks, chalkboard"

5. **저작권 인물 이름 주의**
   - 나쁜: "Photo of B.F. Skinner" (실존 인물 사진 직접 생성)
   - 좋은: "A 1950s psychology professor, serious expression, glasses, tweed jacket"

## visual_description 작성

- 한국어로 작성
- prompt의 핵심 시각 요소를 1줄로 요약
- 사용자와 SceneWeaver가 빠르게 참조할 수 있는 메모 역할
- 예: "1950년대 교실 사진, 칠판에 수학 문제, 30명의 학생, 흑백"

## 유명인 정책 차단 — 약화 단계별 가이드

Google Flow / Imagen / 기타 이미지 생성 모델은 실존 인물을 식별 가능한 단서 조합을 차단한다. 단순히 인물 이름만 빼는 것으로는 부족할 때가 많고, **국적·직업·시그니처 용어·시그니처 발명품** 4가지 단서 중 2개 이상이 결합되면 차단된다.

### 차단 단서 4축

| 축 | 예시 | 위험도 |
|---|---|:-:|
| **신원 단서** | South African, Irish, African-American mathematician | 높음 |
| **직업 단서** | mathematician, physicist, behavioral psychologist | 중간 |
| **시그니처 용어** | Constructionism, World Wide Web, Operant Conditioning, Walden Two | 매우 높음 |
| **시그니처 발명품/장소** | turtle robot, NeXT computer at CERN, Skinner Box, Macintosh prototype | 매우 높음 |

이 4축 중 **3개 이상이 한 프롬프트에 동시 등장하면 거의 확실히 차단**된다.

### 약화 단계 (위에서 아래로 점진적)

차단되면 다음 단계로 내려가며 재시도한다.

| 단계 | 변경 | 예시 |
|:-:|---|---|
| **L0** | 인물 이름 직접 사용 | `Photo of Seymour Papert` (즉시 차단) |
| **L1** | 인물 이름만 제거 | `South African mathematician` |
| **L2** | 국적·인종 단서 제거 | `mathematician` |
| **L3** | 시그니처 용어를 일반 어구로 | `Constructionism` → `Learning by Making` |
| **L4** | 직업 단서 일반화 | `mathematician` → `thoughtful academic` |
| **L5** | 기관명 일반화 | `MIT AI Lab` → `1960s university research office` |
| **L6** | 발명품 명칭 우회 | `turtle robot` → `two-wheeled educational floor robot with a pen attachment` |
| **L7** | 시대 명시 약화 | `1963` → `1960s` |

> **경험칙:** 첫 시도는 L1부터 시작. 차단되면 한 번에 L3-L7까지 동시 적용해서 재시도하는 게 효율적. 한 단계씩 내리면 시간 낭비.

### 사례 1: 시모어 페퍼트 (3장 18씬, 1963 MIT)

**의도:** 페퍼트가 MIT AI 연구실에서 거북이 로봇으로 LOGO를 구상하는 part_intro 씬.

| 시도 | 프롬프트 핵심 | 결과 |
|:-:|---|:-:|
| 1차 | `South African mathematician` + `Piaget developmental psychology books` + `MIT AI Lab` + `Constructionism` + `turtle robot` | ❌ 차단 |
| 2차 | `thoughtful researcher` + `developmental psychology textbooks` + `MIT AI Lab` + `Constructionism` + `turtle robot` (L1-L2) | ❌ 차단 |
| 3차 | `thoughtful academic` + `educational research books` + `1960s university research office` + `Learning by Making` + `two-wheeled educational floor robot with pen attachment` (L3-L7 동시) | ✅ 통과 |

**교훈:** 시그니처 용어(Constructionism)와 시그니처 발명품(turtle robot)을 동시에 약화하지 않으면 통과하지 않는다. 둘 중 하나만 풀면 다른 하나가 트리거.

### 의미 손실 방지 원칙

이미지가 약해져도 영상 후처리 레이어에서 정확한 정보를 보충한다. 이미지는 **분위기**만 담당하고, 사실은 자막·내레이션·overlay가 담당하도록 역할 분담하면 시청자 경험상 손실 없다.

| 정보 | 담당 레이어 |
|---|---|
| 인물 이름·정확 연도 | `scene_meta.text_overlay` |
| 학자 인용·사상 설명 | `narration_text` (TTS 음성) |
| 기관명·지명·부제 | `scene_meta.subtitle` (자막) |
| 분위기·시대감·구도 | 이미지 프롬프트 |

### 인물별 우회 패턴 사전 (시리즈 작업 중 누적)

| 인물 | 차단 단서 | 통과 패턴 |
|---|---|---|
| 시모어 페퍼트 | South African + MIT AI Lab + Constructionism + turtle robot | thoughtful academic + 1960s university research office + Learning by Making + two-wheeled educational floor robot |
| _(향후 추가)_ | _버너스 리, 잡스, 게이츠, 머스크, 올트먼 등은 작업 시 누적_ | |

### 모델별 차이

| 모델 | 정책 강도 | 비고 |
|:-:|:-:|---|
| Google Flow / Imagen | 강함 | 위 4축 동시 등장 시 차단 빈번 |
| nano_banana | 중간 | 인물 묘사 약함 → 자연스러운 우회 가능 |
| 기타 | 변동 | 모델 변경도 우회 옵션 중 하나 |

차단이 풀리지 않으면 **모델을 바꾸는 것도 우회 옵션**이다. 시리즈 가이드의 `default_model`에 묶이지 말고 씬 단위로 override 검토.
