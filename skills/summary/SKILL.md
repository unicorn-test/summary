---
name: summary
description: >
  This skill should be used when the user asks to "한화이글스 요약", "한화 뉴스 요약해줘",
  "이글스 오늘 경기 요약", "이글스 요약", "한화이글스 레포트 만들어줘", "eagles report",
  "한화이글스 뉴스 레포트 생성", "한화 팬 반응 정리해줘", or any request to summarize
  Hanhwa Eagles game results and fan reactions into a report for the team manager.
version: 0.2.0
user-invocable: false
---

# 한화이글스 뉴스 요약 스킬 (서브에이전트 분업 구조)

## 목적

2026 KBO 시즌 진행 중인 한화이글스의 경기 실적과 팬 반응을 자동으로 수집·요약하여,
야구 감독님이 경기 운영 자료로 활용할 수 있는 레포트를 생성함.
오케스트레이터가 전문 서브에이전트에게 단계별로 위임하는 분업 구조로 동작함.

---

## 오케스트레이션 원칙

- **메인(클로니)은 직접 작업하지 않음.** 서브에이전트 호출·결과 통합·사용자 보고만 수행함
- 각 단계는 Agent 도구로 전용 서브에이전트에게 위임함
- 실행 증거(파일 생성·응답 데이터) 없이 완료 보고 금지

---

## 서브에이전트 구성

| 단계 | 서브에이전트 | 팀원 | 호출 방식 | 모델 | 핵심 툴 |
|------|-------------|------|----------|------|---------|
| 1. 수집 | `eagles-researcher` | 애나 | **병렬** (국내/해외) | sonnet | WebSearch, Bash(curl) |
| 2. 초안 | `eagles-summarizer` | 수미 | 순차 (초안 모드) | sonnet | Read, Write, Bash |
| 3. 검수 | `eagles-reviewer` | 퀴니 | 순차 (취합 후 단일) | opus | Read (쓰기 금지) |
| 4. 최종 | `eagles-summarizer` | 수미 | 순차 (최종본 모드) | sonnet | Read, Write, Bash |

---

## 워크플로우

### Step 1 — 뉴스 수집 (병렬 fan-out)

`eagles-researcher`를 **2개 인스턴스로 병렬 호출**한다 (한 메시지에 Agent 2개):

- 인스턴스 A: 범위="국내", 주제=사용자 요청(예: "어제 경기") → 국내 뉴스 최대 3건
- 인스턴스 B: 범위="해외", 주제=사용자 요청 → 해외 뉴스 최대 2건

각 인스턴스는 WebSearch + curl + 텍스트 추출 후 구조화 기사 데이터를 반환한다.
두 결과를 합쳐 기사 5건 데이터셋을 구성한다.

### Step 2 — 요약 초안 작성 (순차)

`eagles-summarizer`를 **초안 모드**로 호출한다.
- 전달: Step 1의 기사 5건 데이터셋
- 반환: 요약 초안 전문 (파일 미생성)

### Step 3 — 품질 검수 (순차, 취합 후 단일 실행)

`eagles-reviewer`를 호출한다.
- 전달: Step 2 초안 전문 + Step 1 원본 기사 데이터
- 반환: 심각도별 수정 요청 목록 (치명/주요/경미) 또는 "통과"

### Step 4 — 최종 레포트 작성 (순차)

`eagles-summarizer`를 **최종본 모드**로 호출한다.
- 전달: Step 2 초안 + Step 3 검수 피드백
- 동작: 피드백 반영 후 `reports/eagles_{YYYYMMDD}_{HHMM}.md` 파일 생성
- 반환: 생성된 파일 경로

검수에서 "치명" 지적이 남으면 Step 2~3을 1회 재수행한다 (최대 3회).

### Step 5 — 사용자 보고 (메인 직접 수행)

메인(클로니)이 생성된 파일 경로와 핵심 요약을 사용자에게 보고한다.

---

## 완료 기준

- [ ] `eagles-researcher` 병렬 호출로 기사 5건(해외 2+국내 3) 데이터 확보
- [ ] `eagles-summarizer` 초안 → `eagles-reviewer` 검수 → `eagles-summarizer` 최종본 순서 수행
- [ ] `reports/eagles_{YYYYMMDD}_{HHMM}.md` 파일이 실제로 생성됨
- [ ] Fact/Opinion 분리, 감독님 관점 인사이트 포함
- [ ] 메인이 파일 경로를 사용자에게 보고

---

## 주의사항

- 나무위키 등 JavaScript SPA는 curl로 본문 추출 불가 → WebSearch 결과로 대체 (researcher가 처리)
- 수치 미확인 항목은 반드시 "(※ 원문 미확인)" 표기
- 보고서 상단에 "기준일: {Get-Date 결과}"로 현재 날짜 명시하고 시즌 진행 중임을 표기
- AGENTS.md 정직한 보고 규칙 준수: 파일 생성 전까지 완료 보고 금지
- 서브에이전트 호출 실패 시 메인이 사용자에게 상황을 보고하고 재시도 여부를 확인

---

## Additional Resources

### Reference Files

- **`skills/summary/references/report-template.md`** — 최종 레포트 전체 템플릿 (summarizer가 참조)

### 서브에이전트 파일

- **`agents/eagles-researcher.md`** — 뉴스 수집·추출 (애나)
- **`agents/eagles-summarizer.md`** — 요약 초안·최종본 작성 (수미)
- **`agents/eagles-reviewer.md`** — 품질 검수 (퀴니)
