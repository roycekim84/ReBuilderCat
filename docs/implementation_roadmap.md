# 냥냥 유적 복원단 구현 로드맵

## 1. 문서 목적

이 문서는 **냥냥 유적 복원단(ReBuilderCat)**을 Flutter로 구현하기 위한 단계별 로드맵이다.

목표는 Codex 또는 개발자가 한 번에 모든 기능을 만들려고 하지 않고, 작은 단위로 완성 가능한 MVP를 쌓아가게 하는 것이다.

---

## 2. 최종 MVP 목표

MVP 완료 시 다음 흐름이 가능해야 한다.

```text
앱 실행
→ 월드맵 표시
→ 유적 선택
→ 유적 상세에서 복원 부위 선택
→ 복원하기 성공/실패 판정
→ 완료하기로 보상 획득
→ 유적 복원도 증가
→ 역사 카드 해금
→ 다음 유적 해금
→ 도감에서 카드 확인
→ 상점에서 무료 보급품 받기
```

---

## 3. 기술 고정

- Flutter
- Dart
- Riverpod
- Hive
- Android 우선
- 순수 Flutter UI
- Flame 보류
- AdMob/IAP는 mock service 먼저 구현

---

## 4. Phase 0. 저장소/문서 확인

### 목표

기존 문서를 읽고 구현 범위를 이해한다.

### 확인 문서

- `docs/concept_design.md`
- `docs/detailed_game_design.md`
- `docs/screen_design.md`
- `docs/game_system.md`
- `docs/level_design.md`
- `docs/asset_image_plan.md`
- `AGENTS.md`

### 완료 기준

- 문서 기준의 MVP 범위를 이해했다.
- 임의로 기능을 늘리지 않는다.

---

## 5. Phase 1. Flutter 프로젝트 초기화

### 목표

실행 가능한 Flutter 앱 뼈대를 만든다.

### 작업 목록

1. Flutter 프로젝트 생성
2. 기본 `main.dart` 구성
3. 앱 이름 설정
4. Riverpod 설정
5. 기본 theme 구성
6. 하단 네비게이션 구성
7. 5개 화면 placeholder 생성

### 생성 화면

- WorldMapScreen
- RelicDetailScreen
- RestorationScreen
- CodexScreen
- ShopScreen

### 완료 기준

- 앱이 정상 실행된다.
- 5개 화면 간 이동이 가능하다.
- 아직 게임 로직은 없어도 된다.

### 권장 커밋

```text
chore: initialize Flutter project
feat: add base navigation screens
```

---

## 6. Phase 2. 데이터 모델 작성

### 목표

게임의 기본 데이터를 표현할 모델을 만든다.

### 작업 목록

1. Region 모델 작성
2. Relic 모델 작성
3. RelicPart 모델 작성
4. HistoryCard 모델 작성
5. PlayerSave 모델 작성
6. RelicProgress 모델 작성
7. RelicPartProgress 모델 작성
8. InventoryItem 모델 작성

### 구현 기준

- 불변 객체 스타일 권장
- `copyWith` 제공 권장
- Hive 저장을 고려한 구조
- 초기에는 단순 class로 시작 가능

### 완료 기준

- seed data를 모델로 표현할 수 있다.
- 저장 데이터 구조를 만들 수 있다.

### 권장 커밋

```text
feat: add core game data models
```

---

## 7. Phase 3. Seed Data 작성

### 목표

초기 유적 6개와 역사 카드 데이터를 코드에 넣는다.

### 기준 문서

- `docs/level_design.md`

### 작업 파일

```text
lib/data/seed/seed_regions.dart
lib/data/seed/seed_relics.dart
lib/data/seed/seed_relic_parts.dart
lib/data/seed/seed_history_cards.dart
lib/data/seed/seed_balance.dart
```

### 초기 유적

1. 작은 항구 사당
2. 붉은 벽돌 왕국
3. 안개의 계단 사원
4. 숲속 고대 사원
5. 정글의 붉은 탑
6. 바위 왕국 유적

### 완료 기준

