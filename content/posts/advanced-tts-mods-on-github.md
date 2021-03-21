---
title: "Advanced TTS Mods on Github"
date: 2021-03-21T15:34:30-07:00
draft: false
---

I've been working on the [Star Wars Legion Tabletop Simulator Mod][1] in some
capacity for almost **2** years now. For those not familiar, Star Wars Legion is
a fairly complex tabletop miniatures game somewhat similar to Warhammer 40K and
similar titles - it is a quasi turn-based tactical game with _lots_ of rules,
tokens, minis, and various fiddly bits.

[1]: https://github.com/swlegion/tts

![An example of the game in action](https://user-images.githubusercontent.com/168174/111917323-922c9c00-8a3c-11eb-8508-f7f0027ccebc.png)

One of the goals originally for this project was that I wanted it to be as
collaborative as possible - i.e. be able to support multiple different people
contributing in different capacities - including scripting and re-designing the
table.

However, I quickly found out that having more than one person making edits to
a monolithic (at the time, 20Mb) JSON file (the format of save files/mod files
for Tabletop Simulator) was not scaling, and, as a result, designed a whole
custom system of tools that I wanted to share with the community.

<!--more-->

---

## Why use source control

I realize not all of my audience is going to understand all of the various
acronyms and technology being used in this blog. In a perfect world, I'd have
unlimited time to explain everything, but here is a quick rundown.

In the beginning, "shared" control of files (in this case, a TTS Save File) was
largely manual. If you've worked in a professional setting for at least 10 years
you probably remember (or still see!) files like `TPS-REPORT-v5-FINAL.doc`. I
like to compare this (manual) version control to PBEM ([Play-by-mail game][2]),
and this is sort of the "obvious" first step for collaborating on a TTS mod.

[2]: https://en.wikipedia.org/wiki/Play-by-mail_game

At some point, you might make the jump to a shared _Google Drive_ or _Dropbox_
(or whatever is the hotness right now in shared file hosting), perhaps with a
fancy auto-synchronization rule. This alleviates the need to explicitly share
(and receive) updates for the mod, but isn't truly collaborative - it ends up
being a locking-style of version control, where effectively only one person can
make edits at a time.

Using a version control system (source control) like [Git][3] or [Mercurial][4]
provides a lot of additional benefits - you can see the history of files, you
can revert (sometimes called "rollback") to earlier versions, and, if done well,
you can often make simultaneous edits to the same file(s), especially ones that
are mostly human-readable (think: text files, not images or binary files).

[3]: https://git-scm.com/
[4]: https://www.mercurial-scm.org/

Additionally, there are existing popular communities, like [GitHub][5], where
you can host your source code for free, and use a lot of free web-based tools
to collaborate. For example, NASA (yes, _the_ NASA) hosts their Open MCT
dashboard on GitHub: <https://github.com/nasa/openmct>.

[5]: https://github.com

![History of OpenMCT](https://user-images.githubusercontent.com/168174/111923604-31618b80-8a5d-11eb-80af-772bc8c7af37.png)

## The problems with source control and TTS

If you've made it this far, you might be asking yourself - OK, that all sounds
interesting - or maybe if you a prior user of Git/GitHub might not understand
exactly why Tabletop Simulator is _not_ a natural fit. Well, that's because of
the _save file format_ of TTS itself - it's a huge monolithic JSON file.

Here are the first ~30 or so lines of over 50K lines of the mod I maintain:

```json
{
  "SaveName": "StarWarsLegion",
  "GameMode": "Star Wars: Legion",
  "Date": "3/5/2021 11:16:45 AM",
  "VersionNumber": "v13.0.5",
  "GameType": "",
  "GameComplexity": "",
  "Tags": [],
  "Gravity": 0.5,
  "PlayArea": 0.5,
  "Table": "Table_None",
  "Sky": "Sky_Museum",
  "Rules": "",
  "Grid": {
    "Type": 0,
    "Lines": false,
    "Color": {
      "r": 0.0,
      "g": 0.0,
      "b": 0.0
    },
    "Opacity": 0.75,
    "ThickLines": false,
    "Snapping": false,
    "Offset": false,
    "BothSnapping": false,
    "xSize": 6.0,
    "ySize": 6.0,
    "PosOffset": {
      "x": 2.0,
      "y": 1.0,
      "z": 0.0
    }
  }
}
```

As soon as more than one person (or even just one person, if you use
[feature branching][6] as a strategy) are working on the mod concurrently, you
now have a problem - how can you tell if an edit is incompatible with another
one? For a while, the answer was "I did it manually". This was a rough process,
and often required patching, re-patching, manual testing, and sometimes even
partial rewrites of contributed features.

Despite adding 2 more developers to our open source team we were spending more
and more time merging changes and less and less time contributing features and
bug fixes:

![A typical edit](https://user-images.githubusercontent.com/168174/111923817-41c63600-8a5e-11eb-999f-bc920c0398d0.png)

[6]: https://guides.github.com/introduction/flow/

So, I developed a system of free open-source NodeJS packages to try and solve
our workflow problems, and to work collaboratively on GitHub, together, like
a typical open source project. I went through many iterations, and landed on
a series of disjoint packages.

> 🔥 **Why NodeJS?**
>
> The short answer is I wanted to choose a language (JavaScript) where I thought
> non-developers would have the best chance of understanding and using what I
> made - that meant "fun" languages like Rust/Go/Kotlin were all out of
> question.
>
> If you like my approach, but have a favorite language you'd like to write
> tools in know that nothing here _requires_ JavaScript (or Node), but of course
> if you want use my packages as-is you'll need them.

## Enter `tts-save-files` & `tts-expander`

The first package I developed was called [`tts-save-files`][7]. This is, to be
honest, the most boring package - I created a series of JSON Schema files to
describe the metadata format of Tabletop Simulator save files:

```json
{
  "$id": "https://tts.swlegion.dev/CameraState.json",
  "$schema": "http://json-schema.org/draft-07/schema",
  "title": "CameraState",
  "type": "object",

  "properties": {
    "Position": {
      "$ref": "./VectorState.json"
    },

    "Rotation": {
      "$ref": "./VectorState.json"
    },

    "Distance": {
      "type": "number"
    },

    "Zoomed": {
      "type": "boolean",
      "default": false
    },

    "AbsolutePosition": {
      "$ref": "./VectorState.json"
    }
  },

  "additionalProperties": false,
  "required": ["Position", "Rotation", "Distance", "Zoomed"]
}
```

[7]: https://github.com/matanlurey/tts-save-format

... and using the [`json-schema-to-typescript`][8] package, auto-generated
TypeScript definition files to use in future tooling:

```ts
/* Generated from CameraState.json */
export interface CameraState {
  Position: VectorState;
  Rotation: VectorState;
  Distance: number;
  Zoomed: boolean;
  AbsolutePosition?: VectorState;
}
```

[8]: https://www.npmjs.com/package/json-schema-to-typescript

This whole project made me much more familiar with the Tabletop Simulator
save file format, so even though it ended up not strictly being necessary, it
was a fun project and has been relatively stable.

Next I created [`tts-expander`][9], the "meat and potatoes" of this approach:

```md
The goals of this library are:

- Avoid storing a giant monolithic `JSON` blob in version control (i.e. GitHub).
- Ease collaboration and code reviews of changes.
- Make it easier to hand-edit files/properties/scripts when desired.
- Allow the development of more advanced editors and tooling for modding.
```

[9]: https://github.com/matanlurey/tts-expander

Basically, it takes a single `TableTopSave.json`, and generates up to 100s of
standalone `.json` (and `.lua`) files, in a directory structure that mirrors
the original file, and retains some extra metadata:

```json
{
  "ContainedObjectPaths": [
    "Roll_Out.f33bcc.json",
    "Hemmed_In.df6ba7.json",
    "Danger_Close.4c75d5.json",
    "Battle_Lines.3ac84e.json",
    "The_Long_March.88029d.json",
    "Major_Offensive.b758f8.json",
    "Disarray.4369e7.json",
    "Advanced_Positions.919d08.json"
  ]
}
```

This allows us to store _every individual object_ in a Tabletop Simulator save
file as a _separate_ file, which means that if someone makes an edit _just_ to
`The_Long_March.88029d.json` it won't have a source control conflict with any
other part of the mod.

The tooling also provides the reverse functionality - taking a directory of
"split" files and combining ("compiling") them back into the single JSON file
format the Tabletop Simulator expects. In our mod, you use `npm run compile`.

With this approach, we now are able to collaborate and merge code without
coordinating on who is making edits (at least, generally speaking), and it works
a lot more like a typical open source project - we don't even store the "full"
JSON file anymore!

## Making it better: `tts-runner`

It would have been sensible to stop there, but I developed yet-another project
to make our development workflow better, [`tts-runner`][10] - which is a simple
NodeJS library for automatically launching TTS (programmatically) and for
finding the "Saves" folder (at least on Windows, more OS support will require
contributions):

```ts
import * as steam from "@matanlurey/tts-runner/steam_finder";

// An example of using tts-runner
async function createSymlink(homeDir?: string): Promise<string> {
  // TODO: Add non-win32 support.
  if (!homeDir) {
    if (os.platform() !== "win32") {
      throw new Error(`Unsupported platform: ${os.platform()}`);
    }
    homeDir = steam.homeDir.win32(process.env);
  }
  await destroySymlink(homeDir);
  const from = path.join(homeDir, "Saves", "TTSDevLink");
  await fs.symlink(
    path.resolve("dist"),
    from,
    os.platform() === "win32" ? "junction" : "dir"
  );
  return from;
}
```

With the above code, I automatically create a symbolic link pointing to the
output of `npm run compile` and place that link in the `Saves` folder so that
you can easily use Tabletop Simulator to load the "compiled" mod.

[10]: https://github.com/matanlurey/tts-runner

## Going even further: `tts-editor`

And of course, I wanted more, so I wrote [`tts-editor`][11], a NodeJS
implementation of Tabletop Simulator's [External Editor API][12].

[11]: https://github.com/matanlurey/tts-editor
[12]: https://api.tabletopsimulator.com/externaleditorapi/

```ts
(async () => {
  // Create an API client and listen to incoming messages.
  const api = new ExternalEditorApi();
  await api.listen();

  // You are now ready to send/receive messages.
  await api.executeLuaCode('print("Hello, World")');
})();
```

Currently I use this library to automatically "reload" the mod on compile!

## Some unsolved problems

I wish I could end with saying "here is a bullet-proof way of making a mod" with
a simple template others could follow for success. Unfortunately, I didn't have
the free time (yet) to do this, but I'm hoping the steps in this blog can be
followed by others, and I'm also happy to answer questions about it on
[Reddit][13] or [via email][14].
Some unsolved problems I'd like to still resolve:

- TTS (and Unity) floating point numbers - whenever the save file is edited by
  TTS itself, floating point numbers will change - just a tiny bit - across the
  entire mod. These can normally just be ignored, but I'd love to build a script
  to do this automatically.

- A way to run TTS, headless, on GitHub actions - I want to be able to, on every
  PR, take a screenshot or screen-cast of the mod running. This would add a nice
  low-bar test for contributions.

- A standard Lua formatter that I can run on the command-line.

[13]: https://www.reddit.com/user/decafmatan
[14]: mailto:blog@lurey.dev
