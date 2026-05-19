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
