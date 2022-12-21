---
layout: default
title: AssetManager
description: How ShooterGame LTS utilizes the AssetManager.
---

# Summary

*Say it with me* - Constructor Helpers Bad. Ctor Helpers Bad.

Okay now we have that out of the way. ShooterGame notoriously makes use of ctor helpers in order to load many assets, this not only forces you into hard pathing your assets - aka you can't move them, and so it makes code prone to breakage - usually you shouldn't move assets unless you have to anyways, but you can!

The other really bad thing about ctor helpers is that they will load whatever they reference *on the spot*. This means that using them in constructors can cause things to load before it's safe to load them. As a result Epic invented `TSoftObjectPtr` and `TSoftClassPtr`, they are awesome and you should use them.

The reason these things absolutely rule is because it gives you full control over when an asset *or a class* is loaded. For Blueprint classes especially this is a huge deal, because loading in Unreal Engine is [recursive](https://en.wikipedia.org/wiki/Recursion). For example if you have a Blueprint that hard references 500 different static meshes and you have a ctor helper that points to that Blueprint, you load all 500 of those static meshes right there and then (which is in the long run slow).

At an absolute worst case you can end up daisy-chaining asset loads and recursively reload the same asset that you're referencing from ([circular referencing](https://en.wikipedia.org/wiki/Circular_reference)), which can cause a really annoying crash, usually only in packaged. A good example is hard referencing something like a `Player Controller` Blueprint, which is likely to indirectly reference most assets in the project provided you don't use `Soft References`.

As such ShooterGame LTS has removed all ctor helpers and is actively replacing any unnecessary hard references, to help with loading times and stability.

# Why do I need to manage my assets bro?

Because it's great! And it makes your life much easier, if you have for example a bunch of weapon skins; `Asset Manager` gives you a fast and easy way to collect up all of those assets and then you could present them on a UI.

ShooterGame LTS is setup to use the `AssetManager` to load a general game data asset when the game or project loads. The game data asset can be found at the root `/Content` directory level.

This code for this asset type can be found under `UShooterGameData` in `ShooterGameData.h` and it contains asset references to massive amount of the core asset references which were previously hardpathed with constructor helpers or we scattered around different assets and unorganized. 

Epic has some pretty good information on the [Asset Manager](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/AssetManagement/) here.

# Why not just use UDeveloperSettings

Aka this:
```cpp
GetDefault<UMyDeveloperSettings>()->MyAsset.LoadSynchronous();
```
Instead of this:
```cpp
UShooterGameData::Get().MyAsset.LoadSynchronous();
```
The answer is pretty simple and very boring; Because I prefer this way - *really I have no better answer than that*. Purely preference here, feel free to replace it on your fork if you dislike it.

I think you can argue all day about which method is better (I think there's barely a difference personally) but my opinion is that it's nicer having it in the content browser, more readable, and I prefer that the game doesn't load if the game data isn't present, You absolutely know if it's broken this way. Theoretically if you use it right then it also makes `Asset Bundling` and breaking up your game data into smaller chunks more sane.