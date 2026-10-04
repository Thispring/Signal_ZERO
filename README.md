# Signal ZERO

<p align="center">
  <img src="ScreenShot/s1.jpg" width="85%" alt="메인 화면">
</p>

Unity로 제작한 3인칭 슈팅 게임입니다.
이전 프로젝트 **Cord: Marigold**의 전투·스테이지 코드를 바탕으로, 공모전 출품을 위해 아트와 세계관을 새로 구성하고 보스전, 보조 로봇, 튜토리얼 기능을 추가했습니다.

---

## 프로젝트 소개

Signal ZERO는 적을 처치하며 스테이지를 진행하는 3인칭 슈팅 게임입니다.
기존 프로젝트가 일반 적 전투를 반복하는 구조였다면, 이번 프로젝트에서는 드론을 소환하는 중간·최종 보스전과 플레이어를 돕는 보조 로봇, 처음 플레이하는 사람을 위한 튜토리얼을 추가했습니다.

| 항목 | 내용 |
| --- | --- |
| 플랫폼 | PC |
| 개발 엔진 | Unity |
| 개발 언어 | C# |
| 개발 기간 | 2025.08.04 ~ 2025.09.04 |
| 개발 인원 | 총 4명 |
| 팀 구성 | 아트 3명 / 기획·프로그래밍 1명 (본인) |

---

## Gameplay

| 일반 스테이지 | 중간 보스 스테이지 |
| :---: | :---: |
| <img src="ScreenShot/s5.jpg" width="100%" alt="일반 스테이지"> | <img src="ScreenShot/s7.jpg" width="100%" alt="중간 보스 스테이지"> |

게임 플레이 영상은 아래 링크에서 확인할 수 있습니다.

