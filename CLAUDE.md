# 조각난 마도서 — 프로젝트 지침

## 프로젝트 개요

한국 판타지 웹소설 (카카오페이지 / 네이버시리즈 타깃). 100화 구성.
35세 과로사 한국 개발자 이재현이 10세 전쟁고아 리온으로 환생한 이야기.

## 폴더 구조

- `chapters/` — 각 화 마크다운 원고 (`ch{NN}.md`)
- `조각난_마도서/01_설정집/` — 세계관·역사·종족·마법체계·검술체계·국가·마도서·등장인물
- `조각난_마도서/02_회차구성/` — 100화 아크 구성 + 회차별 핵심 사건
- `조각난_마도서/03_본문/` — 빌드된 docx 파일
- `build_docs.py` — 마크다운 → docx 변환 스크립트

## 작업 원칙

### 한 화 분량
- 5000~5500자
- `---`, `***`, `===` 같은 구분선 사용 금지
- 소제목은 `## 소제목` (13pt, bold, 검정)

### 톤 매너리즘 회피
이전 화들에서 굳어졌던 매너리즘 어휘를 의식적으로 줄인다:
- "한 결" — 명사 의미가 또렷한 자리 외에는 사용 자제
- "한 호흡" — 부사 채움으로 사용하지 않기 (영창문 안 일부만 허용)
- "자리 잡았다" — 동의어 분산 (자리를 잡았다 / 들어앉았다 / 자리에 섰다 등)
- "자기" 대명사 — 한국어에서 부부/연인 호칭 또는 재귀대명사로만. 본문 서술 시점에서 리온을 가리킬 땐 "리온 / 그", 1인칭 화법에선 "저 / 제가" 사용

### 회차 작업 흐름
1. 아크구성표(`02_회차구성/100화_아크구성.docx`)의 해당 화 핵심 사건 확인
2. 직전 1~2화 컨텍스트 확인
3. 정상 톤으로 5000~5500자 집필
4. **review-team 에이전트로 검수 라운드 돌리기**
5. 다섯 검수자(novel-editor / plot-reviewer / character-reviewer / pacing-reviewer / continuity-checker) 모두 통과 판정 시 화 완료
6. 사용자에게 알림

## 에이전트 팀 사용 규칙 (중요)

### 서브에이전트 호출 정책 (절대 규칙)

이 프로젝트에서 서브에이전트를 호출할 때는 **반드시 teammate 시스템(team_name + name)을 사용**한다.
**`isolation: "worktree"` 사용을 절대 금지**한다. 어떤 상황에서도 예외 없음.

worktree로 spawn하면:
- 사용자에게 보이지 않음 (in-process)
- "Let me launch agents in worktrees" 메시지가 출력됨
- psmux 자동 패널 분할이 동작하지 않음

따라서 모든 서브에이전트 호출은:
1. `team_name` + `name` 파라미터로만 호출
2. `isolation` 파라미터를 지정하지 않거나, 명시적으로 teammate 모드
3. 단일 `Agent` tool 호출이 아니라 다중 spawnTeammate를 한 메시지에 묶어 병렬 호출

이유:
- 사용자가 psmux 환경에서 작업하고 있어, teammate 호출 시 각 에이전트가 별도 tmux 패널에 띄워져 실시간으로 진행 상황을 볼 수 있음
- worktree 격리는 in-process로 돌아 사용자에게 보이지 않음
- 검수 작업의 본질이 "각 검수자의 작업을 실시간으로 보기"이므로 worktree는 부적합

### 검수 라운드 표준 호출

화 검수 시 다음 한 줄 호출:

```
@review-team ch{N} 검수 라운드 돌려줘
```

review-team 에이전트가 다섯 검수자를 **병렬 teammate로 spawn**하여 각각 별도 패널에서 작업.

### 단일 검수자 직접 호출

특정 검수자와 1:1로 깊게 다듬을 때:

```
@novel-editor ch{N} 문장 다듬기 부탁해
@plot-reviewer ch{N} 떡밥만 점검해줘
@character-reviewer ch{N} 인물 톤만 봐줘
@pacing-reviewer ch{N} 박자 점검
@continuity-checker ch{N} 설정 정합 점검
```

이 경우에도 teammate 시스템 사용 (worktree 금지).

## 작업 환경

- OS: Windows 11 + Windows Terminal
- Shell: PowerShell 7+ (psmux teammate 모드 요구사항)
- 멀티플렉서: psmux (claude-code-fix-tty on)
- 에이전트 정의: `~/.claude/agents/` (novel-editor, plot-reviewer, character-reviewer, pacing-reviewer, continuity-checker, review-team)
