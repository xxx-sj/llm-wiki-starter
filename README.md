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

## AI 챗봇 (RAG) — 코드 포함됨, API key 1개만 등록하면 작동

[careerhackeralex/memory](https://careerhackeralex.vercel.app/memory)처럼 wiki 위에 "내가 쌓은 지식 기반으로 답하는 챗봇"을 패턴 B (Cloudflare Pages Functions + Cloudflare Access)로 **이미 구현되어 있음**. OpenAI API key 1개를 Cloudflare 환경변수에 등록하면 바로 작동 — 임베딩과 답변 둘 다 OpenAI.

### 구조

```
사용자 질문
   ↓
POST /api/chat   ← Cloudflare Pages Function (functions/api/chat.ts)
   ↓
1. question → OpenAI embedding (text-embedding-3-small)
2. /embeddings.json fetch + cosine similarity → top-K 노드
3. top-K 본문 + system prompt + question → OpenAI gpt-4o-mini streaming
   ↓
SSE 응답 (cited nodes meta + raw OpenAI stream)
   ↓
브라우저 UI가 markdown 렌더 + 인용 노드 그래프 강조
```

### 포함된 파일

| 파일 | 역할 |
|---|---|
| `viewer/scripts/build-embeddings.ts` | 빌드 시 노드 본문 → OpenAI 임베딩 → `public/embeddings.json` |
| `functions/api/chat.ts` | Cloudflare Pages Function — RAG 파이프라인 + SSE 응답 |
| `viewer/lib/parse-sse.ts` | SSE 스트림 파서 (`cited` 이벤트 + OpenAI delta + `[DONE]`) |
| `viewer/components/ChatPanel.tsx` | 상단 검색바형 chat UI (스트리밍 답변 + cited 칩 + abort/clear) |
| `viewer/components/AnswerRenderer.tsx` | markdown 렌더 + `[node-id]` 인용 자동 링크화 |
| `viewer/components/ForceGraphCanvas.tsx` | 답변 인용 노드 자동 강조 (`chatCitedIds` prop) |
| `viewer/app/ClientGraphPage.tsx` | ChatPanel을 그래프 상단 floating overlay로 배치 |
| `viewer/package.json` `prebuild` | `build.ts && build-embeddings.ts` 체이닝 |

**Chat UI 포함 — OpenAI API key 1개만 등록하면 즉시 작동**. 별도 컴포넌트 작성 불필요.

### 활성화 단계 (5분)

#### 1. API key 발급
- **OpenAI** (임베딩 + 답변 모두): https://platform.openai.com/api-keys — $5 충전 (한참 씀)

#### 2. Cloudflare Pages 환경변수 등록
대시보드 → 본인 Pages 프로젝트 → **Settings** → **Environment Variables** → **Production**:
- `NODE_VERSION` = `20`
- `OPENAI_API_KEY` = `sk-proj-...`

두 변수 설정 후 **Redeploy** (수동 트리거 또는 빈 commit push).

#### 3. 빌드 로그 확인

첫 빌드:
```
[build-graph] wrote .../graph.json (personal only)
[build-embeddings] 0 reused, 5 to embed via text-embedding-3-small...
[build-embeddings] 5/5
[build-embeddings] wrote .../embeddings.json (5 nodes, 1536 dim, ~143KB) — 0 reused, 5 new
```

본문 변경 없는 다음 빌드 (incremental cache 적중):
```
[build-graph] wrote .../graph.json (personal only)
[build-embeddings] all 5 nodes cached — no OpenAI calls
[build-embeddings] wrote .../embeddings.json (5 nodes, 1536 dim, ~143KB) — 5 reused, 0 new
```

`OPENAI_API_KEY`가 없으면 임베딩 단계는 자동 skip (graph.json만 생성 → /api/chat은 `embeddings_missing` 에러).

#### 💡 비용 절감 — `embeddings.json` git commit 권장

`viewer/public/embeddings.json`은 **빌드 산출물이자 다음 빌드의 캐시**. SHA-256 hash로 노드 (title + body) 변경 감지 → **변경된 노드만 OpenAI 호출**.

`viewer/.gitignore`에서 `public/embeddings.json` 줄이 주석 처리돼 있어요. 그래서 git에 commit 가능. 처음 빌드 후:

```bash
git add viewer/public/embeddings.json
git commit -m "cache: initial embeddings"
git push
```

이러면 Cloudflare 빌드에서도 이전 캐시를 시작점으로 사용해 **본문이 안 바뀐 노드는 API 호출 0**. 노드 1만 개 중 1개 수정한 push → OpenAI 호출 1번.

##### Hash 동작 — 무엇이 변경 감지되나

`viewer/scripts/build-embeddings.ts`에서 각 노드의 hash 입력:

```ts
const text = `${node.title}\n\n${htmlToText(body)}`.slice(0, 8000);
const hash = sha256(text);   // 64자 hex
```

| 변경 | hash 변경? | 재계산? |
|---|---|---|
| 본문 markdown 한 글자 수정 | ✅ | ✅ OpenAI 호출 |
| frontmatter `title` 변경 | ✅ | ✅ |
| `node_type`, `memory_type` 변경 | ❌ | ❌ 재사용 |
| `links:` 엣지 추가/제거 | ❌ | ❌ |
| `tags`, `sources`, `confidence` 변경 | ❌ | ❌ |
| `last_reviewed`, `created` 변경 | ❌ | ❌ |
| 파일명/id 변경 | 새 id로 인식 → 신규 임베딩 | ✅ 신규 |
| 노드 파일 삭제 | graph에서 사라짐 → cache 자동 제외 | — |
| `EMBED_MODEL` 상수 변경 | cache `model` 필드 비교로 전체 무효화 | ✅ 전체 |

→ **본문/제목 바뀌면 정확히 그 노드만**, 메타 데이터 정리(`/wiki-lint` 등)는 비용 0.

##### 시나리오 예시

**1) 본문 오타 수정** → 그 노드 1개만 OpenAI 호출
```
[build-embeddings] 4 reused, 1 to embed via text-embedding-3-small...
```

