---
name: shucle-decision-history
description: GitHub 이슈와 MD 파일을 기반으로 특정 결정에 대한 히스토리를 찾아주는 스킬. 결과와 함께 관련 이슈 링크 및 MD 링크를 포함한다.
---

# Shucle Decision History

특정 기능/디자인/정책에 대한 **결정 히스토리**를 GitHub 이슈와 기획 MD에서 추적해 요약한다.
아래 절차를 순서대로 따른다.

## 변경 이력

| 날짜 | 변경 내용 |
|------|-----------|
| 2026-04-22 | 최초 생성 |

---

## Step 1. 입력 수집

아래를 **한 번에** 모아서 물어본다:

**레포** (번호 또는 복수 입력 가능, 미입력 시 전체 검색)

- 1 택시앱(`shucle-taxidriver-product`)
- 2 드라이버앱(`shucle-DriverVehicle-product`)
- 3 라이더앱(`shucle-rider`)
- 4 키오스크(`shucle-kiosk-product`)
- 전체 (레포 미선택 또는 "모두")

**질문** — 어떤 결정의 히스토리를 알고 싶은지 자유 기술

> 예: "결제 화면에서 취소 버튼이 왼쪽에 있는 이유", "택시 콜 수락 UX를 스와이프로 바꾼 이유", "온보딩 단계를 줄인 배경"

---

## Step 2. 키워드 추출

질문에서 **2자 이상 핵심 키워드**를 추출한다.

- 화면명, 기능명, 컴포넌트명, 정책명 우선
- 예: "결제 화면 취소 버튼" → `결제`, `취소`, `버튼`

---

## Step 3. 데이터 조회

아래 순서로 병렬 조회한다.

```
📂 기획 MD 검색 중...
📋 GitHub 이슈 검색 중...
```

### 3-1. MD 파일 조회

`mcp__github__get_file_contents` 로 선택한 레포의 기획서 폴더 목록 조회 → `.md` 파일 경로 수집

레포별 기획서 폴더:

| 레포 | 폴더 |
|------|------|
| `shucle-taxidriver-product` | `Voucher Taxi Driver App/` |
| `shucle-DriverVehicle-product` | `Driver App/`, `Vehicle App/` |
| `shucle-rider` | `01_라이더앱/` |
| `shucle-kiosk-product` | `01_키오스크웹앱/` |

**읽기 전략**

1. 파일명과 Step 2 키워드를 매칭해 관련성 높은 파일 우선 읽기
2. 읽은 MD 내에서 키워드가 등장하는 섹션 특정 (헤더 경로 기록)
3. 관련 섹션 앵커 링크 수집: `https://github.com/hkmc-airlab/{repo}/blob/master/{path}#{anchor}`

### 3-2. GitHub 이슈 조회

`mcp__github__list_issues` 로 선택한 레포 이슈 100개 조회 (state: all).

- 제목에 키워드 포함된 이슈 필터링
- 관련 이슈는 `mcp__github__get_issue` 로 본문 + 댓글 상세 조회
  - 댓글 URL: `https://api.github.com/repos/hkmc-airlab/{repo}/issues/{number}/comments`
  - 본문·댓글에서 결정 배경·논의 내용 추출
- 최대 10개 이슈까지 상세 조회

전체 레포 검색 시에는 4개 레포를 병렬로 조회한다.

---

## Step 4. 결과 분석

수집된 이슈 + MD 내용을 종합해 아래 질문에 답한다:

- **무엇을** 결정했는가 (결론)
- **왜** 그렇게 결정했는가 (근거·배경)
- **언제** 결정됐는가 (이슈 날짜, MD 변경 기록)
- **누가** 논의에 참여했는가 (이슈 참여자)

단서가 부족하면 솔직하게 "이슈 및 MD에서 명확한 결정 근거를 찾지 못했습니다"라고 명시한다.

---

## Step 5. 결과 출력

아래 형식으로 출력한다.

```markdown
## 결정 히스토리: {질문 요약}

### 요약
{결정 내용을 2-4문장으로 요약}

### 결정 배경
{근거·이유를 불렛으로 정리}

### 타임라인
| 날짜 | 내용 | 출처 |
|------|------|------|
| YYYY-MM-DD | ... | [#{이슈번호} 제목](URL) 또는 [MD명 > 섹션](URL) |

### 관련 이슈
- [#{번호} {제목}]({GitHub 이슈 URL}) — {한줄 요약}

### 관련 MD
- [{파일명} > {섹션 경로}]({앵커 URL}) — {한줄 요약}

---
⚠️ 단서 없음 섹션: 이슈·MD에서 근거를 찾지 못한 경우 명시
```

**출력 규칙**

- 관련 이슈·MD가 없으면 해당 섹션 생략하지 말고 "관련 이슈를 찾지 못했습니다"로 명시
- 이슈 URL 형식: `https://github.com/hkmc-airlab/{repo}/issues/{number}`
- MD 앵커 URL 형식: `https://github.com/hkmc-airlab/{repo}/blob/master/{path}#{anchor}`
- 타임라인은 날짜 오름차순 정렬
- 추측성 내용은 "추정:" 접두어를 붙여 명확히 구분

---

## 레포 목록 (참고)

| 번호 | 레포명 | GitHub |
|------|--------|--------|
| 1 | shucle-taxidriver-product | https://github.com/hkmc-airlab/shucle-taxidriver-product |
| 2 | shucle-DriverVehicle-product | https://github.com/hkmc-airlab/shucle-DriverVehicle-product |
| 3 | shucle-rider | https://github.com/hkmc-airlab/shucle-rider |
| 4 | shucle-kiosk-product | https://github.com/hkmc-airlab/shucle-kiosk-product |
