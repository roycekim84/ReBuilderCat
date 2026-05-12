# AGENTS.md - 냥냥 유적 복원단(ReBuilderCat) 구현 가이드

## 1. 프로젝트 개요

이 프로젝트는 **냥냥 유적 복원단(ReBuilderCat)**이라는 모바일 캐주얼 게임이다.

핵심 컨셉:

> 고양이 복원가가 세계의 고대 유적을 여행하며, 확률 복원으로 유적을 하나씩 되살리는 캐주얼 탐험 게임.

게임의 핵심 재미는 다음 선택에 있다.

> 복원할까, 지금 완료할까?

유저는 복원 성공률, 균열 위험, 현재 보상, 다음 성공 보상을 보고 한 번 더 복원할지 결정한다.

---

## 2. 반드시 참고해야 할 문서

구현 전에 아래 문서를 먼저 읽고 기준으로 삼는다.

1. `docs/concept_design.md`  
   전체 컨셉, 방향성, MVP 범위, BM, 아트 방향

2. `docs/screen_design.md`  
   5개 핵심 화면 구성과 UX 흐름

3. `docs/game_system.md`  
   복원 확률, 균열, 보상, 저장 데이터, 광고/IAP 연결 지점

문서와 충돌하는 구현을 하지 않는다. 기능 추가가 필요해 보이면 먼저 문서 기준의 MVP 범위를 우선한다.

---

## 3. 기술 스택

### 3.1 확정 스택

- Framework: Flutter
- Language: Dart
- State Management: Riverpod
- Local Storage: Hive
- Ads: Google AdMob, 초기에는 mock service로 분리
- IAP: in_app_purchase, 초기에는 mock service로 분리
- Target: Android 우선

### 3.2 MVP에서는 사용하지 않을 것

- Flame 엔진
- Firebase
- 로그인
- 클라우드 저장
- 온라인 랭킹
- 서버 API
- 복잡한 장비/스킬 시스템
- 다중 재화 시스템

MVP는 순수 Flutter UI와 간단한 애니메이션으로 구현한다.

---

## 4. 개발 원칙

### 4.1 작게 만들기

이 게임은 화면과 시스템이 커지면 완성 가능성이 낮아진다.

우선순위는 다음과 같다.

1. 복원 버튼의 손맛
2. 성공/실패 피드백
3. 유적 진행도 저장
4. 월드맵 해금
5. 역사 카드 해금
6. 보급품/광고/IAP 연결

### 4.2 단순한 UX 유지

모든 화면은 유저가 다음 행동을 바로 알 수 있어야 한다.

핵심 버튼:

- 복원하기
- 완료하기
- 입장하기
- 복원 시작
- 보급품 받기

### 4.3 실패는 가볍게

실패 표현은 “파괴”가 아니라 “균열 발생”이다.

금지:

- 잦은 완전 파괴
- 잦은 완전 초기화
- 실패할 때마다 강제 광고

### 4.4 광고는 보상형 중심

MVP 광고는 실제 SDK 연결 전에도 인터페이스로 분리한다.

초기 광고 포인트:

- 균열 복구
- 보급품 받기

전면 광고는 보류하거나 매우 낮은 빈도로만 고려한다.

---

## 5. 추천 프로젝트 구조

```text
lib/
  main.dart
  app/
    app.dart
    router.dart
    theme.dart
  core/
    constants/
      app_constants.dart
      balance_constants.dart
    utils/
      random_utils.dart
      date_utils.dart
    storage/
      hive_boxes.dart
      hive_init.dart
  data/
    models/
      region.dart
      relic.dart
      relic_part.dart
      history_card.dart
      player_save.dart
      relic_progress.dart
      inventory_item.dart
    seed/
      seed_regions.dart
      seed_relics.dart
      seed_history_cards.dart
      seed_balance.dart
    repositories/
      game_repository.dart
      save_repository.dart
  services/
    ads/
      ads_service.dart
      mock_ads_service.dart
    iap/
      iap_service.dart
      mock_iap_service.dart
  features/
    world_map/
      world_map_screen.dart
      world_map_controller.dart
      widgets/
    relic_detail/
      relic_detail_screen.dart
      relic_detail_controller.dart
      widgets/
    restoration/
      restoration_screen.dart
      restoration_controller.dart
      restoration_result.dart
      widgets/
    codex/
      codex_screen.dart
      history_card_detail_screen.dart
      codex_controller.dart
      widgets/
    shop/
      shop_screen.dart
      shop_controller.dart
      widgets/
    common/
      widgets/
        app_top_bar.dart
        bottom_nav.dart
        primary_button.dart
        resource_chip.dart
assets/
  images/
    cats/
    relics/
    backgrounds/
    ui/
  audio/
docs/
```

