---
title: "Getting Started With Godot and VSCode"
date: 2021-03-14T13:36:50-07:00
draft: false
---

It's been a while since I wrote anything: it's not that I haven't been writing,
but rather I've created a split-off community-run blog for Star Wars: Legion,
and it's taken a bit of time to want to come back to general blogging.

I've been thinking about creating a new game/project, mostly as a learning
experience, and partially as something to do during this perpetual WFH period.
Instead of reflexively falling back on a technology I'm comfortable with (i.e.
React/TSX or Dart/Flutter), I've decided to try out [Godot][], an open source
and full-featured game engine with both 2D and 3D support.

[Godot]: https://godotengine.org/

Of course, I wasn't fully satisfied with learning or using [GDScript][] - it
seems interesting enough, but I wanted to use a statically typed high-level
language, and I wanted to use VSCode - so this is a good opportunity for [C#].

[GDScript]: https://docs.godotengine.org/en/stable/getting_started/scripting/gdscript/gdscript_basics.html
[C#]: https://docs.godotengine.org/en/stable/getting_started/scripting/c_sharp/index.html

<!--more-->

---

## Installation

I'm a fan of the package manager [Scoop][], a simple command-line installer:

```sh
scoop add extras
scoop install godot-mono
scoop install mono
```

[Scoop]: https://scoop-docs.now.sh/

A few minutes later, Godot and Mono were installed. You can also install them
with `.exe` installers available on their site, but that's totally your choice.
I already had [VSCode][] installed, but if you don't have it, I recommend it.

[VSCode]: https://code.visualstudio.com/

You'll need the following _extensions_ for VSCode:

* [C# for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [Mono Debug](https://marketplace.visualstudio.com/items?itemName=ms-vscode.mono-debug)
* [Godot Tools](https://marketplace.visualstudio.com/items?itemName=geequlim.godot-tools)

## Create a new project

Open the _Godot Editor_, and hit **New Project**. I called mine
`GodotRogueDemo`.

I did nothing else within the _Godot Editor_, and then opened the newly created
folder in VSCode, and hit `F1` - this opens the quick launcher for VSCode, and
I typed **Godot > Godot Tools: Open workspace with Godot Editor**.

You'll then need, once, to tell the VSCode extension where _Godot_ is installed.
Fortunately due to using `scoop`, this is a canonical location - so for me it
was `C:\Users\Matan\scoop\apps\godot-mono\current\godot.exe`, but you might have
it installed elsewhere.

## Create a scene and sprite

I created a new 2D scene, and dragged an instance of `icon.png` over, and then
hit **Right Click -> Attach Script**, and chose **C#**. The script ending up
looking like this:

```cs
using Godot;

public class Icon : Sprite
{
  public override void _Ready()
  {
    GD.Print("Hello World");
  }
}
```

To make debugging available in VSCode, I went, in the Godot Editor, to
**Project > Project Settings**, scrolled down to **Mono > Debugger Agent**, and
enabled the checkbox **Wait for Debugger**. I also bumped the timeout to `10000`
and copied the port to reference in VSCode (mine was `23685`).

Back into VSCode, I went into the **Run and Debug** menu, and created a new
configuration for **C# Mono** - my `launch.json` looks like this, but yours will
likely look very similar:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch",
      "type": "mono",
      "request": "launch",
      "program": "${workspaceRoot}/program.exe",
      "cwd": "${workspaceRoot}"
    },
    {
      "name": "Attach",
      "type": "mono",
      "request": "attach",
      "address": "localhost",
      "port": 23685
    }
  ]
}
```

Save your scene and script, and now it is possible to launch your Godot demo
project, and attach the debugger from VSCode. This isn't a completely seamless
experience (I'm hoping to mess around with `launch.json` more in the near future
and create a specific "debug" task that automates much of this), but it works!

![Demo](https://user-images.githubusercontent.com/168174/111084722-090fe500-84d1-11eb-891e-ddf1d8fd2390.png)
