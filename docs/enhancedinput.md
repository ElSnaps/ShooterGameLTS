---
layout: default
title: EnhancedInput
description: How ShooterGame LTS uses Enhanced Input. It's like normal inputs but much more better.
---

# Why move to enhanced input?

In Unreal Engine 5.1, `Legacy Input` - aka the old way of setting up inputs in the project settings is deprecated. As a result i've modernized the input system for ShooterGame LTS.

# Input Actions

Enhanced input has two main asset types that are heavily utilized. The first one is your `Input Action` data assets, these are prefixed with `IA_` and you should mostly find them under the `Content/Input` directory. Note that input actions only describe *how* an action is triggered - aka if it's a binary state or an axis, they do not describe the key that triggers them. (More on that later)

Input Actions are how you define what used to be `Axis Mappings` and `Action Mappings`, They've now been merged into the same thing. Also an important note is that Enhanced Input has support for `Multi-dimensional Mappings`, for example if you have your movement keys (usually WASD) - These can now be represented by an `FVector2D` instead of two seperate floats like you'd have to do with Legacy Input. In the case of ShooterGame LTS, we just have one generic `Move` function that uses `FVector3D` handles moving Forward / Back, Left / Right, Up / Down (Flying only). You will see it under this function:

```cpp
void AShooterCharacter::Move(const FInputActionValue& Value)
```

# Input Mapping Contexts

The second one of the asset types that are utilized is `Input Mapping Context` data assets, these are prefixed with `IMC_` and are the primary way that you tell Unreal Engine, what keybindings map to which actions. A lot like the `Legacy Input` system where you'd set up your bindings in the project settings, You now do that in mapping contexts instead.

In ShooterGame LTS, I tried to go with a pretty simple setup that somewhat matched what existed there before. You can find the main mapping context under `IMC_Default` under `Content/Input`. In the future I may add more but for now this seemed the simplest solution.

# Loading our Data

One drawback of Enhanced Input is that the related data is now stored as an asset. Which means you must load it yourself.

ShooterGame LTS is setup to use an `AssetManager` to load a general game data asset when the game or project loads. This asset type can be found under `UShooterGameData` in `ShooterGameData.h` and it contains asset references to all the `Input Actions` and `Input Mapping Contexts` in usage. Since this game data asset is checked and loaded when the project starts up you can assume that it's going to be safe to use in any game function.

Here we use it to load an `Input Mapping Context`:
```cpp
UInputMappingContext* DesktopContext = UShooterGameData::Get().DesktopInputMappingContext.LoadSynchronous();
```
and here we use it to load an `Input Action`:
```cpp
UInputAction* MoveAction = UShooterGameData::Get().InputActionMove.LoadSynchronous();
```

# Mapping Context Setup

The main place we bind our input mapping context is under the following function:
```cpp
void AShooterCharacter::PawnClientRestart()
```
This is how we add mapping contexts, you must get the `EnhancedInputLocalPlayerSubsystem` and then call `AddMappingContext()`
```cpp
if (APlayerController* PC = Cast<APlayerController>(GetController()))
{
	if (auto* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer()))
	{
		UInputMappingContext* DesktopContext = UShooterGameData::Get().DesktopInputMappingContext.LoadSynchronous();

		Subsystem->ClearAllMappings();
		Subsystem->AddMappingContext(DesktopContext, 0);
    }
}
```

# Input Action Setup

Like with Legacy Input we setup our inputs in the same function:
```cpp
void AShooterCharacter::SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent)
```
An important distinction with Enhanced Input is that you must cast the input component. This is setup inside your project settings under `DefaultPlayerInputClass`. Any project made after 5.1 should be using this by default now, and atleast in ShooterGameLTS, we aren't ever going to be using anything else, so we `CastChecked` just to make sure we always use it but we don't need additional safety.
```cpp
auto* EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(PlayerInputComponent);

UInputAction* MoveAction = UShooterGameData::Get().InputActionMove.LoadSynchronous();
EnhancedInputComponent->BindAction(MoveAction, ETriggerEvent::Triggered, this, &ThisClass::Move);
```