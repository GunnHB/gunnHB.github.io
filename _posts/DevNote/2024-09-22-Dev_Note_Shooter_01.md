---
title:  "[DevNote] 001. 캐릭터 생성"
excerpt: "언리얼 개발 일지입니다!"

author: gunnHB
categories: 
 - DevNote
tags: 
 - [Github, Git, Unreal, Programming, DevNote, C++]

toc: true
toc_sticky: true
 
date: 2024-09-22
last_modified_at: 2024-09-22
---

🔔 TPS 프로젝트 개발을 진행하면서 작성한 내용입니다! 모든 내용을 기술하진 않았으니 참고 부탁드립니다! 🔔
{: .notice}

## 폰 생성
우선 프로젝트에 사용될 플레이어 `폰Pawn`을 만들어야 합니다. 언리얼 에디터에서 폰을 상속받아 클래스를 생성해주면 됩니다. 클래스명의 경우 저는 PlayerPawn이라고 했습니다.

![image](https://github.com/user-attachments/assets/23539e6c-8da6-4366-a872-7a3e99c06a02){: width="70%" height="70%"}

생성 후 컴파일이 완료되면 `헤더 파일(.h)`과 `cpp 파일(.cpp)`이 생기는 것을 확인할 수 있습니다. <u>헤더 파일은 선언부</u>, <u>cpp 파일은 구현부</u>의 역할을 합니다.
우선 헤더 파일에 플레이어에 사용될 여러 요소들을 추가해봅시다.

### CapsuleComponent
`CapsuleComponent`는 플레이어의 콜리전을 위한 컴포넌트입니다. 이는 다른 물체들과 부딪힘을 감지하는 역할을 하는데, 적에게 공격받거나 땅에 발이 닿았는지 등을 감지할 수 있습니다.

### SkeletalMeshComponent
우선 `Mesh`란 물체를 나타내는 삼각형들의 집합입니다. 마치 하나의 조각상을 여러 삼각형으로 쪼개어 표현한 것과 같습니다. 여기서 파생된 `SekeltalMesh`는 뼈대가 존재하여
애니메이션 재생이 가능한 매시를 말합니다. `SkeletalMeshComponent`는 이러한 SkeletalMesh의 기능을 담고 있는 컴포넌트를 의미합니다.

## 컴포넌트 추가하기
생성된 폰에 컴포넌트를 추가하기 위해 다음과 같이 작성해줍시다.

```c++
// PlayerPawn.h

UCLASS()
class SHOOTER_API APlayerPawn : public APawn
{
	GENERATED_BODY()

public:
	APlayerPawn();

protected:
	UPROPERTY(VisibleAnywhere) TObjectPtr<UCapsuleComponent> mCapsuleComp = nullptr;
	UPROPERTY(VisibleAnywhere) TObjectPtr<USkeletalMeshComponent> mMeshComp = nullptr;
}
```

컴포넌트를 사용하기 위해 변수를 선언합니다. 여기서 TObjectPtr을 이용해 변수를 선언하는데 일반적으로 사용되는 *을 이용해 선언해도 됩니다.
그리고 선언부의 가장 앞에는 UPROPERTY라는 매크로를 사용합니다. 이는 변수를 외부에서 편집할 수 있도록 도와주는 역할을 한다고 이해할 수 있습니다.

이렇게 선언한 변수들은 cpp 파일에서 초기화해주면 됩니다.

```c++
APlayerPawn::APlayerPawn()
{
	PrimaryActorTick.bCanEverTick = true;

    mCapsuleComp = CreateDefaultSubobject<UCapsuleComponent>(TEXT("CapsuleComponent"));
    mMeshComp = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("MeshComponent"));
}
```

생성한 플레이어 폰에 `액터 컴포넌트Actor Component`를 생성하여 변수에 할당해줍니다. `CreateDefaultSubobject`는 UObject를 생성해주는 역할을 합니다.
이는 반드시 생성자에서 수행되어야 합니다.

```c++
APlayerPawn::APlayerPawn()
{
    ...

    SetRootComponent(mCapsuleComp);
    mMeshComp->SetupAttachment(mCapsuleComp);
}
```

이렇게 생성된 컴포넌트들은 계층 구조를 지정해줍니다. CapsuleComponent를 가장 상단으로 만들고 하위에 SkeletalMeshComponent를 두어
CapsuleComponent의 이동에 SkeletalMeshComponent의 상대적 위치 값에는 영향받지 않도록 합니다.

```c++
APlayerPawn::APlayerPawn()
{
    ...

    mCapsuleComp->SetCapsuleHalfHeight(100.f);
	mCapsuleComp->SetCapsuleRadius(30.f);

    mMeshComp->SetRelativeLocation(FVector(0.f, 0.f, 100.f));
	mMeshComp->SetRelativeRotation(FRotator(0.f, -90.f, 0.f));
}
```

이후 컴포넌트들의 값들을 수정해줍니다.

먼저 캡슐의 `SetCapsuleHalfHeight`는 캡슐 높이의 절반 값을 세팅하는 작업을 수행합니다. 이를 통해 캡슐의 높이를 설정할 수 있습니다.
`SetCapsuleRadius`는 캡슐의 반경을 설정해줍니다. 이를 이용해 캡슐의 크기 조절이 가능합니다. 참고로 CapsuleRadius 값은 HalfHeight 값을 넘길 수 없습니다.

다음 매시의 `SetRelativeLocation`는 매시의 상대적 위치를 수정하는 역할을 합니다. 일반적으로 캡슐 높이의 값만큼 아래로 내려줍니다.
`SetRelativeRotation`는 매시의 상대적 회전 값을 수정해줍니다. 이는 기본 매시를 확인하면서 수정해주는 것이 좋습니다.

언리얼에서의 회전은 X가 Roll, Y가 Pitch, Z가 Yaw로 표현됩니다.
{: .notice--info}

## 매시 세팅하기
플레이어로 사용할 매시를 세팅해봅시다. 원하는 어떠한 매시를 사용해도 괜찮습니다. 해당 프로젝트에서는 [파라곤의 Murdock]("https://www.unrealengine.com/marketplace/ko/product/paragon-murdock")을 사용했습니다.

매시를 다운로드받으면 컨텐츠 폴더에 해당 모델에 대한 여러 에셋들을 확인할 수 있습니다. 그 중 [ParagonMurdock\Characters\Heroes\Murdock\Skins\A_Executioner\Mesh]에 존재하는
매시를 사용하겠습니다. 해당 SkeletalMesh를 우클릭하여 레퍼런스 복사해줍니다.

![image](https://github.com/user-attachments/assets/1b18a79d-6cbe-4317-8e73-9fc5176f9ebb){: width="70%" height="70%"}

그 이후 cpp파일로 돌아옵니다.

```c++
APlayerPawn::APlayerPawn()
{
    ...

	ConstructorHelpers::FObjectFinder<USkeletalMesh>
    meshAsset (TEXT("/Script/Engine.SkeletalMesh'/Game/ParagonMurdock/Characters/Heroes/Murdock/Skins/A_Executioner/Mesh/Murdock_Executioner.Murdock_Executioner'"));

	if(meshAsset.Successed())
		mMeshComp->SetSkeletalMesh(meshAsset);
}
```

위처럼 ConstructorHelpers::FObjectFinder를 이용해 에셋을 가져옵니다. 가져온 에셋의 성공 여부를 판단하여 컴포넌트의 SetSkeletalMesh를 이용해 에셋을 세팅해주면 됩니다.

이를 이용해 최종적으로 컴파일 후 폰을 뷰포트에 끌어다 놓으면 에셋이 적용된 플레이어 폰을 확인할 수 있습니다.

![image](https://github.com/user-attachments/assets/58487085-f6d2-4349-aa4a-34fcb6c051fd){: width="70%" height="70%"}

## 마무리
이번 포스팅에서는 클래스의 생성과 컴포넌트의 추가 등을 진행했습니다. 다음 포스팅에서는 EnhancedInput을 이용한 플레이어의 이동을 구현해보겠습니다.

감사합니다!😁