- 월드맵에서 초기 유적 목록을 읽을 수 있다.
- 각 유적의 부위 목록을 읽을 수 있다.
- 각 유적의 역사 카드 데이터를 읽을 수 있다.

### 권장 커밋

```text
feat: add initial relic seed data
```

---

## 8. Phase 4. Hive 저장 구조 구현

### 목표

플레이어 진행도를 로컬에 저장한다.

### 작업 목록

1. Hive 초기화
2. player save box 생성
3. 신규 저장 데이터 생성 로직
4. 저장 데이터 로드 로직
5. 저장 데이터 업데이트 로직
6. 앱 재실행 후 유지 확인

### 저장 대상

- 골드
- 복원 재료
- 광고 제거 여부
- 유적별 진행도
- 부위별 진행도
- 열린 역사 카드
- 일일 보급품 수령 날짜

### 완료 기준

- 앱 첫 실행 시 저장 데이터가 생성된다.
- 앱 재실행 시 데이터가 유지된다.

### 권장 커밋

```text
feat: add Hive save system
```

---

## 9. Phase 5. Repository / Controller 구조 작성

### 목표

UI와 게임 로직을 분리한다.

### 작업 목록

1. GameRepository 작성
2. SaveRepository 작성
3. WorldMapController 작성
4. RelicDetailController 작성
5. RestorationController 작성
6. CodexController 작성
7. ShopController 작성

### 원칙

- UI 위젯 안에 복원 판정 로직을 직접 넣지 않는다.
- 상태 변경은 controller/repository에서 처리한다.
- 화면은 상태를 읽고 표시하는 역할에 집중한다.

### 완료 기준

- 각 화면에서 필요한 상태를 Riverpod으로 읽을 수 있다.
- 복원 로직 구현 준비가 끝난다.

### 권장 커밋

```text
feat: add repositories and controllers
```

---

## 10. Phase 6. 월드맵 화면 구현

### 목표

유적 진행도와 해금 상태를 보여주는 월드맵을 만든다.

### 작업 목록

1. 파트 목록 표시
2. 유적 노드 표시
3. 잠김/해금/완료 상태 표시
4. 별 등급 표시
5. 유적 선택 시 상세 화면 이동

### MVP UI

- 실제 지도 이미지 없어도 됨
- 세로 리스트 또는 간단한 노드 경로로 구현 가능
- 고양이 위치 아이콘은 placeholder 가능

### 완료 기준

- 첫 유적은 해금되어 있다.
- 잠긴 유적은 입장할 수 없다.
- 해금된 유적은 상세 화면으로 이동한다.

### 권장 커밋

```text
feat: implement world map screen
```

---

## 11. Phase 7. 유적 상세 화면 구현

### 목표

선택한 유적의 복원 상태와 부위 목록을 보여준다.

### 작업 목록

1. 유적명 표시
2. 전체 복원도 표시
3. 대표 이미지 placeholder 표시
4. 복원 부위 목록 표시
5. 부위별 잠김/진행 가능/완료 상태 표시
6. 복원 시작 버튼 구현

### 완료 기준

- 유저가 복원 가능한 부위를 선택할 수 있다.
- 선택한 부위로 복원 화면에 진입한다.

### 권장 커밋

```text
feat: implement relic detail screen
```

---

## 12. Phase 8. 복원 핵심 루프 구현

### 목표

게임의 핵심인 복원하기/완료하기 루프를 구현한다.

### 작업 목록

1. 복원 세션 생성
2. 복원 재료 확인
3. 복원 재료 차감
4. 성공률 계산
5. 랜덤 판정
6. 성공 처리
7. 실패/균열 처리
8. 완료하기 처리
9. 보상 지급
10. 저장 처리

### 기준 문서

- `docs/game_system.md`
- `docs/detailed_game_design.md`

### 완료 기준

- 복원하기 버튼이 실제로 성공/실패 판정을 한다.
- 성공 시 단계와 보상이 증가한다.
- 실패 시 균열 또는 보상 감소가 적용된다.
- 완료하기 시 골드가 지급된다.
- 진행도가 저장된다.