폴더명은 필요에 따라 조정 가능하지만, feature 단위 분리는 유지한다.

---

## 6. 화면 구현 범위

MVP에서 구현할 5개 화면:

1. `WorldMapScreen`
2. `RelicDetailScreen`
3. `RestorationScreen`
4. `CodexScreen`
5. `ShopScreen`

### 6.1 WorldMapScreen

역할:

- 파트/유적 선택
- 유적 잠금/해금 상태 표시
- 별 달성도 표시
- 유적 상세로 이동

### 6.2 RelicDetailScreen

역할:

- 유적 전체 복원도 표시
- 복원 부위 목록 표시
- 복원할 부위 선택
- 복원 화면으로 이동

### 6.3 RestorationScreen

역할:

- 복원하기/완료하기 핵심 플레이
- 성공률, 균열 위험, 현재 보상, 다음 보상 표시
- 성공/실패 애니메이션 또는 최소한의 피드백 표시
- 실패 후 광고 복구 제안

### 6.4 CodexScreen

역할:

- 열린 역사 카드 목록 표시
- 잠긴 카드 힌트 표시
- 카드 상세 보기

### 6.5 ShopScreen

역할:

- 무료 보급품 받기
- 광고 보급품 받기
- 간단한 복원 도구 표시
- 광고 제거 IAP 표시

---

## 7. 핵심 모델 설계 기준

구현 모델은 `docs/game_system.md`를 기준으로 한다.

필수 모델:

- `Region`
- `Relic`
- `RelicPart`
- `HistoryCard`
- `PlayerSave`
- `RelicProgress`
- `RelicPartProgress`
- `InventoryItem`

각 모델은 불변 객체 스타일을 권장한다.

- `copyWith` 제공 권장
- JSON 변환 또는 Hive adapter 준비
- 초기에는 간단 구현 후 Hive 타입 어댑터를 붙여도 됨

---

## 8. 상태관리 기준

Riverpod을 사용한다.

권장 Provider:

- `gameRepositoryProvider`
- `playerSaveProvider`
- `worldMapControllerProvider`
- `relicDetailControllerProvider`
- `restorationControllerProvider`
- `codexControllerProvider`
- `shopControllerProvider`
- `adsServiceProvider`
- `iapServiceProvider`

복원 판정 로직은 UI 위젯 안에 직접 넣지 않는다. 반드시 controller 또는 repository로 분리한다.

---

## 9. 저장 기준

Hive를 사용한다.

초기 구현에서는 다음 박스를 권장한다.

```text
player_save_box
settings_box
```

MVP에서는 단일 `PlayerSave` 객체를 저장하는 방식으로 충분하다.

저장 타이밍:

- 복원 시도 후
- 완료하기 후
- 유적 해금 후
- 역사 카드 해금 후
- 보급품 수령 후
- IAP 구매 복원 후

---

## 10. 복원 판정 구현 기준

복원하기 버튼을 누르면 다음 순서로 처리한다.

1. 재료 확인
2. 재료 차감
3. 성공률 계산
4. 랜덤 판정
5. 성공 또는 실패 적용
6. 저장
7. 결과 반환
8. UI 피드백 출력

복원 결과 타입 예시:

```dart
enum RestorationOutcomeType {
  success,
  crack,
  levelDown,
  materialShortage,
}

class RestorationResult {
  final RestorationOutcomeType type;
  final int previousLevel;
  final int currentLevel;
  final int previousReward;
  final int currentReward;
  final int crackLevel;
  final String message;
}
```

