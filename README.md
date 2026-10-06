# Elegoo printer work

University lab work on the Elegoo Centauri Carbon, inside two repos I do not own:

- Protocol and connector: [AaronFJung/Connect3Dp](https://github.com/AaronFJung/Connect3Dp)
- Phone app: [AaronFJung/Handy3Dp](https://github.com/AaronFJung/Handy3Dp)

I am not republishing either tree. The lab funded the project. This page is only what my commits do.

## Connect3Dp — Elegoo connector

My commits on that repo:

| Commit | What it is |
|---|---|
| [`3872187`](https://github.com/AaronFJung/Connect3Dp/commit/3872187) | `ELEGOOMachineConnector` plus an `Elegoo3Dp` collection of captured calls |
| [`711fbd0`](https://github.com/AaronFJung/Connect3Dp/commit/711fbd0) | Follow-up on that connector |
| [`ed4ec56`](https://github.com/AaronFJung/Connect3Dp/commit/ed4ec56) | More connector behavior, a small JS client surface, and two sample gcode files |
| [`c55e5ce`](https://github.com/AaronFJung/Connect3Dp/commit/c55e5ce) | Merge from the old `Lorttexwolf/Connect3Dp` remote |

The connector speaks Elegoo's SDCP protocol over a WebSocket. `ELEGOOCmd` in `Lib3Dp/Connectors/ELEGOO/ELEGOOConstants.cs` is the command table I added: status and attributes, start / pause / stop / resume, file list / detail / delete, history and history video, and the video-stream toggle. The C# class also uploads a file, toggles lights, and sets fan speed, then maps printer status onto the shared `MachineStatus` enum so the rest of Connect3Dp can treat an Elegoo like the other brands.

I got those calls by watching the official web UI's network traffic. I am not pasting the packet layouts here.

`ed4ec56` also commits `ABSWallHook.gcode` and `CanHolder.gcode`. Those are print files, not documentation.

## Handy3Dp — the phone app

Expo / React Native app (`expo-router`). My commits:

| Commit | What it is |
|---|---|
| [`332a7e8`](https://github.com/AaronFJung/Handy3Dp/commit/332a7e8) | Printer UI: home, settings, live status, files, history, controls, filament, and the Connect3Dp client wiring |
| [`2746234`](https://github.com/AaronFJung/Handy3Dp/commit/2746234) | AMS spool matching, and normalization of the websocket payloads before the UI reads them |
| [`afe3fc8`](https://github.com/AaronFJung/Handy3Dp/commit/afe3fc8) | Turn the Elegoo camera stream on from the phone. Other brands do not get that path |

Aaron J owns the repo (2 commits beside mine). The app talks to a printer through the Connect3Dp websocket client vendored under `vendor/connect3dp`.

```
phone (Expo)
  home / printer tabs / filament / controls
    → Connect3Dp websocket client
      → printer on the LAN
         Elegoo path: SDCP commands + camera enable
```

## What is not in these repos

A co-authored lab paper is listed on my resume. It is not in either git history, so I am not linking it from here.

## License

This note is mine. Their repositories stay under their owners. All rights reserved.
