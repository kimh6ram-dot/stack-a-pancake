# Pancake Stack — 게임 스펙 (Burger Stack 클론)

이 게임은 `04_꼬직/260929_stack game`(Burger Stack)의 100% 클론이다. 물리/상태머신/
렌더링/UI 구조·룰은 전부 동일하며, **재료(에셋)와 그에 따른 설정값만 교체**했다.
전체 아키텍처·상태머신·QA 방법론 등 상세 스펙은 원본의
`04_꼬직/260929_stack game/GAME_SPEC_BURGER_STACK.md`를 그대로 참고할 것
(§1~§22 전체가 여기에도 유효).

## 이 클론에서 바뀐 것

- **재료 구성**: 재료가 1종(`pancake`)뿐이다. 버거처럼 여러 재료가 랜덤으로 섞이지
  않고, 매번 같은 팬케이크가 쌓인다. 토핑(수동 완성용)은 `pancake_top`(버터+시럽
  올라간 팬케이크) 1종.
  - `BS.START_INGREDIENT = 'pancake'`
  - `BS.END_INGREDIENT = 'pancake_top'`
  - `BS.REGULAR_INGREDIENTS = ['pancake']`
- **에셋**: `assets/01_pancake/pancake.png`, `assets/01_pancake/pancake_top.png`.
  원본 제공 PNG 중 `Pancake_top.png`는 알파 채널이 전부 255(불투명)였고 체커보드
  무늬가 RGB에 그대로 구워져 있던 파일이었다 — 무채색(R≈G≈B) + 체커보드 밝기대역
  (220~260) 픽셀을 투명 처리해 복구했다(스크립트는 세션 스크래치패드에만 존재,
  저장소에는 없음). 타일 패리티(위치) 기반 판정은 안티앨리어싱 때문에 가장자리에서
  틀렸다 — 무채색+밝기대역만으로 판정하는 쪽이 훨씬 안정적이었다.
- **접촉 비율(INGREDIENT_CONTACT)**: 팬케이크는 원판형이라 bbox 자체가 이미 꽉 찬
  접촉선이라 `{top:0, bottom:1}`(보정 없음)도 gap=0px였지만, 그러면 "낱장이 따로
  노는" 느낌이라 사용자 피드백으로 겹침을 키웠다. 최종값은 config.js 주석 참고
  (`pancake: {top:0.17, bottom:0.88}`, `pancake_top: {top:0, bottom:0.81}`) —
  햄버거처럼 깊게 겹친 시안과 보정 없는 시안의 중간값으로 확정.
- **물리/밸런스 로직은 수정 안 함**: 동적 흔들림(sway)·충돌(collapse)·착지 연속성
  로직은 전부 원본 그대로이며 재료 종류와 무관하게 동작한다(layerCount 기반이라
  재료 교체의 영향을 받지 않음).
- **`pickNextIngredient` 방어 수정**(`src/game/physics.js`): 원본은 재료 풀이
  여러 개라 "직전과 같은 재료 연속 금지"(`maxStreak`) 필터링 후에도 후보가 항상
  남았지만, 팬케이크는 재료가 1종뿐이라 필터링하면 후보가 0개가 되어
  `candidates[undefined]`로 깨지는 버그가 있었다 — 필터링 결과가 빈 배열이면
  필터링을 적용하지 않고 원래 pool을 그대로 쓰도록 방어 코드 추가. **원본
  Burger Stack에는 적용 안 함**(버거는 8종이라 해당 버그가 발생하지 않음).
- **UI 텍스트**: 타이틀 "팬케이크 쌓기", 토핑 버튼 "토핑 올리기"(원본은 "빵 얹기").
  그 외 버튼/헤드라인 문구는 범용적이라 그대로 유지.
- **저장 파일명**: `pancake-stack-YYYYMMDD-HHmmss.png` (원본은 `burger-stack-...`).

## QA

`tools/` 아래 모든 스크립트가 이 클론 기준으로 재료 이름(`pancake`/`pancake_top`)에
맞게 갱신되어 있고, 전부 통과 확인됨(261008):
`qa.js`, `qa-topping.js`, `qa-sway.js`, `qa-sway-highlayer.js`, `qa-floor-resize.js`,
`qa-landing-x-trace.js`, `measure-contact-gaps.js`.