---

## 11. 밸런스 기준

초기 성공률 테이블은 `docs/game_system.md`를 따른다.

처음 구현에서는 `balance_constants.dart` 또는 seed balance 파일에 하드코딩해도 된다.

초기에는 다음 난이도만 사용한다.

- tutorial
- easy
- normal

hard는 데이터 구조만 준비하고 실제 유적에는 나중에 적용한다.

---

## 12. 초기 콘텐츠 기준

MVP 초기 유적:

### Part 0. 작은 시작섬

1. 작은 항구 사당

### Part 1. 아시아의 첫 유적

1. 붉은 벽돌 왕국
2. 안개의 계단 사원
3. 숲속 고대 사원
4. 정글의 붉은 탑

### Part 2. 인도와 남아시아

1. 바위 왕국 유적

총 6개 유적.

각 유적은 3~5개 부위를 가진다.

초기에는 실제 이미지가 없어도 placeholder asset 또는 Flutter Container 기반 임시 UI를 사용한다.

---

## 13. UI 구현 기준

### 13.1 임시 아트 허용

초기 구현에서는 다음을 허용한다.

- 색상 박스
- 아이콘
- 간단한 도형
- 텍스트 기반 유적 표시
- placeholder 이미지 경로

단, asset 경로 구조는 미리 잡아둔다.

### 13.2 애니메이션 최소 기준

MVP에서 최소로 필요한 애니메이션:

- 복원 성공 시 숫자 증가 강조
- 실패 시 흔들림 또는 균열 표시
- 유적 완료 시 반짝 효과
- 월드맵 해금 시 노드 강조

Flutter 기본 애니메이션으로 구현한다.

---

## 14. 광고 서비스 설계

실제 AdMob 연결 전에는 `MockAdsService`를 사용한다.

인터페이스 예시:

```dart
abstract class AdsService {
  Future<bool> showRewardedAd({required RewardedAdPlacement placement});
  Future<bool> canShowRewardedAd(RewardedAdPlacement placement);
}

enum RewardedAdPlacement {
  repairCrack,
  supplyBox,
}
```

광고 제거 구매자가 있으면 광고 없이 true를 반환하는 처리는 상위 서비스 또는 shop controller에서 담당한다.

---

## 15. IAP 서비스 설계

초기에는 `MockIapService`를 사용한다.

인터페이스 예시:

```dart
abstract class IapService {
  Future<bool> purchaseRemoveAds();
  Future<bool> restorePurchases();
}
```

실제 `in_app_purchase` 연결은 MVP 기본 루프 완성 후 진행한다.

---

## 16. 라우팅 기준

초기에는 Flutter 기본 Navigator 또는 go_router를 사용할 수 있다.

프로젝트가 작으므로 둘 중 하나를 선택한다.

권장:

- 단순 Navigator 2.0 없이 기본 Navigator로 시작 가능
- 화면이 늘어나면 go_router 도입 가능

라우팅은 `app/router.dart`에 모은다.

---

## 17. 구현 로드맵

### Phase 1. 프로젝트 초기화

- Flutter 프로젝트 생성
- Riverpod 설정
- Hive 초기화
- 기본 Theme 구성
- 하단 네비게이션 구성
- 5개 빈 화면 라우팅

완료 기준:

- 앱 실행 가능
- 5개 화면 이동 가능

---

### Phase 2. 데이터 모델과 seed 구성

- Region 모델
- Relic 모델
- RelicPart 모델
- HistoryCard 모델
- PlayerSave 모델
- 초기 seed 데이터 작성
- GameRepository 작성

완료 기준:

- 앱에서 초기 유적 목록을 읽을 수 있음
- 기본 저장 데이터가 생성됨

---

### Phase 3. 월드맵/유적 상세 구현

- WorldMapScreen UI 구현
- 유적 잠금/해금 상태 표시
- RelicDetailScreen UI 구현
- 복원 부위 목록 표시
- 복원 화면 진입 구현

완료 기준:

