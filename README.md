# LLM Wiki Starter

[Karpathy의 LLM Wiki 패턴](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)과 [careerhackeralex/memory](https://careerhackeralex.vercel.app/memory) 시각화를 합친 **개인 지식 그래프 템플릿**.

- **콘텐츠**: markdown 파일 (Claude/Codex로 자동 추가 또는 직접 작성)
- **시각화**: Next.js 14 SSG + react-force-graph-2d (다크 모드, hover 인접 하이라이트)
- **호스팅**: Cloudflare Pages 또는 Vercel 무료 플랜 (순수 정적 export)
- **편집 → 자동 배포**: git push 1~2분 후 라이브 갱신
- **자동 연결**: Claude Code로 새 노드 추가 시 기존 노드와 의미적으로 연결

---

## 사용 모델 한눈에

```
Use this template → 본인 GitHub private repo 생성
                   ↓
              로컬 git clone
                   ↓
       Cloudflare Pages 또는 Vercel에 연결
                   ↓
   본인 PC에서 Claude Code/Codex로 노드 추가
                   ↓
              git push
                   ↓
       자동 빌드 → 자동 배포 (1~2분)
```

---

## 빠른 시작 (10분)

### 1. 이 repo를 본인 GitHub로 복제

GitHub에서 **"Use this template"** 버튼 클릭 → 본인 계정에 새 repo 생성 (**private 추천**, 회사 자료는 무조건 private).

또는 CLI로:
```bash
gh repo create my-wiki --template xxx-sj/llm-wiki-starter --private --clone
cd my-wiki
```

### 2. 로컬에서 viewer 실행 (선택)

배포 전에 로컬에서 미리 확인:
```bash
cd viewer
npm install
npm run dev
# http://localhost:3000 → welcome 노드 1개 그래프 확인
```

### 3. 배포 — 호스팅 선택

| 사용 | 추천 |
|---|---|
| **회사/팀 자료 포함, 동료 여럿이 쓸 예정** | **Cloudflare Pages** (무료 플랜이 상업적 사용 허용) |
| 개인 사이드 프로젝트, 본인만 | Vercel 또는 Cloudflare 둘 다 OK |

#### 옵션 A. Cloudflare Pages (권장 — 회사용 무료)

1. https://dash.cloudflare.com → **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**
2. GitHub 연결 (필요시 Cloudflare GitHub App 권한 부여 — private repo 선택해 권한 줘야 함)
3. 본인 repo 선택 → **Begin setup**
4. Build configuration:
   - **Framework preset**: `Next.js (Static HTML Export)`
   - **Build command**: `cd viewer && npm install && npm run build`
   - **Build output directory**: `viewer/out`
   - **Root directory**: 비워두기
   - **Environment variables**:
     - `NODE_VERSION` = `20`
5. **Save and Deploy** → 1~2분 후 `<project>.pages.dev` URL 발급
6. 이후 main push마다 자동 빌드/배포

**Cloudflare Pages 무료 플랜**:
- ✅ 상업적 사용 허용 (회사 자료 OK)
- ✅ Bandwidth 무제한
- ✅ Builds 500/월 (=하루 16번)
- ✅ Custom domain 무료
- ✅ Preview deployment (PR마다 URL)

#### 옵션 B. Vercel (개인용)

1. https://vercel.com/new → 본인 repo 선택
2. **Root Directory를 `viewer`로 변경** ← 중요
3. Framework Preset: Next.js (자동 감지)
4. **Deploy** 클릭 → 1~2분 후 `<project>.vercel.app` 발급

**Vercel Hobby 무료 플랜 주의**:
- ⚠️ **Personal use만 허용 — 회사 사용 금지** (ToS 위반 시 차단)
- ✅ Bandwidth 100GB/월
- ✅ 빌드 100/일

회사 자료 한 글자라도 들어가면 Cloudflare Pages 권장.

### 4. 첫 노드 추가

#### Claude Code로 (자동 연결 + 분해 제안)
```bash
cd <repo root>
claude   # Claude Code 실행
> /wiki-ingest https://your-favorite-article.com
```
Claude가 다음을 자동:
- 원본을 `raw/`에 저장
- 노드 후보 분해 (id, node_type, scope 제안)
- 기존 노드와의 연결 후보 (`links:` + 엣지 타입) 제안
- 승인하면 `wiki/personal/{node_type}/...md` 작성
- `wiki/log.md` 업데이트

#### Codex로
Codex에 같은 slash command 매핑하거나, README의 `CLAUDE.md`를 system prompt로 주입.

#### 직접 markdown 작성
```bash
cat > wiki/personal/통찰/$(date +%F)-first.md <<'EOF'
---
id: 2026-XX-XX-first
title: 첫 통찰
node_type: 통찰
memory_type: mental_model
created: 2026-XX-XX
last_reviewed: 2026-XX-XX
links: []
---

# 첫 통찰

여기 본문.
EOF
```

작성 후 `/wiki-validate wiki/personal/통찰/...md`로 frontmatter 검증 가능.

### 5. welcome 노드 정리 (선택)

```bash
rm wiki/personal/주제/2026-05-19-welcome.md
```

### 6. push → 자동 배포

```bash
git add . && git commit -m "first node"
git push
```

Cloudflare/Vercel webhook → 자동 재빌드 → 1~2분 후 배포.

---

## 구조

```
.
├── CLAUDE.md                        # Claude Code 운영 지침 (스키마, 워크플로우)
├── .claude/commands/                # 슬래시 커맨드 4개
│   ├── wiki-ingest.md               # 새 자료 → 노드 분해 + 자동 연결
│   ├── wiki-lint.md                 # 깨진 링크, 고아 노드 등 점검
│   ├── wiki-stats.md                # 노드/엣지 통계
│   └── wiki-validate.md             # 직접 쓴 markdown 보강
├── raw/                             # 원본 자료 (append-only)
├── wiki/
│   ├── personal/                    # 개인 콘텐츠 (배포 포함)
│   │   ├── 의미/ 통찰/ 절차/ 사건/ 주장/ 주제/
│   ├── work/                        # 회사/팀 자료 (배포에서 통째로 제외)
│   ├── index.md                     # 사람용 목차
│   └── log.md                       # 변경 연대기
└── viewer/                          # Next.js 14 SSG viewer
    ├── app/                         # / /node/[id] /log /stats
    ├── components/                  # ForceGraph, NodePanel, FilterBar 등
    ├── lib/                         # build-graph.ts, filter-graph.ts, schema.ts
    ├── scripts/build.ts             # markdown → graph.json 변환
    └── next.config.mjs              # output: 'export' (정적)
```

---

## 스키마 (6 + 9)

### 노드 타입 6종

| 타입 | 정의 | 예 |
|---|---|---|
| 의미 | 개념/용어의 정의 | "REST란?" |
| 통찰 | 관찰에서 추출한 인사이트 | "캐시 워밍은 비싸다" |
| 절차 | 단계적 how-to | "Vercel 배포 단계" |
| 사건 | 특정 시점의 일/경험 | "장애 회고 X사" |
| 주장 | 누군가의 의견/논증 | "RAG보다 wiki가 낫다 (Karpathy)" |
| 주제 | 다른 노드를 묶는 카테고리 | "지식관리" |

### 엣지 타입 9종

| 타입 | 의미 |
|---|---|
| 지지 | A가 B를 뒷받침 |
| 반박 | A가 B를 반박 |
| 확장 | A가 B를 확장/심화 |
| 구체화 | A가 B의 구체 사례 |
| 정련 | A가 B를 다듬은 후속 |
| 유사 | A와 B가 비슷한 패턴 |
| 촉발 | A가 B 생각의 계기 |
| 주제태그 | A가 주제 B에 속함 |
| 전제 | A가 B의 전제 조건 |

엣지는 directed (방향성 있음). 자세한 frontmatter 규격은 [`CLAUDE.md`](CLAUDE.md) 참고.

---

## 회사 자료 보호

- `wiki/work/` 아래 노드는 **빌드에서 통째로 제외**됨 (기본값)
- 환경변수 `WIKI_INCLUDE_WORK=true`일 때만 빌드에 포함
- Cloudflare/Vercel 배포 빌드는 환경변수 미설정 → personal만 포함
- 로컬에서 work 포함 보고 싶을 때: `WIKI_INCLUDE_WORK=true npm run dev`

→ 회사 자료를 wiki/work/에 두면 **로컬에서만 보이고 공개 URL에는 절대 안 올라감**.

---

## AI 챗봇 추가하기 (선택)

[careerhackeralex/memory](https://careerhackeralex.vercel.app/memory)처럼 wiki 위에 RAG 챗봇을 얹어 "내가 쌓은 지식 기반으로 질문에 답"하게 만드는 패턴. starter는 기본적으로 챗봇 미포함이지만, 아래 가이드로 직접 추가할 수 있다.

### 기본 흐름 (RAG)

```
사용자 질문
   ↓
질문 → 임베딩 (e.g. OpenAI text-embedding-3-small)
   ↓
임베딩 → 코사인 유사도 → top-K 노드
   ↓
top-K 본문 + 질문 → LLM (Claude/GPT) → 답변
   ↓
답변 + 인용된 노드 ID → UI 표시
```

### 패턴 3가지 — 비용/보안/단순함 트레이드오프

| 패턴 | 누가 LLM 비용 부담 | API key 위치 | 호스팅 영향 | 추천 시점 |
|---|---|---|---|---|
| **A. BYOK** (브라우저) | 사용자 각자 | 사용자 `localStorage` | ✅ 정적 그대로 | 본인 + 가까운 1~5명만 |
| **B. Cloudflare Functions** | repo 주인 | Cloudflare 환경변수 | ⚠️ Pages Functions 추가 | 회사 동료 5~20명 |
| **C. 별도 백엔드** | repo 주인 | 외부 서버 환경변수 | ❌ 동적 백엔드 필요 | SaaS, 익명 일반 사용자 |

### 패턴 A — BYOK (가장 빠른 시작, 정적 호스팅 유지)

**빌드 타임 (`scripts/build.ts` 확장)**:
1. 환경변수 `OPENAI_API_KEY` 있을 때만 임베딩 생성
2. 모든 노드 본문 → OpenAI `text-embedding-3-small` API
3. `public/embeddings.json` 출력: `{ "<node_id>": [0.123, -0.456, ...] }`
4. CI(빌드)에서 키 없으면 임베딩 스킵, keyword search만 작동

**런타임 (브라우저, 새 `<ChatPanel>` 컴포넌트)**:
1. 사용자가 첫 사용 시 본인 API key 입력 → `localStorage.setItem('llm_api_key', ...)`
2. 질문 → fetch로 OpenAI Embeddings API 호출 (브라우저 → OpenAI 직접, CORS OK)
3. `embeddings.json`과 코사인 유사도 → top-5 노드
4. top-5 본문 + 질문 → Anthropic/OpenAI Chat Completions (SSE streaming)
5. 답변 markdown 렌더, 인용된 노드 ID는 자동 링크화

**필요한 라이브러리**:
- `@anthropic-ai/sdk` 또는 `openai` (브라우저 모드)
- 또는 fetch 직접 (둘 다 CORS 허용)

**비용 (BYOK 사용자 입장)**:
- 빌드 시 임베딩: 100 노드 / 50k tokens = $0.001
- 질문 1회: 임베딩 + LLM = ~$0.01 (10원)

**보안 주의**:
- localStorage에 평문 API key 저장 — XSS 발생 시 유출 위험
- CSP (Content Security Policy) 헤더 설정 권장
- 본인/팀 신뢰 환경에서만 권장

### 패턴 B — Cloudflare Pages Functions (회사용 권장)

**빌드는 패턴 A와 동일** (embeddings.json 생성).

**런타임 (Cloudflare Pages Functions 추가)**:
1. `functions/api/chat.ts` 생성 — Cloudflare Pages는 이 경로를 자동으로 Edge function으로 노출
2. 사용자 질문 → POST `/api/chat`
3. Function이 환경변수의 API key로 OpenAI/Anthropic 호출
4. SSE로 답변 스트리밍

```typescript
// functions/api/chat.ts (개념적 예시)
export async function onRequestPost(context: any) {
  const { question, topK } = await context.request.json();
  const apiKey = context.env.ANTHROPIC_API_KEY;
  
  // 1. 질문 임베딩 (OpenAI)
  // 2. embeddings.json fetch + 코사인 유사도
  // 3. top-K 노드 본문 + question → Anthropic streaming
  // 4. SSE 응답
  
  return new Response(stream, {
    headers: { 'content-type': 'text/event-stream' }
  });
}
```

**Cloudflare 무료 한도**:
- Pages Functions: **100,000 requests/day** (무료)
- Vectorize (선택, 벡터 DB로): **5M vectors 무료**
- Workers AI (선택, 자체 LLM): 일부 모델 무료 — 단 한국어 약함

**설정**:
- Cloudflare Pages 대시보드 → Settings → Environment Variables → `ANTHROPIC_API_KEY` 또는 `OPENAI_API_KEY` 추가
- Production + Preview 환경 모두 설정

**장점**:
- 사용자에게 API key 요구 안 함
- 모든 사용자(동료, 외부)가 즉시 사용 가능
- repo 주인이 사용량 통제 (rate limit, 인증 추가 가능)

**비용 (repo 주인 입장)**:
- 100 노드, 동료 10명 × 일 10질문 = 월 3,000 질문 = ~$30/월 (Claude Haiku 기준)
- 더 저렴하게: Claude Haiku 대신 GPT-4o-mini = ~$5/월

### 패턴 C — 별도 백엔드 (SaaS급 운영)

별도 서비스(Vercel Functions, Railway, Fly.io 등)로 백엔드 분리. 인증, rate limit, 사용자별 history, 사용량 분석 등 본격 운영용. starter 범위 밖.

### Phase별 구현 권장

starter 받은 사람이 점진적으로 추가:

**Phase 1 (1~2일, API key 0)** — keyword search
- `scripts/build.ts`에 MiniSearch/Lunr 인덱스 빌드 추가
- `public/search-index.json` 출력
- 상단에 검색창 + 결과 노드 리스트 + 그래프 강조
- 이것만 해도 "검색 용이"는 충족

**Phase 2 (3~5일, BYOK 필요)** — semantic + RAG
- 패턴 A 또는 B로 embeddings + LLM 추가
- 채팅 UI 컴포넌트
- 답변 스트리밍

**Phase 3 (2~3일, 다듬기)** — UX 완성
- 대화 히스토리 (localStorage)
- 답변 인용 노드 → 그래프 강조
- 모바일 친화 레이아웃

### 추가 참고

- 검색 우선이면 [MiniSearch](https://github.com/lucaong/minisearch) 추천 (한글 token 분리 옵션)
- 임베딩 비용 최소화: 본문 너무 길면 chunking (~512 token)
- 한국어 임베딩 품질: OpenAI > Cohere multilingual > Cloudflare BGE-M3

---

## 협업

개발팀이라면 git/PR 흐름 그대로:

1. 팀원을 GitHub repo collaborator로 초대
2. 각자 본인 PC에 clone + Claude Code 설치
3. branch에서 작업 → PR → 머지 → Cloudflare/Vercel 자동 배포

forward-only `links:` 정책 덕분에 새 노드 추가 시 기존 파일을 수정하지 않아 **conflict가 거의 발생하지 않음**.

---

## 라이선스

MIT

## Credits

- Knowledge graph 시각화 컨셉: [careerhackeralex/memory](https://careerhackeralex.vercel.app/memory)
- LLM Wiki 워크플로우 패턴: [Andrej Karpathy](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 메모리 타입 분류: [Hindsight](https://hindsight.vectorize.io/)