**2) `last_reviewed` 날짜만 갱신** → 호출 0
```
[build-embeddings] all 5 nodes cached — no OpenAI calls
```

**3) 새 엣지(`links:`) 추가** → 호출 0 (graph.json에만 반영, embedding은 그대로)

**4) 새 노드 추가** → 신규 1개만 호출

**5) 모델 변경** (`text-embedding-3-small` → `-large`) → cache의 `model` 비교로 무효화, 전체 재계산

#### 4. API 동작 테스트
```bash
curl -N -X POST https://<your-cf-url>/api/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "이 wiki는 어떤 시스템이야?"}'
```

응답 (SSE — OpenAI Chat Completions 포맷):
```
event: cited
data: [{"id":"...", "title":"...", "node_type":"...", "score":0.872}]

data: {"choices":[{"delta":{"content":"이"}}]}
data: {"choices":[{"delta":{"content":" wiki는"}}]}
data: {"choices":[{"delta":{"content":" ..."}}]}
data: [DONE]
```

### 보안 강화 (필수 권장)

#### Cloudflare Access (Zero Trust) — 본인만 접근
대시보드 → **Zero Trust** → **Access** → **Applications** → **Add an application** → **Self-hosted**:
- Application name: `LLM Wiki`
- Application domain: `<your-cf-url>` (또는 custom domain)
- Identity provider: Google (또는 GitHub)
- Policy: `Allow` + Email is `<본인-email>`
- 저장

→ 본인 Google 계정 SSO 통과해야 wiki 전체 접근 가능. 다른 사람이 URL 알아도 막힘.

회사 팀과 공유: 정책을 `Email ends with @yourcompany.com`으로 변경.

#### WAF Rate Limit — abuse 차단
도메인 → **Security** → **WAF** → **Rate limiting rules**:
- Path equals `/api/chat`, Method `POST`
- Threshold `10 requests per 1 minute` per IP
- Action `Block`

#### 추가 보안 (코드에 이미 적용됨)
- Input 길이 1000자 cap
- topK clamp [1, 10]
- system prompt에 prompt-injection 가드
- "(wiki 외부 지식)" prefix 강제 (hallucination 표시)

### 비용 감각 (OpenAI 통합 모델)

| 시나리오 | 월 비용 |
|---|---|
| 본인 1명 + 일 10 질문 = 월 300 질문 | ~$0.5 |
| 팀 10명 + 일 10 질문 = 월 3,000 질문 | ~$5 |
| 팀 50명 + 일 20 질문 = 월 30,000 질문 | ~$50 |
| 임베딩 빌드 (push마다) | 100 노드 = $0.001 (사실상 무료) |

- Cloudflare Pages Functions: **100k req/day 무료** (대부분 한도 안 닿음)
- gpt-4o-mini: $0.15 input / $0.60 output per 1M tokens
- 한국어 답변 더 좋게: `CHAT_MODEL = 'gpt-4o'` (10배 비싸짐)

### 다른 모델로 바꾸기

`functions/api/chat.ts` 상단 상수:
- `EMBED_MODEL = 'text-embedding-3-small'` → `'text-embedding-3-large'` (3072 dim, 더 정확하지만 비용 ↑)
- `CHAT_MODEL = 'gpt-4o-mini'` → `'gpt-4o'` (답변 품질 ↑, 10배 비쌈) 또는 `'gpt-4-turbo'` 등

Anthropic Claude로 답변 가고 싶으면 chat.ts의 OpenAI Chat Completions 블록을 Anthropic Messages API로 교체 + `ANTHROPIC_API_KEY` 환경변수 추가.

### 한국어 임베딩 품질

OpenAI text-embedding-3 > Cohere Embed Multilingual > Cloudflare BGE-M3 (이론적).

### Chat UI 동작 (이미 포함됨)

`viewer/components/ChatPanel.tsx`가 그래프 상단 floating 검색바로 렌더됨. 흐름:

1. 사용자가 상단 검색바에 질문 입력 → Enter
2. `/api/chat` POST → SSE 스트리밍 시작
3. `event: cited` → 인용 노드 5개를 칩으로 답변 상단에 표시 + 그래프에서 흰색 강조 (나머지 dim)
4. `data: {choices:[{delta:...}]}` → 답변이 토큰별로 스트리밍
5. `[node-id]` 인용은 `AnswerRenderer`가 자동 링크화 → 클릭 시 우측 NodePanel에 본문 표시
6. `data: [DONE]` → 종료, 커서 사라짐
7. ✕ 버튼으로 답변 닫기 (그래프 강조도 풀림)

UX 인스피레이션: [careerhackeralex/memory](https://careerhackeralex.vercel.app/memory).

#### 커스터마이즈 포인트

| 변경하고 싶다면 | 파일 |
|---|---|
| placeholder, 색, 폭, 위치 | `viewer/components/ChatPanel.tsx` |
| markdown 렌더, 인용 링크 스타일 | `viewer/components/AnswerRenderer.tsx` |
| 인용 노드 그래프 강조 색/방식 | `viewer/components/ForceGraphCanvas.tsx` `highlightSet` |
| ChatPanel 위치 (floating → 별도 페이지 등) | `viewer/app/ClientGraphPage.tsx` |
| SSE 파싱 (다른 LLM provider로 교체 시) | `viewer/lib/parse-sse.ts` |

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
