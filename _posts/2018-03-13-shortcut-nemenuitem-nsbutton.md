---
title:  "Shortcut of NSMenuItem and NSButton"
date:   2023-07-20 10:08:50 +0800
toc: true
toc_label: "Table Of Content"
toc_icon: "columns"  # corresponding Font Awesome icon name (without fa prefix)
---

The shortcut key, or hotkey, is called `Key Equivalent` in Mac development. To use it, just press the corresponding key combination, and it will trigger a predefined application function.

Among NSControl's subclass, NSMenuItem and NSButton support key equivalent, which can be implemented in a programmatic way or by changing the xib object property. 

## 1. Composition of Key Equivalent 

Normally, the key equivalent is a composite of two parts: some modifier keys and one basic key.

There are 4 possible modifier keys:

Symbol | Name | 
--- | --- | 
⌘ | Command
⌃ | Control
⇧ | Shift
⌥ | Option, alias Alternate

The normal key, includes alphabet, number, punctuation, symbol key, etc.

Key Equivalent may have no modifier key. This is very common in high-efficiency professional applications, for example, in Photoshop, moving layer's shortcut key is `V`.

For symbol key, it refers to keys like `⎋` (escape), `↑` (up arrow). On Mac laptops, `⌦` (delete), `⇞` (page up), `⇟` (page down), `↖` (home), and `↘` (end) do not have dedicated physical keys. But there are substitutes to accomplish the same function:

Key | Substitute 
--- | ---
Delete | Fn + ⌫
Page Up | Fn + ↑
Page Down | Fn + ↓
Home | Fn + ←
End | Fn + →

## 2. Changing the xib object property

In **Attributes** panel of NSMenuItem/NSButton, **Key Equivalent** input field is used to set the shortcut. For example, a character, E, will be displayed capitalized.

If the shortcut includes `⇧` key, the input field will show an Alternates button, to let you choose displaying style between, like  `⌘+` and `⇧⌘=`.

