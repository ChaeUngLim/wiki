# 하네스 엔지니어링 위키

gajae(gjc)를 활용해 만든 Harness Engineering 한국어 학습 위키입니다.

- 사이트: https://chaeunglim.github.io/wiki/
- 내용: `content/` (Obsidian vault 원본 `harness-wiki/wiki/`의 사본)
- 원문 출처: [learn-harness-engineering](https://github.com/walkinglabs/learn-harness-engineering), [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode), [gajae-code](https://github.com/Yeachan-Heo/gajae-code) (모두 MIT)
- 사이트 생성: [Quartz](https://quartz.jzhao.xyz/) v5 (MIT, `LICENSE.txt`)

## 다시 빌드하기

```sh
npm ci
npx quartz build      # public/ 생성
```

## 이 사이트는 어떤 원리로 만들어지나

한 줄로: **마크다운 노트를 Quartz라는 "사이트 생성기"가 HTML로 일괄 변환하고, GitHub Pages가 그 파일을 그대로 보여 준다.** 사람이 HTML을 직접 짠 것은 없습니다.

비유하면 원고(마크다운) → 인쇄소(Quartz)가 책(HTML)으로 찍음 → 서점(GitHub Pages)이 진열만 함.

### 전체 흐름 (4단계)

```
① 원문 자료          ② 위키 원고(마크다운)        ③ Quartz 빌드          ④ GitHub Pages
raw/ (강의·README) → harness-wiki/wiki/*.md   →  npx quartz build   →  gh-pages 브랜치
   gjc(AI)가 읽고        53개 .md + [[링크]]        public/ 375개 파일      chaeunglim.github.io/wiki
   요약·정리             wiki-lint로 검사           (HTML·CSS·JS)           (정적 파일 그대로 전송)
```

1. **원고 만들기**: gjc 세션이 `raw/`의 강의·프로젝트·하네스 원문을 읽고, 규칙 파일(`AGENTS.md`)대로 한국어 마크다운 페이지를 썼습니다. 각 파일 위에는 설정 블록(frontmatter: `title`, `aliases`, `tags`)이 있고, 본문에는 `[[concepts/ralph-loop]]` 같은 페이지 간 링크가 있습니다. 규칙 위반은 `wiki-lint.mjs`로 검사했습니다.
2. **복사**: 그 마크다운을 이 저장소의 `content/`로 복사합니다.
3. **변환 (Quartz)**: `npx quartz build`가 마크다운을 하나씩 읽어
   - `unified`·`remark`·`rehype` 계열 라이브러리로 마크다운 → HTML로 바꾸고,
   - `[[링크]]`를 실제 웹 주소로 바꾸고(예: `[[harness/oh-my-claudecode]]` → `../harness/oh-my-claudecode`),
   - `title`은 페이지 제목, `aliases`(별칭)는 별칭 주소로 들어와도 원래 페이지로 넘겨 주는 페이지, `tags`는 태그 모음 페이지(`tags/topic/...`)로 만들고,
   - 모든 페이지의 제목·본문·링크 관계를 `static/contentIndex.json` 한 파일에 모읍니다(그래프 뷰와 검색이 이걸로 동작),
   - 화면 틀(목차·검색창·그래프 칸)은 `preact` 컴포넌트로 그리고, CSS·JS는 `esbuild`·`lightningcss`로 묶어
   - 결과물을 `public/` 폴더에 냅니다(현재 375개 파일, `index.html`이 첫 화면).
4. **배포 (GitHub Pages)**: `public/` 내용을 이 저장소의 `gh-pages` 브랜치에 올립니다. GitHub Pages는 서버에서 아무것도 계산하지 않고, 그 파일을 주소 그대로 내보냅니다. `/wiki/concepts/ralph-loop`에 들어오면 `concepts/ralph-loop.html`을 보내 주는 식입니다.

브라우저에서 움직이는 것(그래프 뷰, 검색, 링크에 마우스를 올리면 뜨는 미리보기, 다크 모드, 페이지 전환)은 모두 함께 내려받은 JS 파일이 **사용자 브라우저 안에서** 처리합니다.

### 이 저장소 `ChaeUngLim/wiki`의 구성

| 위치 | 내용 |
| --- | --- |
| `main` 브랜치 | 원본: `content/`(마크다운 53개) + Quartz 프로그램·설정(`quartz.config.yaml`) + 이 README |
| `gh-pages` 브랜치 | 변환된 완성 사이트(HTML 375개). Settings → Pages가 이 브랜치를 서비스 |
| GitHub의 **Wiki 탭** | 저장소마다 기본으로 켜져 있는 메모장 기능. 이 사이트와는 **별개**이며 사용하지 않음 |

### 위키를 고친 뒤 사이트에 반영하기

```sh
rsync -a --delete --exclude .obsidian --exclude '무제.*' ~/Desktop/harness-wiki/wiki/ content/
npx quartz build
# 그다음 public/ 내용을 gh-pages 브랜치에 다시 올린다
```
