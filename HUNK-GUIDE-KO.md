# Hunk 분석 정리 (한국어)

> AI가 짠 코드를 사람이 검토하기 위한 터미널 diff 뷰어, **Hunk** 에 대한 조사 기록.
> 저장소 내용을 직접 확인하며 정리했습니다.

## 링크

| 항목                    | 주소                                     |
| ----------------------- | ---------------------------------------- |
| 내 저장소 (포크)        | https://github.com/bmshin94/hunk         |
| 원본 저장소             | https://github.com/modem-dev/hunk        |
| 공식 사이트             | https://hunk.dev                         |
| 공식 문서               | https://hunk.dev/docs/                   |
| 확장 모음 (GitHub 토픽) | https://github.com/topics/hunk-extension |
| 확장 목록 (공식)        | https://hunk.dev/extensions              |
| 커뮤니티 (Discord)      | https://discord.gg/WZFjaP6Gt8            |
| npm 패키지              | https://www.npmjs.com/package/hunkdiff   |

- 라이선스: **MIT** (상업적 이용 가능)
- 확인 시점 버전: **0.22.0**
- 스폰서: [Modem](https://modem.dev)

---

## 1. 이게 뭐하는 건가

한 줄 요약: **코드가 바뀐 부분을 터미널에서 예쁘고 조작 가능하게 보여주는 프로그램.**

`git diff`는 텍스트가 주르륵 흘러가서 뭐가 중요한지 안 보인다. Hunk는 같은 내용을
"읽는 diff"가 아니라 **"리뷰하는 diff"** 로 바꾼다.

주요 기능:

- 왼쪽 사이드바에 변경된 파일 목록
- 터미널인데 **마우스 클릭** 지원
- **AI/에이전트가 남긴 메모가 코드 바로 옆에 표시** (가장 큰 차별점)
- Split / Unified 레이아웃 자동 전환
- Watch 모드 (파일 저장하면 자동 새로고침)
- `hunk log` 커밋 히스토리 브라우저 (범위 선택 후 누적 diff 리뷰 가능)
- Git / Jujutsu / Sapling 지원

### 왜 만들어졌나

AI가 코드를 10배 빠르게 쏟아내는데 리뷰는 여전히 손으로 한다. 그 병목을 겨냥한 도구다.

---

## 2. 설치 및 사용법

### 설치

```bash
# macOS / Linux — Node 불필요, 단일 실행파일
curl -fsSL https://hunk.dev/install.sh | sh

# Windows 포함 — Node.js 22+ 필요
npm i -g hunkdiff

# 기타
brew install hunk
mise use -g hunk
```

확인: `hunk --version`

### 기본 명령어

```bash
hunk diff          # 현재 작업 중인 변경사항 (가장 많이 사용)
hunk show          # 마지막 커밋
hunk show HEAD~1   # 이전 커밋
hunk log           # 커밋 히스토리 브라우저
hunk diff --watch  # 자동 새로고침
```

### 화면 안 단축키 (핵심)

| 키                | 동작                            |
| ----------------- | ------------------------------- |
| `↑` `↓` / `j` `k` | 한 줄 이동                      |
| `[` `]`           | 이전/다음 변경 덩어리(hunk)     |
| `,` `.`           | 이전/다음 파일                  |
| `/`               | 검색 (`n` / `N` 다음·이전)      |
| `c`               | 코멘트 달기                     |
| `e`               | 해당 파일을 에디터로 열기       |
| `y`               | 복사                            |
| `1` `2` `0`       | Unified / Split / Auto 레이아웃 |
| `t`               | 테마 변경                       |
| `s`               | 파일 목록 토글                  |
| `?`               | 도움말                          |
| `q`               | 종료                            |

### 설정 파일

`~/.config/hunk/config.toml`

```toml
theme = "github-dark-default"
mode = "auto"          # auto / split / unified
line_numbers = true
tab_width = 4
sidebar = "auto"
```

### Git 기본 pager로 등록

```bash
git config --global core.pager "hunk pager"
```

이후 `git diff`, `git show` 가 자동으로 Hunk로 열린다.

---

## 3. 플러그인인가, 스킬인가, MCP인가

**셋 다 아니다. 독립 실행 CLI 프로그램이다.** 다만 셋 모두와 관련이 있다.

| 질문          | 답                                                     |
| ------------- | ------------------------------------------------------ |
| 플러그인인가? | 아니오. **플러그인을 받는 쪽** (자체 확장 시스템 보유) |
| 스킬인가?     | 아니오. **스킬을 제공하는 쪽**                         |
| MCP인가?      | **예전엔 맞았지만 지금은 제거됨**                      |

### 스킬은 "제공하는" 쪽

`packages/hunk/skills/hunk-review/SKILL.md` 에 에이전트용 사용설명서가 들어있다.

```bash
hunk skill path   # 스킬 파일 경로 출력
```

### MCP는 제거됨 (코드로 확인)

`packages/hunk/src/session/broker/brokerServer.ts:757` 에 옛 MCP 경로의 묘비(tombstone)가 남아있다.

> "This app no longer exposes agent-facing MCP tools. Use the session CLI instead."

0.3~0.5 버전엔 실제 MCP 서버였으나, 지금은 **CLI + 스킬 방식**으로 전환했다.
`hunk mcp serve` 는 `hunk daemon serve` 의 옛 별칭일 뿐 MCP 프로토콜과 무관하다.

> 인사이트: **MCP 없이도 좋은 에이전트 도구를 만들 수 있다.**
> "AI가 프로토콜을 몰라도 명령어만 알면 된다"는 판단.

### 자체 확장 시스템

```bash
hunk extension install acme/hunk-word-diff@v1.2.0
hunk extension list / update / remove
```

---

## 4. API 토큰이 필요한가

**필요 없다. 완전 무료.**

소스 전체를 검색한 결과 `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `apiKey` 등
**AI API를 호출하는 코드가 전혀 없다.**

### AI 코멘트는 어디서 오나

```
내 Claude Code (이미 쓰는 것)
      ↓ 코멘트 작성해서 전달
   Hunk (그리기만 함)
      ↓
    내 화면
```

Hunk는 "모니터"고, AI는 사용자가 이미 가진 것. **추가 비용 0원.**

### 로컬 보안 방식

| 항목        | 내용                                      |
| ----------- | ----------------------------------------- |
| 인증        | Ed25519 키쌍 자동 생성 (`credentials.ts`) |
| 파일 권한   | `0600` (본인만 읽기)                      |
| 바인딩      | `127.0.0.1:47657` — 루프백 전용           |
| 사용자 설정 | 불필요 (자동)                             |

외부 통신은 `hunk update`(버전 확인)와 `hunk extension install`(확장 설치) 뿐.
**코드는 외부로 전송되지 않는다.** 사내 코드도 안전.

---

## 5. 왜 GitHub에서 유명한가

> 별 개수는 직접 확인하지 못했고, 저장소 내부 증거로 추론한 내용이다.

1. **타이밍** — "AI 코드 리뷰"는 2026년 현재 가장 뜨거운 지점
2. **개발 속도** — PR 번호 #1099까지, 0.1.0 → 0.22.0, CHANGELOG 11만 6천 자
3. **설치 편의** — curl / npm / brew / mise / nix, Windows 지원, Node 없는 단일 바이너리
4. **Omarchy 기본 탑재** — DHH가 만든 리눅스 배포판에 기본 도구로 포함
5. **회사 지원** — Modem 스폰서, 개인 취미 프로젝트가 아님
6. **마케팅 투자** — `packages/term-video`(터미널 녹화 전용 패키지),
   `skills/hunk-launch-video`(런칭 영상 제작 스킬), 전용 사이트, Discord 운영
7. **기술적 신선함** — **React로 터미널 UI**를 만든다 (OpenTUI)

---

## 6. 로컬 에이전트 구축에 도움이 되는가

도움이 된다. 3단계로 활용 가능.

### 레벨 1 — 그대로 사용

```bash
# 터미널 A
hunk diff

# 터미널 B (Claude Code)
"Hunk 스킬 로드해서 리뷰해줘"
```

에이전트가 쓰는 명령어:

```bash
hunk session list                                    # 열린 창 찾기
hunk session review --repo . --json                  # 변경 구조 읽기
hunk session navigate --repo . --file a.ts --hunk 2  # 화면 이동
hunk session comment add ...                         # 코멘트 달기
hunk session highlight add ...                       # 범위 하이라이트
```

### 레벨 2 — session-broker SDK 재사용 (핵심 가치)

```
packages/session-broker/       # 본체
packages/session-broker-core/  # 프로토콜
packages/session-broker-bun/   # Bun 어댑터
packages/session-broker-node/  # Node 어댑터
```

"**AI 에이전트가 로컬 GUI 앱을 조종하는**" 구조가 재사용 가능한 SDK로 분리되어 있다.

이미 구현된 것:

- 루프백 전용 로컬 데몬
- Ed25519 인증 + 권한 스코프 (`navigate_to_hunk`, `comment`, `highlight` 등)
- 다중 창 라우팅 (`{appId, sessionId}` 네임스페이스)
- WebSocket 재연결 처리
- 리소스 제한, 버전 호환성 관리

→ 어떤 로컬 앱이든 "AI가 조종하게" 만들 때 인증/통신을 처음부터 짤 필요가 없다.

### 레벨 3 — 설계 패턴 학습

| 배울 점         | 내용                                      |
| --------------- | ----------------------------------------- |
| MCP를 버린 이유 | 프로토콜보다 CLI + 스킬이 단순하다는 판단 |
| 권한 최소화     | 필요한 동작만 스코프로 분리               |
| 스킬 작성법     | `SKILL.md`가 좋은 교본                    |
| 역할 분리       | TUI는 사람용, CLI는 에이전트용            |

---

## 7. React나 PHP로 만들 수 있는가

### React — 이미 React다

Hunk 본체가 React로 만들어졌고(OpenTUI), **확장도 React로 작성한다.**

```tsx
// ~/.config/hunk/extensions/my-pane.tsx
import { useMemo } from "react";
import type { ExtensionPaneProps, HunkExtensionAPI } from "hunkdiff/extension";

function MyPane({ files, selectedFileId, theme, actions }: ExtensionPaneProps) {
  const ordered = useMemo(() => [...files].sort((a, b) => a.path.localeCompare(b.path)), [files]);

  return (
    <scrollbox scrollY={true} width="100%" height="100%">
      {ordered.map((file) => (
        <text
          key={file.id}
          content={` ${file.path}  +${file.stats.additions} -${file.stats.deletions}`}
          onMouseDown={() => actions.selectFile(file.id)}
        />
      ))}
    </scrollbox>
  );
}

export default function (hunk: HunkExtensionAPI) {
  hunk.registerPane({ id: "flat", title: "Flat files", placement: "right", component: MyPane });
}
```

`<div>` 대신 `<text>`, `<scrollbox>` 를 쓰는 것만 다르다.

> 주의: **React를 직접 설치하면 안 된다.** 호스트가 자기 React 인스턴스를 제공한다.
> 직접 번들하면 hooks dispatcher가 두 개가 되어 컴포넌트가 깨진다.
> `react`, `@opentui/*`, `hunkdiff/extension` 은 `devDependencies`(타입용)에만 둔다.

브라우저 버전도 개발 중이다 (`docs/browser-review-rebuild.md`, Phase 4/6 진행).

### PHP — TUI 본체는 비현실적, 웹 버전 백엔드로는 가능

PHP는 "요청 → 응답" 구조라 터미널 실시간 렌더링/이벤트 루프에 맞지 않는다.

대신 이런 구조는 충분히 가능하다.

```
브라우저 (React)        ← diff 화면 렌더링
      ↕ HTTP / JSON
PHP 백엔드 (Laravel)    ← git diff 파싱, 코멘트 DB, 로그인/권한, AI API 호출
```

오히려 "팀용 웹 코드리뷰 도구"는 이 구조가 더 적합하다.

---

## 8. 수익화 분석

### 먼저: 구조적 제약 3가지

#### (1) 유료 확장을 팔기 어렵다

`docs/extensions.md:128`:

> "Extensions are shared as plain git repositories — **there is no registry to publish to**"

확장 배포 = git clone. TypeScript 소스가 그대로 노출되고, 결제/라이선스 검증 장치가 없다.
**DRM이 통하지 않는다.**

→ **결론: "확장은 무료로 배포하고, 수익은 서버에서 받는다"가 유일한 길.**

```
확장 (무료, 오픈소스)  →  API 키 인증  →  내 서버 (유료)
소스를 복제해도 서버 계정이 없으면 작동하지 않음
```

#### (2) 감사(Audit) 기능은 기술적으로 이미 준비됨

`examples/extensions/review-snapshot-export/` 에서 확인:

```js
ctx.review.snapshot();
// 저장된 모든 리뷰 노트 + 파일 식별자 + 리비전을 한 번에 반환
```

문서에 "Publishers can use the same check **before an irreversible network request**" 라는
문장이 있어, 서버 전송을 공식적으로 상정하고 있다.

#### (3) 플랫폼 연동은 매우 쉽다

`examples/extensions/github-pr/` 가 **의존성 0개**로 구현되어 있다.
구조는 `플랫폼 API에서 diff 가져오기 → 임시 패치 파일 → hunk patch로 위임` 뿐.

GitHub은 공식 예제가 있지만 **GitLab, Bitbucket, Gerrit, Azure DevOps는 비어있다.**

#### 확장 API로 아직 못 하는 것 (`docs/extensions.md:2469`)

- 메뉴 항목 추가
- 커스텀 노트 렌더러
- 세션 커맨드 추가
- (그리고 확장 API 자체가 **experimental** — 마이너 버전마다 깨질 수 있음)

---

### 수익 모델 5가지

| 모델                   | 난이도    | 수익성     | 속도      | 리스크   | 추천도 |
| ---------------------- | --------- | ---------- | --------- | -------- | ------ |
| 1. AI 코드 감사 로그   | 중        | 매우 높음  | 느림      | 중       | ★★★★★  |
| 2. 플랫폼 커넥터 선점  | 낮음      | 낮음(간접) | 빠름      | 낮음     | ★★★★★  |
| 3. 팀 리뷰 동기화 SaaS | 높음      | 높음       | 느림      | **높음** | ★★     |
| 4. 타 분야 이식        | 매우 높음 | 매우 높음  | 매우 느림 | 높음     | ★★★    |
| 5. 콘텐츠 / 컨설팅     | 매우 낮음 | 중         | 즉시      | 낮음     | ★★★★   |

#### 모델 1 — AI 코드 감사 로그 (B2B, 최고 수익성)

리뷰 종료 시 `ctx.review.snapshot()` 으로 "누가, 언제, 어떤 AI 코드를, 어떻게 검토했는지"를
서버로 전송하고 규제 대응 리포트를 생성한다.

**수요 근거:** EU AI Act(인간 감독 증빙 의무), 금융권 감사, 의료기기 SW 문서화, ISO 27001 / SOC 2.
규제는 "안 사면 벌금"이라 지갑이 가장 빨리 열린다.

가격(가정): Free(로컬 내보내기만) / Team 좌석당 $15월 / Enterprise 연 $10,000~

#### 모델 2 — 플랫폼 커넥터 선점 (가장 먼저 시작할 것)

```bash
hunk gl 123      # GitLab      (비어있음)
hunk bb 123      # Bitbucket   (비어있음)
hunk gerrit 123  # Gerrit      (비어있음)
hunk ado 123     # Azure DevOps (비어있음)
```

직접 수익은 없지만 **유입 경로**다. 무료 홍보 채널 2개:

1. GitHub 저장소에 `hunk-extension` 토픽 → https://github.com/topics/hunk-extension 자동 노출
2. `website/src/data/extensions.ts` 에 PR → https://hunk.dev/extensions 공식 등재

GitLab은 유럽/기업 점유율이 높아 모델 1의 고객층과 정확히 겹친다.

#### 모델 3 — 팀 리뷰 동기화 SaaS (비추천)

**리스크:** `docs/browser-review-rebuild.md` 기준 **본체가 이미 브라우저 공유 리뷰를 개발 중**이다.
본체가 흡수하면 제품이 사라진다. 하려면 본체가 안 할 영역(Slack 연동, 결재 워크플로)으로 차별화 필요.

#### 모델 4 — 타 분야 이식 (고위험 고수익)

`session-broker` SDK로 "AI가 화면을 짚어주며 검토를 돕는다"를 다른 도메인에 적용:

- **계약서 diff 리뷰어** (법무팀/로펌 — 시장 가장 큼)
- DB 스키마 diff, 데이터/CSV diff, 디자인 diff

개발자 시장은 경쟁이 치열하고 무료를 원하지만, 법률 시장은 시간당 비용이 비싸 지갑이 열린다.

#### 모델 5 — 콘텐츠 / 컨설팅 (자본 0, 즉시 시작)

한국어 Hunk 콘텐츠는 거의 없어 선점 가능. 다만 **수익 모델이라기보다 마케팅 엔진**으로 쓰는 게 맞다.
콘텐츠로 포지셔닝 확립 → 모델 1 영업 시 신뢰도 확보.

---

### 추천 전략: 2 + 5 → 1

> 커넥터로 이름을 알리고(2), 콘텐츠로 신뢰를 쌓고(5), 감사 로그로 수익을 낸다(1).

#### 90일 로드맵

**1~30일 — 씨앗 뿌리기 (비용 0원)**

- [ ] GitLab 커넥터 확장 제작 (github-pr 예제에서 API만 교체)
- [ ] GitHub 공개 + `hunk-extension` 토픽 추가
- [ ] `website/src/data/extensions.ts` 등재 PR
- [ ] 한국어 소개 글 1편

**31~60일 — 신뢰 쌓기**

- [ ] Bitbucket 또는 Gerrit 커넥터 추가
- [ ] 감사 로그 확장 프로토타입 (로컬 JSON 내보내기만, 무료 → 수요 검증)
- [ ] 한국어 콘텐츠 3편 추가
- [ ] Hunk Discord 활동 시작

**61~90일 — 수익화**

- [ ] 감사 로그 서버 구축 (PHP/Laravel로 충분)
- [ ] 유료 티어 오픈 (클라우드 보관 + 리포트 PDF)
- [ ] 무료 사용자 중 기업 이메일 사용자에게 직접 연락
- [ ] 규제 산업(금융/의료) 파일럿 1곳

---

### 피해야 할 것

| 하지 말 것                       | 이유                                                               |
| -------------------------------- | ------------------------------------------------------------------ |
| 본체 포크해서 자기 제품으로 판매 | 6개월에 0.1 → 0.22. 개발 속도를 따라잡을 수 없다                   |
| 유료 확장 판매                   | 소스가 노출되어 복제 즉시 가능                                     |
| 본체 로드맵에 있는 기능 개발     | `docs/browser-review-rebuild.md` 확인 필수                         |
| "Hunk" 이름/로고 사용            | 상표 이슈 가능. `"~ for Hunk"` 정도로                              |
| 확장 API 버전 미고정             | experimental이라 마이너 버전마다 깨질 수 있음 → `@v0.22.0` 핀 고정 |

---

## 최종 요약

| 질문               | 답                                                        |
| ------------------ | --------------------------------------------------------- |
| 정체               | 독립 CLI 앱. AI 코드 리뷰용 터미널 diff 뷰어              |
| 플러그인/스킬/MCP? | 전부 아님. 스킬은 제공하는 쪽, MCP는 제거됨               |
| API 토큰           | 불필요. 완전 무료 (AI는 사용자 것을 사용)                 |
| 인기 이유          | 타이밍 + 개발 속도 + 설치 편의 + Omarchy 탑재 + 회사 지원 |
| 에이전트 구축      | `session-broker` SDK가 핵심 자산                          |
| React / PHP        | React는 이미 그것. PHP는 웹 버전 백엔드로 가능            |
| 수익화             | **코드는 무료, 서버로 과금.** 감사 로그(B2B)가 최고 유망  |

### 핵심 문장

> **"코드는 공짜로 주고, 서버로 돈 받아라."**
> 소스가 전부 노출되는 배포 구조에서는 이것이 유일한 길이다.