### 권장 커밋

```text
feat: implement restoration core loop
```

---

## 13. Phase 9. 유적 클리어와 해금 구현

### 목표

복원도에 따라 별 등급을 계산하고 다음 유적을 해금한다.

### 작업 목록

1. 부위별 완료 단계 저장
2. 유적 전체 복원도 계산
3. 1성/2성/3성 계산
4. 60% 이상이면 클리어 처리
5. 다음 유적 해금
6. 유적 완료 팝업 표시

### 완료 기준

- 작은 항구 사당을 클리어하면 붉은 벽돌 왕국이 열린다.
- 모든 유적이 순서대로 해금된다.
- 별 등급이 월드맵에 반영된다.

### 권장 커밋

```text
feat: unlock relics based on restoration progress
```

---

## 14. Phase 10. 도감/역사 카드 구현

### 목표

유적 클리어 시 역사 카드가 해금되고 도감에서 확인할 수 있게 한다.

### 작업 목록

1. HistoryCard seed data 연결
2. 유적 60% 이상 복원 시 카드 해금
3. CodexScreen 카드 목록 표시
4. 잠긴 카드 표시
5. 카드 상세 화면 또는 상세 패널 구현
6. 100% 복원 시 완전 복원 표시

### 완료 기준

- 유적 클리어 후 도감에 카드가 열린다.
- 잠긴 카드는 힌트만 보인다.

### 권장 커밋

```text
feat: add codex and history card unlocks
```

---

## 15. Phase 11. 상점/보급품 구현

### 목표

플레이가 막히지 않도록 재료 지급 구조를 만든다.

### 작업 목록

1. ShopScreen UI 구현
2. 무료 일일 보급품 구현
3. 광고 보급품 mock 구현
4. 광고 제거 mock 구매 구현
5. 보유 골드/재료 표시

### 완료 기준

- 하루 1회 무료 보급품을 받을 수 있다.
- mock 광고 보급품으로 재료를 받을 수 있다.
- mock 광고 제거 상태가 저장된다.

### 권장 커밋

```text
feat: implement shop and supply rewards
```

---

## 16. Phase 12. 광고/IAP 서비스 인터페이스 구현

### 목표

실제 광고/IAP 연동 전에 서비스 구조를 분리한다.

### 작업 목록

1. AdsService 인터페이스 작성
2. MockAdsService 작성
3. IapService 인터페이스 작성
4. MockIapService 작성
5. Provider 연결
6. 광고 제거 구매자 처리 분기

### 완료 기준

- 실제 SDK 없이도 광고 보상 플로우 테스트 가능
- 나중에 실제 AdMob/IAP로 교체 가능

### 권장 커밋

```text
refactor: add ads and iap service interfaces
```

---

## 17. Phase 13. UI 손맛 개선

### 목표

기능이 아니라 게임다운 느낌을 만든다.

### 작업 목록

1. 복원 성공 시 반짝 효과
2. 실패 시 흔들림 효과
3. 보상 숫자 증가 애니메이션
4. 균열 상태 시각화
5. 고양이 상태 placeholder 표시
6. 유적 단계별 placeholder 변경
7. 버튼 눌림 피드백

### 완료 기준

- 복원하기 버튼을 누를 때 즉각적인 반응이 있다.
- 성공/실패가 텍스트만이 아니라 시각적으로 느껴진다.

### 권장 커밋

```text
feat: improve restoration feedback animations
```

---

## 18. Phase 14. 실제 AdMob 연동

### 목표

보상형 광고를 실제로 연결한다.

### 작업 목록

1. google_mobile_ads 패키지 추가
2. Android 설정
3. 테스트 광고 ID 설정
4. RewardedAd 로드/표시 구현
5. 균열 복구 광고 연결
6. 보급품 광고 연결
7. 광고 로드 실패 처리

### 완료 기준

- 테스트 보상형 광고가 정상 표시된다.
- 광고 시청 완료 후 보상이 지급된다.

### 권장 커밋

