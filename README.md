# 추신 (Chushin)

**쿠팡 뷰티 스킨케어 가성비 추천** 오픈 API + Claude/GPT/Gemini용 프롬프트.
ml당/g당 정규화된 가격 기준으로 줄 세워서 돌려준다.

---

## 핵심

추신은 두 부분:
1. **오픈 HTTP API** — `https://chushin.sihyun.workers.dev/v1/search`. 누구나 호출 가능.
2. **프롬프트** (`SKILL.md`) — LLM 에게 API 를 어떻게 쓰고 응답을 어떻게 렌더할지 알려줌.

→ **curl 을 호출할 수 있고 자연어를 이해하는 LLM 이면 뭐든 씀**. Claude Code / ChatGPT / Gemini / Cursor / 커스텀 에이전트 전부 OK.

---

## 사용 예시 (어느 LLM 이든)

```
"3만원 이하 수분크림 가성비 5개"
"로켓배송 되는 세럼 중 100ml당 가장 싼 거"
"선크림이랑 세럼 각각 top 3"   ← 각 카테고리 별도 호출
"선크림, 세럼 중에서 top 5"      ← 합쳐서 1회 호출
```

LLM 이 카테고리·가격필터·정렬 추출해서 API 호출 → 결과를 가성비 순위표로 렌더.

---

## 환경별 설치

### 1. Claude Code — 가장 쉬움

```bash
mkdir -p ~/.claude/skills/chushin
curl -fsSL https://raw.githubusercontent.com/SihyunAdventure/chushin/main/SKILL.md \
  -o ~/.claude/skills/chushin/SKILL.md
```

Claude Code 재시작. 자동 로드.

### 2. ChatGPT Custom GPT — 2분 세팅

ChatGPT Plus 필요. [Create a GPT](https://chat.openai.com/gpts/editor) 에서:

1. **Instructions**: 아래 raw URL 의 [`SKILL.md`](https://raw.githubusercontent.com/SihyunAdventure/chushin/main/SKILL.md) 내용을 붙여넣기 (frontmatter `---` 블록은 제거)
2. **Actions**: 이 저장소의 [`openapi.yaml`](./openapi.yaml) 를 Import from URL:
   ```
   https://raw.githubusercontent.com/SihyunAdventure/chushin/main/openapi.yaml
   ```
3. Authentication: **None** (public API)
4. Save → 한국어로 쿼리 테스트

### 3. Gemini / 기타 function-calling LLM

[`SKILL.md`](./SKILL.md) 의 내용을 system instruction 으로 넣고, function/tool 하나 정의:

```python
# 예: Gemini API
tool = {
  "name": "chushin_search",
  "description": "쿠팡 스킨케어 가성비 검색",
  "parameters": {  # same as openapi.yaml schema
    "category": "string (required)",
    "min_price": "integer | null",
    "max_price": "integer | null",
    "rocket": "boolean | null",
    "sort": "unit | price",
    "limit": "1..20"
  }
}
# function 핸들러는 https://chushin.sihyun.workers.dev/v1/search 로 GET
```

OpenAI Assistants API, Cursor, Cline, Anthropic Tool Use 등 같은 패턴.

### 4. LLM 없이 — 순수 curl

```bash
curl -s 'https://chushin.sihyun.workers.dev/v1/search?category=moisture-cream&max_price=30000&limit=5' | jq
```

## 카테고리 (12개)

`cleansing-oil`, `cleansing-foam`, `cleansing-water`, `toner-skin`, `mist`, `essence`, `serum-ampoule`, `eye-cream`, `lotion`, `moisture-cream`, `cream`, `sunscreen`

한글 별명 (`선크림`, `수분크림`, `세럼`, `토너` 등) API 가 자동 매핑.

---

## 동작

쿠팡 뷰티 12개 스킨케어 카테고리를 매일 수집하는 [hotinbeauty](https://hotinbeauty.com) 데이터를 Cloudflare Worker 로 서빙. 단위 혼재 (`10ml당` vs `100ml당` vs `1개당`) 를 `price_per_100` 으로 정규화해 진짜 가성비 비교 가능.

## 한계

- **평점·리뷰 없음** — 크롤러가 수집 안 함. 가성비·가격만 근거
- **3일치 데이터** — "30일 최저가" 같은 장기 판단 불가
- **할인율 기반 추천 없음** (v0.3~) — 쿠팡 원가는 seller-editable 이라 할인 % 가 신뢰할 signal 이 아님

## 라이선스

MIT. SLA 는 없음 — 개인 프로젝트라 언젠가 내려갈 수 있으니 오래 쓸 거면 fork 추천.
