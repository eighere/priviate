# 경제·AI 아침 브리핑 — 클라우드 자동화 세팅 가이드

컴퓨터가 꺼져 있어도 **GitHub 서버**가 매일 아침 8시(한국시간)에 실행해서 **Notion에 브리핑을 올려주는** 구조입니다.

## 구성
- `brief.py` — 마켓 데이터(yfinance) + 뉴스(Google News RSS) + LLM 요약 → Notion 업로드
- `.github/workflows/brief.yml` — 매일 08:00 KST 실행 (UTC 23:00)
- `requirements.txt` — 의존성

## 준비물 (딱 2개 필수)
1. **Notion integration 토큰** (`secret_...`)
2. **LLM API 키** (예: DeepSeek API)

---

## 1단계 — Notion integration 만들기
1. https://www.notion.com/my-integrations 접속
2. **"New integration"** 클릭 → 이름은 아무거나(예: `brief-bot`) → 워크스페이스 선택
3. 생성 후 **"Internal Integration Secret"** 값(`secret_...`)을 복사해 둠 ← 이게 `NOTION_TOKEN`

## 2단계 — 브리핑 DB를 integration에 공유
1. 노션에서 **"경제·AI 아침 브리핑"** DB 페이지를 연다
2. 우상단 **`···` → Connections(연결) → "brief-bot"** 추가
   - (안 보이면 "Add connections"에서 방금 만든 integration 검색)

## 3단계 — LLM API 키 준비
- **DeepSeek** 사용 시: https://platform.deepseek.com → API Keys → 새 키 생성
  - 기본값이 DeepSeek라서 키만 넣으면 됨
- 다른 모델(OpenAI 등)을 쓰려면 `LLM_BASE_URL`, `LLM_MODEL` 시크릿도 추가

## 4단계 — GitHub 저장소 만들기
1. https://github.com/new → **Private** 저장소 생성 (예: `morning-brief`)
2. 이 폴더(`클라우드 브리핑`)의 파일 4개를 저장소 루트에 올린다
   - `brief.py`, `requirements.txt`, `.github/workflows/brief.yml`, (README)
   - (GitHub 웹에서 "Add file → Upload files"로 드래그하면 됨)

## 5단계 — 시크릿 등록
1. 저장소 **Settings → Secrets and variables → Actions → New repository secret**
2. 아래 2개(필수) 등록:
   | Name | Value |
   |---|---|
   | `NOTION_TOKEN` | `secret_...` (1단계 값) |
   | `LLM_API_KEY` | LLM API 키 |
3. (선택) 기본값 바꾸려면: `NOTION_DB_ID`, `LLM_BASE_URL`, `LLM_MODEL`
   - DB ID 기본값: `914fdb862eb24bae9d376ec228a428a5` (브리핑 DB)

## 6단계 — 테스트
1. 저장소 **Actions** 탭 → 왼쪽 `daily-brief` → **Run workflow** → Run workflow(수동 실행)
2. 초록 체크 뜨면 노션 DB에 페이지가 생겼는지 확인

## 주의/한계 (솔직히)
- 마켓 스냅샷은 **yfinance**로 긁어서 채우고, 국고채 3·10년은 자동으로 못 구해서 `-`로 남아.
- 뉴스는 **Google News RSS** 헤드라인 기반이라, 내가 직접 웹서치할 때보다 항목 수/깊이는 약간 얕을 수 있어.
- LLM이 지어내는 걸 막기 위해 "헤드라인에 없는 수치는 만들지 말 것"을 프롬프트에 넣어뒀어.
- 무료 GitHub Actions는 월 실행 제한이 있지만 하루 1회면 충분히 무료 범위야.
- PDF는 이 클라우드판에선 생성 안 하고(로컬 전용), 노션 텍스트 브리핑만 올려. PDF까지 클라우드에서 원하면 알려줘.