```text
feat: integrate rewarded ads
```

---

## 19. Phase 15. 실제 IAP 연동

### 목표

광고 제거 구매를 연결한다.

### 작업 목록

1. in_app_purchase 패키지 설정
2. 상품 id 정의
3. 구매 플로우 구현
4. 구매 성공 시 isAdRemoved 저장
5. 구매 복원 구현
6. 구매 실패/취소 처리

### 완료 기준

- 테스트 구매 또는 샌드박스 구매가 가능하다.
- 구매 복원이 가능하다.

### 권장 커밋

```text
feat: integrate remove ads purchase
```

---

## 20. Phase 16. 출시 전 QA

### 목표

내부 테스트 업로드 가능한 상태로 만든다.

### 체크리스트

- 앱 첫 실행 정상
- 저장 데이터 생성 정상
- 앱 재실행 후 저장 유지
- 6개 유적 해금 흐름 정상
- 복원 성공률 정상
- 실패/균열 정상
- 보상 지급 정상
- 역사 카드 해금 정상
- 보급품 날짜 제한 정상
- 광고 로드 실패 시 앱 멈추지 않음
- 광고 제거 구매자 처리 정상
- 작은 화면에서도 UI 깨지지 않음

### 권장 커밋

```text
test: verify MVP flow
chore: prepare android test build
```

---

## 21. Phase 17. 출시 준비

### 작업 목록

1. 앱 아이콘 제작
2. 스플래시 제작
3. 스토어 스크린샷 준비
4. 개인정보처리방침 준비
5. 광고 ID 설정 확인
6. IAP 상품 등록 확인
7. Android App Bundle 빌드
8. Google Play 내부 테스트 업로드

---

## 22. 개발 중 우선순위

항상 다음 순서를 우선한다.

1. 게임 루프 동작
2. 저장 안정성
3. 화면 이동 명확성
4. 성공/실패 피드백
5. 도감/해금 보상
6. 광고/IAP
7. 아트/사운드 고도화

---

## 23. 구현 보류 항목

아래 기능은 MVP 이후에 작업한다.

- 고양이 꾸미기
- 미션/업적
- 추가 파트
- Flame 엔진
- 고급 파티클
- 사운드 전체 적용
- 클라우드 저장
- 랭킹
- 이벤트 시스템

---

## 24. 첫 Codex 작업 프롬프트 예시

```text
이 저장소는 Flutter 모바일 게임 "냥냥 유적 복원단(ReBuilderCat)" 프로젝트다.
먼저 docs 폴더의 기획 문서와 AGENTS.md를 모두 읽고, MVP 구현 범위를 파악해라.

첫 작업 목표는 예쁜 최종 게임이 아니라, Flutter 프로젝트 초기화와 5개 화면 라우팅, Riverpod/Hive 기반의 기본 구조를 만드는 것이다.

요구사항:
1. Flutter 프로젝트를 현재 저장소에 생성한다.
2. Riverpod을 적용한다.
3. Hive 초기화 구조를 준비한다.
4. WorldMapScreen, RelicDetailScreen, RestorationScreen, CodexScreen, ShopScreen 5개 화면을 만든다.
5. 하단 네비게이션 또는 기본 라우팅으로 화면 이동이 가능하게 한다.
6. 아직 실제 게임 로직은 구현하지 말고, placeholder UI와 폴더 구조를 AGENTS.md 기준으로 만든다.
7. 빌드 오류가 없게 한다.

작업 후 변경 파일과 다음 단계 제안을 요약해라.
```

---

## 25. 최종 로드맵 정리

이 프로젝트는 한 번에 완성하려고 하면 커진다. 반드시 다음 순서로 진행한다.

```text
화면 뼈대
→ 데이터 모델
→ seed data
→ 저장
→ 월드맵/상세
→ 복원 루프
→ 도감/해금
→ 상점/mock BM
→ 손맛 개선
→ 실제 광고/IAP
→ 출시 준비
```

이 순서를 지키면 혼자서도 출시 가능한 크기로 유지할 수 있다.