- 유저가 유적을 선택하고 복원 부위를 선택할 수 있음

---

### Phase 4. 복원 핵심 루프 구현

- RestorationController 작성
- 복원 성공률 계산
- 재료 소모
- 성공/실패 판정
- 균열 처리
- 완료하기 보상 지급
- 저장 처리

완료 기준:

- 복원하기/완료하기 루프가 정상 작동
- 앱 재실행 후 진행도 유지

---

### Phase 5. 유적 클리어/도감 구현

- 유적 복원도 계산
- 60/80/100% 기준 처리
- 다음 유적 해금
- HistoryCard 해금
- CodexScreen 구현

완료 기준:

- 유적 클리어 시 카드가 열림
- 월드맵 다음 유적이 열림

---

### Phase 6. 상점/보급품 구현

- ShopScreen 구현
- 무료 일일 보급품
- 광고 보급품 mock
- 균열 복구 mock 광고 처리
- 광고 제거 mock 구매 처리

완료 기준:

- 보급품을 받을 수 있음
- 광고 제거 상태가 저장됨

---

### Phase 7. UI 손맛 개선

- 성공/실패 피드백 강화
- 숫자 변화 애니메이션
- 고양이 상태 placeholder 추가
- 유적 단계별 placeholder 이미지 변경
- 사운드 연결 준비

완료 기준:

- 버튼을 누를 때 게임다운 반응이 있음

---

### Phase 8. 실제 광고/IAP 연동

- AdMob 설정
- 보상형 광고 연결
- in_app_purchase 설정
- 광고 제거 상품 연결
- 구매 복원 구현

완료 기준:

- 테스트 광고 동작
- 테스트 구매 또는 스토어 샌드박스 동작

---

### Phase 9. 출시 준비

- 앱 아이콘
- 스플래시
- 개인정보처리방침 링크 준비
- 광고/IAP 스토어 설정
- Android 빌드 설정
- QA 체크

완료 기준:

- Google Play 내부 테스트 업로드 가능

---

## 18. 금지사항

MVP 중에는 아래를 하지 않는다.

- 화면 추가 남발
- 장비 강화 시스템 추가
- 캐릭터 레벨 시스템 추가
- 온라인 랭킹 추가
- 서버 저장 추가
- 스토리 대사 대량 추가
- 재화 3개 이상 추가
- 전면 광고 과다 삽입
- 실패 시 완전 파괴 중심 밸런스
- 실제 유적 사진/로고/공식 이미지 무단 사용

---

## 19. 커밋 기준

작업 단위별로 작은 커밋을 권장한다.

커밋 메시지 예시:

```text
chore: initialize Flutter project
feat: add seed relic data
feat: implement restoration attempt logic
feat: add world map screen
feat: unlock history card on relic clear
refactor: separate ads service interface
```

---

## 20. 테스트 기준

최소 테스트 대상:

- 성공률 계산
- 복원 성공 처리
- 복원 실패 처리
- 균열 패널티
- 완료 보상 계산
- 유적 복원도 계산
- 별 등급 계산
- 다음 유적 해금
- 역사 카드 해금
- 일일 보급품 날짜 처리

테스트가 어렵다면 최소한 핵심 계산 함수는 순수 함수로 분리한다.

---

## 21. Codex 작업 시 첫 목표

처음 작업 목표는 “예쁜 게임”이 아니라 “플레이 가능한 뼈대”다.

첫 번째 구현 목표:

> Flutter 앱을 생성하고, 5개 화면 이동과 seed 데이터 기반 월드맵/복원 루프가 동작하는 MVP 뼈대를 만든다.

아트와 사운드는 placeholder로 시작한다.

---

## 22. 최종 구현 방향

냥냥 유적 복원단은 작고 단단해야 한다.

이 게임이 성공하려면 복잡한 기능보다 다음 세 가지가 먼저 완성되어야 한다.

1. 복원 버튼을 누르는 재미
2. 유적이 조금씩 복원되는 만족감
3. 고양이와 세계를 여행하는 귀여운 분위기

이 세 가지를 해치지 않는 기능만 추가한다.
