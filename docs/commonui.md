---
layout: default
title: CommonUI
description: Some info about the CommonUI plugin and it's implementation within ShooterGame LTS.
---

# CommonUI?!

Hear me out - *it's actually pretty awesome*. One of the things Lyra got right, even if it did completely over-complicate it. Although the goal of Lyra is to be a mega modular, extensible transformer, so over there, it kind of does make sense.

Another important thing to note is that there is barely any documentation on CommonUI, so hopefully this somewhat serves as one.

# The ShooterGame LTS implementation

ShooterGame LTS does not use CommonUI in the same way as Lyra does, and there's a few very key differences:
- No extension points.
- No gameplay tags.
- Uses the HUD class.
- Does not rely on CommonGame.

So with that out the way lets get into it. First thing is first setting up CommonUI is a pain in the arse. They don't make it easy. But once it works, from a UX / UI Designer perspective, it's a black box that works awesomely, you don't have to really care how it works, you just throw it things at it to display.

`UShooterHUDWidgetHost` is the main widget that *always* gets added to the screen by `UShooterHUD`. By default it has 4 layers (`UCommonActivatableWidgetQueue`) where widgets can be contained on the screen :
- `Layer_Background` (For HUD elems like Healthbars)
- `Layer_Foreground` (For HUD elems like Scoreboards)
- `Layer_Menu` (For Menu elems like an ESC Menu)
- `Layer_Dialog` (For popup boxes aka "Do this thing? Yes / No")

Any content that gets added to the screen, you should do so via these layers, *friendship ended with `AddToViewport()`*. Instead now you can get a reference to `UShooterHUDWidgetHost` via `ShooterHUD` and call into this function:
```cpp
UCommonActivatableWidget* UShooterHUDWidgetHost::CreateWidgetOnLayer(EShooterUILayer Layer, TSubclassOf<UCommonActivatableWidget> WidgetClass);
```
Just like magic, if you call this function, it will automatically create and sort your widget onto the correct layer. **Important notes:**
- Only 1 widget is allowed at a time per layer.
- Widgets will "queue", so if you want to display a new widget, you must create it and deactivate the currently shown widget.
- You are still responsible for handling pointers to your widgets. (I store most of them in `UShooterHUD`)

If you came here from Lyra, you might be sad that i'm not using the gameplay tag extension points (or happy if you're like me). Frankly, I don't like the complexity that system adds and I think it overcomplicates the design process in favour of modularity. This project isn't about modularity, so I don't use it. Instead here you are responsible for your widgets and things are much more towards the "traditional" style of using the HUD class. It's likely not that much work to change it to "The Lyra way" on your fork if you prefer it that way.

# Activatable Widgets

My first opinion of these things was "wtf is this". When you get your brain into "CommonUI" mode, It actually makes sense. Pretty much all this widget does is show / hide when you activate or deactivate it, you don't have to `RemoveFromViewport()` or `RemoveFromParent()` anymore, which came with a GC cost. Instead you now just call `DeactivateWidget()` on it, and it will automatically remove itself from the screen, bringing any queued widgets forward to the front. As such it is important that you keep a reference to your widget if you want to reuse it.

You can add widgets back onto to the screen with:
```cpp
UCommonActivatableWidget* UShooterHUDWidgetHost::AddWidgetToLayer(EShooterUILayer Layer, UCommonActivatableWidget* Widget);
```

# Activatable Widget Containers

The base class of our UI layers. As of now there exists two main types:
- `UCommonActivatableWidgetStack`
- `UCommonActivatableWidgetQueue`

Both of these provide the same behaviour of automatically managing and handling widget deactivations / visibility. The main key difference is that `ActivatableWidgetStacks` let you have a root content widget that can't be removed.

# Automatic Input Handling

When you first setup CommonUI, if you're absolutely new to it and by chance *didn't* copy Lyra's implementation over, You will probably be like "Yo wtf! why is my cursor randomly showing up!". Well this is actually one of the things that makes CommonUI awesome, but out of the box either need to disable `Enable Default Input Config` under the `Common Input Settings` in your Project Settings, or you need to do it the "Lyra way" and subclass `UCommonActivatableWidget` and override `GetDesiredInputConfig()` like this:

## .cpp
```cpp
TOptional<FUIInputConfig> UShooterActivatableWidget::GetDesiredInputConfig() const
{
	switch (InputConfig)
	{
		case EShooterWidgetInputMode::GameAndMenu:
			return FUIInputConfig(ECommonInputMode::All, GameMouseCaptureMode);
		case EShooterWidgetInputMode::Game:
			return FUIInputConfig(ECommonInputMode::Game, GameMouseCaptureMode);
		case EShooterWidgetInputMode::Menu:
			return FUIInputConfig(ECommonInputMode::Menu, EMouseCaptureMode::NoCapture);
		case EShooterWidgetInputMode::Default:
		default:
			return TOptional<FUIInputConfig>();
	}
}
```
## .h
```cpp
public:

	virtual TOptional<FUIInputConfig> GetDesiredInputConfig() const override;

protected:

	/** The desired input mode to use while this UI is activated, for example do you want key presses to still reach the game/player controller? */
	UPROPERTY(EditDefaultsOnly, Category = Input)
	EShooterWidgetInputMode InputConfig = EShooterWidgetInputMode::Default;

	/** The desired mouse behavior when the game gets input. */
	UPROPERTY(EditDefaultsOnly, Category = Input)
	EMouseCaptureMode GameMouseCaptureMode = EMouseCaptureMode::CapturePermanently;
```
From here in every widget you ever setup, in the class defaults of the Blueprint you can specify the input mode you want whenever that widget comes on the screen, *pretty cool huh?!* So for example an Escape Menu you might need your cursor, so you set the input config on that widget to `Menu` and then it will automatically handle showing your cursor while the widget is up.

# Gotchas / Cons

For whatever reason, with CommonUI Epic has decided that a whole bunch of classes should be abstract, and as such to even just use them you have to create a class from them, which can add code bloat. For this reason ShooterGameLTS has a specific module under `/Source` called `ShooterUI` which is for `Widget` classes that you could consider to be "reusable" and don't really care about the fact your game exists. This module is **NOT** for your Game UI code, put that inside `ShooterGame` module in the `/UI` folder.