---
layout: default
title: GameSettings
description: Whoa dude, we actually have like, a usable settings menu now.
---

# Fortnite

Obligatory mention, but pretty much GameSettings Plugin is a plugin from Epic that is based on the settings menu from Fortnite. It offers a very nice base and mechanics for a settings menu, and allows you to easily skin it to your liking. At first glance it can look a bit spooky, but it's really not once you understand how it works.

# Game Setting Classes

`UGameSetting` is a single setting, for example: Window Resolution. It represents the base class for any setting that might appear in your list. There are a few notable types and if you ever played Fortnite you will probably recognize them visually.

Enumerated Options:
- `UGameSettingValueDiscreteDynamic` (Sets a float value)
- `UGameSettingValueDiscreteDynamic_Number`(Sets an int)
- `UGameSettingValueDiscreteDynamic_Enum` (Sets an Enum)

Slider:
- `UGameSettingValueScalarDynamic` (Sets a float value)

`UGameSettingCollection` is basically just an array of `UGameSetting`, You can think of them as a category, if you had a bunch of display options like "Window Mode" and "Resolution" that you wanted to go under a "display" heading. You'd make them a part of the collection.

`UGameSettingRegistry` is basically just a place that holds all the `UGameSettingCollection` to be used on the settings screen and is the interface between the actual settings screen widget.

# The ShooterGame LTS implementation

Since ShooterGame didn't have much in the way of a `UGameUserSettings` class in the first place, the stock one has been completely replaced in favour of a *heavily* stripped down version from Lyra. It's now found under `UShooterSettingsLocal.h`

Just like Lyra, the setting registry is split into seperate .cpp files by category to make the code more managable, You can find them at:
- `ShooterSettingRegistryAudio.cpp`
- `ShooterSettingRegistryGameplay.cpp`
- `ShooterSettingRegistryVideo.cpp`

**Important note:** In `ShooterSettingRegistry.h` there is the following macro:
```cpp
#define GET_LOCAL_SETTINGS_FUNCTION_PATH(FunctionOrPropertyName)
``` 
This macro allows each setting to grab the getters and setters directly from the `UGameUserSettings` class. As an example of this usage here is the way that we hook the WindowMode setting up to actually change the values in `UShooterSettingsLocal`:
```cpp
auto* WindowMode = NewObject<UGameSettingValueDiscreteDynamic_Enum>();
WindowMode->SetDynamicGetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(GetFullscreenMode));
WindowMode->SetDynamicSetter(GET_LOCAL_SETTINGS_FUNCTION_PATH(SetFullscreenMode));
```
The macro basically retrieves the getter and setter functions through our `UShooterLocalPlayer` which allows it to just directly hook into them, So when we edit the setting on the front-end, it changes the value, no more effort or setup required.

It is also important that the type of `UGameSetting` that you use is correct for the function you're targeting, as an example the WindowMode setting uses `UGameSettingValueDiscreteDynamic_Enum`, and if we look at the setter function we can see the parameter is of enum type:
```cpp
void SetFullscreenMode(EWindowMode::Type InFullscreenMode);
```

You can also see Epic's official [Game Settings Documentation](https://docs.unrealengine.com/5.0/en-US/lyra-sample-game-settings-in-unreal-engine/) for more information regarding how it works.