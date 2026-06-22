# summary 플러그인

한화이글스 뉴스를 자동 수집·요약·검수하여 감독님용 레포트를 생성하는 Claude Code 플러그인.

## 기능

| 컴포넌트 | 설명 |
|---------|------|
| `/summary:summary` | 한화이글스 뉴스 요약 레포트 생성 슬래시 명령 |
| `eagles-researcher` | 국내·해외 뉴스 수집·추출 서브에이전트 (애나) |
| `eagles-summarizer` | 요약 초안·최종본 작성 서브에이전트 (수미) |
| `eagles-reviewer` | 요약 정확성·완전성 검수 서브에이전트 (퀴니) |

### 워크플로우

```
사용자 요청
  → [1] eagles-researcher (병렬: 국내 3건 + 해외 2건 수집)
  → [2] eagles-summarizer (초안 작성)
  → [3] eagles-reviewer   (정확성·완전성 검수)
  → [4] eagles-summarizer (최종본 작성 → reports/ 저장)
  → 사용자에게 파일 경로 보고
```

---

## 플러그인 설치

> `claude plugins install`은 마켓플레이스 등록 후 설치하는 2단계 방식임.

### 로컬 설치

```bash
# 1단계: 마켓플레이스 등록
claude plugins marketplace add /path/to/summary

# 2단계: 플러그인 설치
claude plugins install summary
```

### Git 저장소에서 설치

```bash
# 1단계: GitHub 저장소를 마켓플레이스로 등록
claude plugins marketplace add https://github.com/unicorn-test/summary

# 2단계: 플러그인 설치
claude plugins install summary
```

---

## 플러그인 조회

설치된 플러그인 목록 확인:

```bash
claude plugins list
```

특정 플러그인 상세 조회:

```bash
claude plugins details summary
```

등록된 마켓플레이스 목록:

```bash
claude plugins marketplace list
```

---

## 플러그인 업그레이드

```bash
claude plugins update summary
```

모든 플러그인 일괄 업그레이드:

```bash
claude plugins marketplace update
claude plugins update summary
```

---

## 플러그인 삭제

```bash
claude plugins uninstall summary
```

마켓플레이스 제거:

```bash
claude plugins marketplace remove summary-marketplace
```

---

## 사용 방법

### 슬래시 명령

```
/summary:summary 어제 경기
/summary:summary 오늘 경기
/summary:summary 최근 3일
```

### 자연어 요청

```
한화이글스 어제 경기 요약해줘
이글스 뉴스 레포트 만들어줘
한화 팬 반응 정리해줘
```

---

## 디렉토리 구조

```
summary/
├── .claude-plugin/
│   ├── plugin.json          # 플러그인 매니페스트
│   └── marketplace.json     # 마켓플레이스 카탈로그
├── skills/
│   └── summary/
│       ├── SKILL.md         # 오케스트레이션 워크플로우 정의
│       └── references/
│           └── report-template.md  # 레포트 템플릿
├── agents/
│   ├── eagles-researcher.md # 뉴스 수집 에이전트
│   ├── eagles-summarizer.md # 요약 작성 에이전트
│   └── eagles-reviewer.md   # 품질 검수 에이전트
├── commands/
│   └── summary.md           # /summary:summary 슬래시 명령
└── reports/                 # 생성된 레포트 저장 디렉토리
```

---

## 출력 예시

생성된 레포트는 `reports/eagles_{YYYYMMDD}_{HHMM}.md` 형식으로 저장됨.

```markdown
# 2026 한화이글스 뉴스 요약 레포트
> 보고 일자: 2026-06-22
> 독자: 야구 감독님

## 📊 시즌 현황
...

## 📰 [해외] 뉴스 1 — ...
## 📰 [국내] 뉴스 3 — ...

## 💬 팬 여론 종합
## ⚾ 감독님께 드리는 핵심 인사이트
```