![](https://user-images.githubusercontent.com/55504/37417474-5cafc540-27eb-11e8-830a-58ddb9c7defb.gif)

## 3. Programmatic Way

### 3.1 Normal Key

The following code sets menuItem's shortcut as `E`.

{% highlight objective_c linenos %}
[menuItem setKeyEquivalentModifierMask:!NSEventModifierFlagCommand];
[menuItem setKeyEquivalent:@"e"];
{% endhighlight %}

The following code sets menuItem's shortcut as `⌘E`.

{% highlight objective_c linenos %}
[menuItem setKeyEquivalent:@"e"];
{% endhighlight %}

The following code sets menuItem's shortcut as `⇧⌘E`.

{% highlight objective_c linenos %}
[menuItem setKeyEquivalent:@"E"];
{% endhighlight %}

### 3.2 Symbol Key

First of all, you need to know the corresponding Unicode value.

Symbol | Name | Unicode
--- | --- | ---
⌫ | Backspace | 0x0008
⇥ | Tab | 0x0009
↩ | Return | 0x000d
⎋ | Escape | 0x001b
← | Left | 0x001c
→ | Right | 0x001d
↑ | Up | 0x001e
↓ | Down | 0x001f
␣ | Space | 0x0020
⌦ | Delete | 0x007f
↖ | Home | 0x2196
↘ | End | 0x2198
⇞ | Page Up | 0x21de 
⇟ | Page Down | 0x21df

**Note:** `NSBackspaceCharacter`, `NSTabCharacter`, and `NSCarriageReturnCharacter` are defined in *NSText.h*, others are not. All of them (except 4 arrow keys) can be found [here](https://unicode-table.com/en){:target="_blank"}. Arrow keys do not use 0x2190, 0x2191, 0x2192, 0x2193, I don't know the reason. 
{: .notice--info}

The following code sets menuItem's shortcut as `⌘↩`.

{% highlight objective_c linenos %}
NSString *s = [NSString stringWithFormat:@"%c", NSCarriageReturnCharacter];
[menuItem setKeyEquivalent:s];
{% endhighlight %}

The following code sets menuItem's shortcut as `⌘↑`.

{% highlight objective_c linenos %}
NSString *s = [NSString stringWithFormat:@"%C", 0x001e];
[menuItem setKeyEquivalent:s];
{% endhighlight %}

**Attention:** the format specifiers are different, the latter **MUST** use `%C`.
{: .notice--danger}

One more thing, the shortcuts displayed by menu items are different than the symbols on xib canvas and those printed on some keyboards.

In xib | Running application
--- | --- 
<img width="200" alt="" src="https://user-images.githubusercontent.com/55504/37520582-9bb5dd4c-2958-11e8-9c6e-0e442beca151.png"> | <img width="200" alt="" src="https://user-images.githubusercontent.com/55504/37520583-9bf7487c-2958-11e8-8363-1e3990fe26bc.png">

Some keyboard models:

<a href="https://user-images.githubusercontent.com/55504/37498987-5c8c51ca-28fc-11e8-880e-5389466e76ee.jpg" target="_blank">
<img width="150" alt="" src="https://user-images.githubusercontent.com/55504/37498987-5c8c51ca-28fc-11e8-880e-5389466e76ee.jpg" />
</a>
<figure>
  <figcaption>Apple - Magic Keyboard with Numeric Keypad</figcaption>
</figure>
 
<a href="https://user-images.githubusercontent.com/55504/37498988-5cd81308-28fc-11e8-9423-a70fe8c2aa5f.jpg" target="_blank">
<img width="150" alt="" src="https://user-images.githubusercontent.com/55504/37498988-5cd81308-28fc-11e8-9423-a70fe8c2aa5f.jpg" />
</a>
<figure>
  <figcaption>Belkin - YourType Bluetooth Wireless Keypad</figcaption>
</figure>

## 4. Dynamic Menu Items

Option key has an alias, Alternate key, it is called so when holding it, the menu item will appear in a different way. 

In Apple's *Human Interface Guidelines*, this menu item is called [Dynamic Menu Items](https://developer.apple.com/macos/human-interface-guidelines/menus/menu-anatomy/){:target="_blank"}, and invisible by default.

For instance, click the MacOS desktop's left top `` menu item, then hold the option key, and you will see, "About This Mac" changes to "System Information..." and its triggered action changes too.

![](https://user-images.githubusercontent.com/55504/37418708-cde64e8a-27ed-11e8-884a-022c6a51fe6d.gif)

In xib, to implement this:

1. Add 2 NSMenuItem, they **MUST** be adjacent, with no other menu item between them.
2. Set a valid shortcut for them, and the second's shortcut **MUST** be the first's shortcut, plus the`⌥` key.
3. Check **Alternate** property for the second NSMenuItem.

Now, run the app and it will work as above `` menu item example.

![](https://user-images.githubusercontent.com/55504/37417755-03bbaffc-27ec-11e8-8cad-b9445ae05777.gif)

Besides, in step 2, if you switch shortcuts of these two menu items, the default visible will be the second one instead, while they're still **alternate**.

![](https://user-images.githubusercontent.com/55504/37417756-040844fc-27ec-11e8-9539-ff3e068b7c15.gif)

## 5. Some Design Guideline

1. Do **NOT** add a shortcut for the contextual menu. [Link](https://developer.apple.com/macos/human-interface-guidelines/menus/contextual-menus/){:target="_blank"}
2. Do **NOT** conflict with system shortcuts or other popular shortcuts, like `⇧⌘Q` (log out account), `⌘C` (copy), etc.
3. Only add shortcuts for frequently used actions, to relieve the user's learning and remembering burden. For example, about, is a rarely-used action, it does not need a shortcut.
4. If the text on NSMenuItem and NSButton has the suffix "...", it usually means that the corresponding action is important and needs to be reconfirmed by the user. So an alert panel normally pops up and asks the user if he wants to do so.

## 6. Other Weird Facts

1. `⌃⇧1` and `⇧1` can not exist together, or the former triggers the latter's action.
2. `⌃⌥⇧1` and `⇧1` can not exist together, or the former triggers the latter's action.
3. `⌃⌥⇧1` and `⌃⇧1` can not exist together, or the former triggers the latter's action.
4. `⌃⇧A`, its log info shows incorrectly for **⌃A**, both in code and xib. What's more, In code, if `keyEquivalent` is a capitalized alphabet and `keyEquivalentModifierMask` does not include `NSEventModifierFlagShift`, the system will add `⇧` automatically in the shortcut UI.
5. In xib, set a menu item or button's Key Equivalent with `⇧`, `⌘`, `=`, and choose alternates as `⌘+`, then `⌘=` and `⇧⌘=` can both trigger its action.
6. When setting `keyEquivalentModifierMask` for NSButton, it can not include `NSEventModifierFlagControl`, or the shortcut will not work.

## 7. Reference

1. [*Human Interface Guidelines*](https://developer.apple.com/macos/human-interface-guidelines/menus/menu-anatomy/){:target="_blank"}
2. [*Application Menu and Pop-up List Programming Topics*](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MenuList/Articles/SettingMenuKeyEquiv.html#//apple_ref/doc/uid/20000264-BBCDCAIG){:target="_blank"}
3. [https://stackoverflow.com/a/6230866/353927](https://stackoverflow.com/a/6230866/353927){:target="_blank"}
4. [Unicode character table](https://unicode-table.com/en){:target="_blank"}

## 8. Demo Project

[Here](https://github.com/cool8jay/KeyEquivelant){:target="_blank"} is demo project.
