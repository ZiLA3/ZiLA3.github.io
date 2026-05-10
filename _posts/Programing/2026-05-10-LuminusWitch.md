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

"루미너스 위치"는 마법을 완성하며 던전을 탐험하는 덱빌딩 로그라이크 게임입니다. 
Unity 기반으로 4인으로 제작하였습니다. 개발 기간은 2025.06 ~ 2026.01 까지 진행하였으며
버닝비버 전시 출품을 목표로 개발을 진행하였습니다.

---
## 맡은 역할

- 게임 시스템 구조 설계
- UI/UX 구조 설계 및 개발
- 일정 관리 및 협업 조율
- 구현 방향 정리 및 기획 문서 작성
- 이펙트 디자인

## 기술 스택
Engine: Unity
Language: C#
Collaboration: Git, Notion, Figma
Tools: Photoshop, GIMP...

## 경험

### 유지보수성 확보 

덱 빌딩 로그라이크의 장르 특성상 반복 사용되는 기능이 많아 재사용이 가능한 구조를 구현해야했다. 아이템, 주문, 유물, 버프, 맵 디자인, 기믹 등을 추가할 시 코드를 최대한 생성하지 않고, 데이터로 생성이 가능하도록 개발했습니다. 

이런 부분에서 가장 드러나는 부분은 주문인데, SkillData와 SkillBehavior로 분리하여, 스킬들이 겹치는 부분인 쿨타임과 사용 가능한 유닛 상태 등을 SkillData로 관리하고, SkillBehavior로 각 스킬마다 필요한 스탯을 분리하여 (예: 대시 거리) 같은 스킬을 쉽게 재사용 할 수 있었습니다. 

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
싱글톤 디자인을 채용하면서 많은 고민을 했었다. 가장 쉽고 간편한 방법이지만 데이터와 상태가 꼬일 가능성이 많고, 테스트, 멀티스레딩에서 분명한 단점이 존재하였기 때문이다. 그러나, 싱글플레이 방식의 로그라이크라는 부분과 짧은 기간, 팀원의 실력 등을 고려해 싱글톤 디자인을 채용하게 되었다.
당시에는 이 방식이 옳다고 느꼈으나, 지금 생각해보면 아쉬운 부분으로 남는다.

### 협업

팀원들이 보다 적극적으로 참여하고 활발한 소통이 되어야 엇나가지 않고 좋은 게임이 만들어 진다고 생각한다. 게임 아트를 그려주면 최대한 빠르게 그 결과물을 보여주도록 매 주마다 회의를 통해 현재 게임 개발 진행사항, 기획에서 수정하고 싶은 부분, 다음 주까지 해야할 일들을 정하도록 했다. 

또한 최대한 소통에 들어가는 비용을 줄이고 싶었다. 4인 이라는 소규모로 진행된 만큼, 문서화도 중요하지만 정말 중요한 게임에 집중하지 못할까 두려웠다. 그래서 최대한 즉각적인 소통을 통해서 개발을 진행하고자 회의를 매 주로 잡은 것도 있었다. 최소한으로 만들어야 하는 최종 기한을 설정하고, 그에 맞춰서 매 주마다 기획을 추가하고 게임 개발을 진행하였다. 이를 통해서 인원 모두가 기획에 참여한다는 느낌을 받았으면 했다.

### 전시 준비
전시를 준비하면서, 크게 두 가지 목표를 잡았었다. 첫 번째는 정해진 시간 안에 무조건 절차를 진행하고 게임을 제출하는 것이였고, 두 번째는 처음 정한 게임의 내용까지 제작을 완성하는 것이였다. 두 목표를 달성하기 위해서는 유지보수성과 협업 효율이 뛰어났어야 했었고, 처음 시스템을 구성할 때 최대한 유지보수성에 힘을 쓰면서 진행한 결과, 성공적으로 처음 정한 부분까지 진행하게 되었다. 

### 이펙트 디자인
4인으로 진행한 개발이다 보니, 이펙트 디자인을 할 사람이 없어서 내가 진행하게 되었다. 개발보다 어려운 부분이 많았고, 기능들을 하나씩 살펴보면 이해할 수 있었으나, 여러 기능을 섞어서 보기 좋은 이펙트를 만드는 데에는 어려움이 있었다. 이를 해결해 보고자 전시회 당시 인디 게임 개발자들에게 질문하며 알아보다 다들 이펙트디자인 담당을 둔다는 사실을 알게 되며 게임 개발에 난항을 겪었었던 점이 아쉬운 부분이였다.

## 성과

- 버닝비버 온라인 전시 출품
- 발표 우수평가 장학금 수상
- 팀 프로젝트 완성 및 시연 진행 

