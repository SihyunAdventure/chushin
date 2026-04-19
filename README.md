# 추신 (Chushin)

> 쿠팡 뷰티 12개 스킨케어 카테고리에서 **ml당/g당 가성비** 상위 상품을 조회하는 Claude Skill.

Claude Code 에서 "3만원 이하 수분크림 가성비 추천" 같은 자연어로 물어보면, 이 Skill 이 자동으로 활성화돼서 쿠팡 실시간 데이터를 ml당/g당 가격으로 정규화한 순위표를 돌려준다.

## 빠른 설치 (Claude Code 기준)

```bash
mkdir -p ~/.claude/skills/chushin
curl -fsSL https://raw.githubusercontent.com/SihyunAdventure/chushin/main/SKILL.md \
  -o ~/.claude/skills/chushin/SKILL.md
```

Claude Code 재시작하면 끝. 

## 써보기

Claude Code 세션에서 아무렇게나 물어봐라:

- "3만원 이하 수분크림 가성비 5개"
- "지금 할인 제일 많이 하는 선크림"
- "로켓배송 되는 세럼 중 100ml당 가장 싼 거"
- "에센스 top 3, 1만원 아래"

Skill 이 자동으로 카테고리/가격범위/정렬모드/로켓필터 를 추출해서 API 에 쿼리하고, 한국어 마크다운 표로 정리해준다.

## 지원 카테고리 (12개)

클렌징 오일 · 클렌징 폼 · 클렌징 워터 · 토너/스킨 · 미스트 · 에센스 · 세럼/앰플 · 아이크림 · 로션 · 수분크림 · 일반/영양크림 · 선크림

## 왜 "가성비" 인가

쿠팡 상품마다 표시 단위가 제각각 (`10ml당 X원` vs `100ml당 Y원` vs `1개당 Z원`) 이라 숫자만 보고 비교하면 **순위가 왜곡됨**.
- 예: `10ml당 440원` 상품이 `100ml당 4,373원` 상품보다 "싸보이지만" 실제로는 100ml 환산 시 4,400원 vs 4,373원 → 후자가 더 쌈.

이 Skill 은 백엔드에서 정규화해서 **100ml/100g 기준 통일 가격** 을 돌려준다. 숫자 크기만 봐도 바로 비교 가능.

## MVP 한계 (솔직하게)

- ❌ **평점/리뷰 기반 추천** — 쿠팡 크롤러가 평점 수집을 아직 안 함. "가성비" 만 근거.
- ❌ **30일/90일 최저가 판단** — 현재 3일치 관측 데이터만 있음. "진짜 최저가 타이밍" 조언 못 함.
- ❌ **정가 조작 감지** — 쿠팡이 표시한 정가·할인율 그대로 씀. 할인율 80%+ 표시된 건 의심 여지 있음 (가이드에는 명시함).
- ❌ **Claude.ai 웹/데스크탑 미지원** — curl 도구가 필요해서 **Claude Code 전용**.
- ❌ **쿠팡 외 채널** — 올리브영/화해 통합은 v0.2 이후.

## 설치 확인

```bash
curl -s 'https://chushin.sihyun.workers.dev/v1/search?category=moisture-cream&limit=3' | head -c 300
```

JSON 응답 (`results` 배열 포함) 이 오면 성공.

## 동작 원리

```
[사용자 쿼리]
     ↓
[Claude Code + chushin Skill]
     ↓ curl
[chushin Worker on Cloudflare edge]
     ↓
[Neon Postgres, read-only view]
     ↓
[hotinbeauty 크롤러가 수집한 쿠팡 스킨케어 카탈로그]
```

- **API:** `https://chushin.sihyun.workers.dev/v1/search`
- **무료, 계정 불필요.** 호출당 응답 시간 100ms~2초 (Cloudflare edge 캐시 24h).
- **Rate limit:** 분당 60회 (per-IP, soft). 친구끼리 쓰기에 충분.

## 라이선스

MIT. 자유롭게 가져다 써도 됨. 다만 이 Worker (`chushin.sihyun.workers.dev`) 는 개인 프로젝트라 SLA 없음. 오래 쓸 거면 fork 해서 각자 배포 추천.
