---
layout: default
title: IWYU
description: All about include what you use.
---

# History lessons

So back in the days before fire was invented at Epic Games, in Unreal we used to use these things called "Monolithic Header Files", and basically all that means is that it's a gigantic header file with a whole bunch of other includes in it, they are enormous - and you may only use like 1 of the included headers, but that doesn't matter because you're still including everything anyway.

If you look at older Unreal Engine projects ported over UDK or made in earlier UE4 (Especially ShooterGame), you might see some of these headers included around:
```cpp
#include "Engine.h"
#include "SlateBasics.h"
#include "SlateExtras.h"
#include "ParticleDefinitions.h"
#include "SoundDefinitions.h"
```
These are Monolithic Headers, are mostly *non-optimal* to use (if you value yours and other people's time) because they increase your compile times fairly dramatically.

You can find these headers pretty easily in any half decent IDE / Text editor if you search for this include or macro (These files shouldn't mostly be used now but exist for backwards compatibility):
```cpp
#include "Misc/MonolithicHeaderBoilerplate.h"
MONOLITHIC_HEADER_BOILERPLATE()
```

# Okay great, Wtf is a IWYU

A solution to better compile times! All it means is literally, you include exactly, and only what you're going to use - and very importantly you mostly do it in the .cpp file rather than in your .h file.

Typically your header file will have includes looking something like this:
```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"
```

`CoreMinimal.h` is the header to include most of Unreal's core types, which is still technically a bit naughty, but far less naughty. Mostly it looks like Epic is moving away from this now and getting stricter with IWYU, if you look at newer code like Lyra, they manually include absolutely everything, even stuff like `TArray` like so:
```cpp
#include "Containers/Array.h"
```
For now with ShooterGame LTS, I still am continuing to use `CoreMinimal.h` until Epic says we probably should stop doing that.

We then also have `GameFramework/Actor.h`, which is the parent class and `MyActor.generated.h` which is the generated file that UBT makes in order to make your life much easier in regards to not having to write a ridiculous amount of boilerplate code.

Epic has actually a [pretty good documentation on IWYU](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/BuildTools/UnrealBuildTool/IWYU/), so if you're really interested in the fine details or migrating your own project, they probably explain it better than me.

# Why do I care? Cut to the chase.

ShooterGame LTS has been entirely migrated to IWYU and `BuildSettingsVersion.V2` which is currently as of Unreal Engine 5.1, the official standard. When Epic inevitably starts enforcing stricter IWYU, the project will also be moving to that.

Another thing you might notice about the project is that the `Private` / `Public` source split is gone, and for game modules you no longer need to use it.

As a result of moving to IWYU, `ShooterGame.h` is also gone. Previously this acted as a project level monolithic header combined with the module definition, and it also held a bunch of macros and other mess that just got chucked in there over the years. The macros can now be found under `ShooterMacros.h`, the module header itself is found under `ShooterGameModule.h`.