[Signal ZERO Gameplay Video](https://www.youtube.com/watch?v=Flv2JUqmpLE)

---

## Download

게임 실행 파일은 아래 링크에서 다운로드할 수 있습니다.

- [Download for Windows](https://drive.google.com/file/d/1VdHJm5ZFoXLx-FKhUd7o9kDCNno-rHhl/view)
- [Download for macOS](https://drive.google.com/file/d/1Au9_S1u2VYu-R0du6c2_p5_7zrxpcQBt/view)

---

## My Role

기획과 프로그래밍을 혼자 담당했습니다.

- Cord: Marigold의 전투·스테이지·상점 코드를 가져와 이번 프로젝트에 맞게 수정
- 중간·최종 보스전 구현 (드론 소환, 일반 탄·미사일 공격, 중간 보스 순간이동)
- 보조 로봇 3종(가드·보호막·회복)과 상점 업그레이드 연동
- 보호막 적과 보조 로봇의 자동 공격 대상 탐색 구현
- 단계별 튜토리얼과 대사 출력 구현
- 보스 등장 영상, 버스트 연출 등 컷신과 UI 구현

---

## 주요 구현 내용

### 보스 공격 패턴과 드론 소환

보스는 `대기 → 공격 → 대기`를 반복하며, 대기 시간이 시작될 때마다 드론을 무작위 위치에 소환합니다(중간 보스 2~6기, 최종 보스 4~9기).
대기 시간이 끝나면 남은 드론 수에 따라 다음 공격이 정해집니다.

- 드론을 모두 처치하면 이번 공격을 건너뜁니다.
- 남은 드론이 소환 수의 절반 미만이면 발사 횟수와 데미지를 절반으로 줄입니다.
- 공격이 끝날 때마다 시퀀스 값이 1씩 늘어나고, 이후 소환되는 드론과 보스 미사일의 체력이 그만큼 증가합니다.

중간 보스는 미사일과 일반 탄 공격을 번갈아 사용하고, 체력이 90·70·40·20%가 되면 좌우로, 60·10%가 되면 원래 위치로 순간이동합니다. 최종 보스는 미사일 공격만 사용하며, 남은 드론 비율에 따라 데미지 배율(2배/4배)이 달라집니다.
미사일은 목표 위치를 향해 포물선으로 날아가며, 플레이어가 사격으로 격추할 수 있습니다.

공격 실행은 `BossBehaviorSystem`의 코루틴이 담당하고, 공격이 끝나면 `OnMissileAttackFinished` / `OnNormalAttackFinished` 이벤트로 `BossFSM`에 알려 다음 대기를 시작합니다. 드론 처치도 `BossDroneManager.OnDroneDestroyed` 이벤트로 전달하며, 구독한 이벤트는 `OnDestroy`에서 해제합니다.

**관련 코드** [BossFSM.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Boss/BossFSM.cs) · [BossBehaviorSystem.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Boss/BossBehaviorSystem.cs) · [BossDroneManager.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Boss/BossDroneManager.cs) · [BossMissile.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Boss/BossMissile.cs)

---

### 보조 로봇

플레이어 옆에서 지원하는 보조 로봇을 구현했습니다. 상점에서 타입을 고를 수 있고, D 키로 스킬을 사용합니다. 스킬 지속 시간이 끝나면 쿨타임이 시작됩니다.

| 타입 | 동작 |
| --- | --- |
| 가드 | 지속 시간 동안 적의 공격 대상을 플레이어에서 보조 로봇으로 바꿉니다. 가드 중 새로 소환된 적도 같은 대상을 공격합니다. |
| 보호막 | 지속 시간 동안 플레이어가 피해를 받지 않습니다. 미사일과 부딪히면 보호막이 즉시 해제됩니다. |
| 회복 | 지속 시간 동안 1초마다 플레이어 체력을 회복합니다. |

가드 발동·해제는 `SupportBotFSM`의 정적 이벤트로 알리고, 각 적의 `EnemyFSM`이 이를 구독해 공격 대상을 바꿉니다.
상점 업그레이드로 지속 시간과 수치가 늘어나며, 업그레이드 레벨이 3 이상이면 스킬 사용 시 무기를 즉시 재장전하고, 6 이상이면 일정 시간 동안 적을 자동 공격합니다.

**관련 코드** [SupportBotFSM.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/SupportBot/SupportBotFSM.cs) · [SupportBotStatusManager.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/SupportBot/SupportBotStatusManager.cs) · [EnemyFSM.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Enemy/EnemyFSM.cs)

---

### 적 탐색과 보호막 적

보호막 적이 필드에 있는 동안 보호 대상 적(`EnemyShield`)에 보호막 이펙트를 표시하고, 레이어를 `Ignore Raycast`로 바꿔 플레이어의 Raycast 사격에 맞지 않도록 했습니다. 보호막 적이 사라지면 레이어를 원래대로 되돌립니다.

보조 로봇의 자동 공격은 이 보호막 적을 가장 먼저 노리도록 했습니다. `SearchEnemyManager`는 적의 소환·처치 이벤트를 받아 활성화된 적 목록을 관리하고, 거리 제곱(`sqrMagnitude`)으로 가장 가까운 적을 찾습니다.
자동 공격은 `가장 가까운 보호막 적 → 보스 → 가장 가까운 일반 적` 순서로 대상을 정합니다. 일반 적과 보호막 적에게는 별도의 데미지 함수(`SupportBotTakeDamage`)를 사용해, 보조 로봇의 공격으로는 플레이어의 버스트 게이지가 차지 않도록 했습니다.

**관련 코드** [SearchEnemyManager.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/GameManager/SearchEnemyManager.cs) · [SupportBotFSM.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/SupportBot/SupportBotFSM.cs)

---

### 튜토리얼

`TutorialStep` 추상 클래스(`Enter` / `Execute` / `Exit` / `Skip`)를 상속해 단계별 클래스를 만들고, `TutorialController`가 Inspector에 등록된 단계 목록을 순서대로 실행합니다.
각 단계는 적 3마리 처치, 버스트 사용, 스킬 키 입력, 상점 이용 등 자신의 조건을 확인한 뒤 컨트롤러에 다음 단계 진행을 요청합니다. 대사는 `DialogSystem`으로 출력합니다.

튜토리얼이 끝나면 남은 튜토리얼 적을 정리하고, 플레이어 체력과 보조 로봇을 복구한 뒤 1스테이지 적 소환을 시작합니다.

**관련 코드** [TutorialController.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Tutorial/TutorialController.cs) · [TutorialStep.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Tutorial/TutorialStep.cs) · [TutorialStep_WaitForSkillUse.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Tutorial/TutorialStep_WaitForSkillUse.cs) · [TutorialStep_OpenShop.cs](https://github.com/Thispring/Signal_ZERO/blob/main/Script/Tutorial/TutorialStep_OpenShop.cs)

---

> 본 리포지토리는 포트폴리오 공개를 목적으로 프로젝트의 스크립트 코드만 포함하고 있습니다.
>
> 게임 에셋 및 전체 Unity 프로젝트 파일은 포함되어 있지 않습니다.
