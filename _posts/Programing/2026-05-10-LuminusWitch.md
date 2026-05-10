---
title: "LuminusWitch"
categories:
  - Programing
tags:
  - Lists
  - Programing
  - Review
---
# Luminus Witch 

“루미너스 위치”는 마법 조합과 성장 요소를 중심으로 던전을 탐험하는 덱빌딩 로그라이크 게임입니다.
Unity 기반의 4인 팀 프로젝트로 개발했으며, 2025.06 ~ 2026.01 동안 개발을 진행했습니다.
버닝비버 전시 출품을 목표로 프로젝트를 운영하였습니다.

## 프로젝트 정보
- 장르 : 덱빌딩 로그라이크
- 개발 인원 : 4인
- 개발 기간 : 2025.06 ~ 2026.01
- 개발 환경 : Unity / C#
- 플랫폼 : PC
- 목표 : 버닝비버 전시 출품

[**스토브 인디**](https://store.onstove.com/ko/games/103045)

---

## 맡은 역할

- 게임 시스템 구조 설계
- 플레이 로직 및 데이터 구조 구현
- UI/UX 구조 설계 및 개발
- 일정 관리 및 협업 조율
- 구현 방향 정리 및 기획 문서 작성
- 이펙트 디자인

[![소개영상](http://i.ytimg.com/vi/KmP79IpMWQo&t/0.jpg)](https://www.youtube.com/watch?v=KmP79IpMWQo&t)

![screenshot_1](_posts\Programing\Images\LuminusWitch1.png)

![screenshot_2](_posts\Programing\Images\LuminusWitch2.png)

![screenshot_3](_posts\Programing\Images\LuminusWitch3.png)

![screenshot_4](_posts\Programing\Images\LuminusWitch4.png)

## 기술 스택
Engine: Unity
Language: C#
Collaboration: Git, Notion, Figma
Tools: Photoshop, GIMP...

## 경험

### 유지보수성 확보 

덱 빌딩 로그라이크 장르 특성상 반복적으로 사용되는 기능이 많았기 때문에, 재사용성과 확장성을 고려한 구조 설계를 중요하게 생각했습니다. 아이템, 주문, 유물, 버프, 맵 기믹 등의 콘텐츠를 추가할 때마다 새로운 코드를 반복 작성하기보다, 데이터 기반으로 동작을 정의할 수 있도록 시스템을 설계했습니다.

특히 주문 시스템에서는 SkillData와 SkillBehavior를 분리하여 구현했습니다. SkillData에서는 쿨타임, 사용 가능 상태 등 공통 데이터를 관리하고, SkillBehavior에서는 각 스킬의 실제 동작과 개별 스탯(예: 대시 거리)을 담당하도록 역할을 분리했습니다. 이를 통해 기능 간 중복을 줄이고, 새로운 스킬을 보다 빠르게 추가하거나 기존 기능을 재사용할 수 있는 구조를 구성했습니다.

<details>
<summary> SkillData </summary>

```cs
[CreateAssetMenu(fileName = "New SkillData", menuName = "Skill/SkillData")]
    public class SkillData : ScriptableObject
    {
        [Header("Information")] 
        public string skillName;
        public Sprite skillIcon;
        [TextArea] public string skillDescription;

        public enum Rarity { Common, Uncommon, Rare, Epic, Legendary }
        public Rarity rarity;
        public int price;
        
        [Header("")]
        [Required] public SkillBehavior behavior;
        
        [Tooltip("스킬 쿨타임")]
        public float cooldownTime;

        [Tooltip("시전 조건 상태")]
        public UnitStateType castableState;
        
        public ChargeProperty chargeProperty;
    }
```
</details> 

<details>
<summary> SkillBehavior </summary>

```cs
public abstract class SkillBehavior : ScriptableObject
    {
        [SerializeField] protected Animations animation = Animations.Attack;
        protected virtual void PlayAnimation(Unit caster) => caster.animator.Play(animation);

        /// <summary>
        /// 스킬이 발동 될 때 호출되는 함수
        /// </summary>
        public abstract void Enter(Unit caster);

        /// <summary>
        /// 스킬이 비활성화 될 때 호출되는 함수
        /// </summary>
        public abstract void Exit(Unit caster);

        /// <summary>
        /// 스킬이 완료되었는지 여부를 반환하는 함수
        /// </summary>
        public abstract bool IsComplete(Unit caster);

        /// <summary>
        /// 스킬이 활성화 되어 있는 동안 매 프레임 호출되는 함수
        /// </summary>
        public virtual void SkillUpdate(Unit caster, float deltaTime) { }
    } 
```
</details>

### 싱글톤
프로젝트 초기에는 개발 속도와 구현 난이도를 고려하여 싱글톤 패턴을 채택했습니다. 싱글플레이 기반의 로그라이크 게임이라는 점과 제한된 개발 기간, 팀 규모를 고려했을 때 가장 효율적인 선택이라고 판단했기 때문입니다.

다만 개발이 진행되면서 데이터와 상태 관리의 결합도가 높아졌고, 테스트와 확장 측면에서 구조적 한계도 경험할 수 있었습니다. 이를 통해 단순히 구현 편의성뿐 아니라 유지보수성과 확장성까지 고려한 설계의 중요성을 배우게 되었습니다.

### 협업

팀원들이 적극적으로 참여하고 활발하게 소통할 수 있어야 더 좋은 게임이 만들어진다고 생각했습니다. 따라서 아트 리소스가 완성되면 가능한 빠르게 게임 내 결과물을 공유하고, 매주 회의를 통해 현재 개발 진행 상황과 개선이 필요한 기획 요소, 다음 주 목표를 함께 정리하며 프로젝트를 진행했습니다.

또한 4인 규모의 소규모 팀인 만큼, 과도한 문서 작업보다 빠른 의사결정과 즉각적인 피드백이 중요하다고 판단했습니다. 이를 위해 최소한의 핵심 목표와 개발 일정을 먼저 설정한 뒤, 매주 기획을 보완하고 기능을 추가하는 방식으로 개발을 진행했습니다. 이러한 과정을 통해 팀원 모두가 프로젝트 방향과 기획에 직접 참여하고 있다는 공감대를 형성할 수 있었습니다.

### 전시 준비
전시를 준비하며 가장 중요하게 생각한 목표는 두 가지였습니다. 첫 번째는 정해진 일정 안에 프로젝트를 완성해 실제로 출품까지 진행하는 것이었고, 두 번째는 초기 기획 단계에서 설정한 핵심 게임 경험을 끝까지 구현하는 것이었습니다.

이를 위해서는 제한된 기간 안에서도 기능 수정과 확장이 가능한 구조, 그리고 원활한 협업 환경이 중요하다고 판단했습니다. 따라서 프로젝트 초기부터 유지보수성과 협업 효율을 고려해 시스템을 설계했으며, 기능 간 의존성을 줄이고 반복 작업을 최소화하는 방향으로 개발을 진행했습니다. 그 결과 일정 내에 전시 출품을 완료했을 뿐 아니라, 초기 목표로 설정했던 핵심 콘텐츠까지 구현할 수 있었습니다.

## 성과

- 버닝비버 온라인 전시 출품
- 발표 우수평가 장학금 수상
- 팀 프로젝트 완성 및 시연 진행 

