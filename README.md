# 추신 (Chushin)

Claude Code 용 **쿠팡 뷰티 스킨케어 가성비 추천** 스킬.
ml당/g당 정규화된 가격 기준으로 줄 세워서 돌려준다.

## 설치

```bash
mkdir -p ~/.claude/skills/chushin
curl -fsSL https://raw.githubusercontent.com/SihyunAdventure/chushin/main/SKILL.md \
  -o ~/.claude/skills/chushin/SKILL.md
```

Claude Code 재시작.

## 사용 예시

Claude Code 에 아무렇게나 한국어로 물어보면 됨:

- "3만원 이하 수분크림 가성비 5개"
- "지금 할인 많이 하는 선크림 top 10"
- "로켓배송 되는 세럼 중 100ml당 가장 싼 거"
- "에센스, 토너 각각 추천해줘" (최대 3개 카테고리 동시)

Skill 이 카테고리·가격필터·정렬모드 를 알아서 추출한다.

## 동작

쿠팡 뷰티 12개 스킨케어 카테고리를 매일 수집하는 [hotinbeauty](https://hotinbeauty.com) 데이터를 Cloudflare Worker (`chushin.sihyun.workers.dev/v1/search`) 로 서빙. 단위 혼재 (`10ml당` vs `100ml당` vs `1개당`) 를 `price_per_100` 으로 정규화해 진짜 가성비 비교 가능.

## 한계

- **Claude Code 전용** — curl 도구가 필요해 Claude.ai 웹/데스크탑 미지원
- **평점·리뷰 없음** — 크롤러가 수집 안 함. 가성비만 근거
- **3일치 데이터** — "30일 최저가" 같은 장기 판단 불가

## 라이선스

MIT. SLA 는 없음 — 개인 프로젝트라 언젠가 내려갈 수 있으니 오래 쓸 거면 fork 추천.
