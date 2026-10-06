# Connect3Dp and Handy3D

A vendor-agnostic way to watch and control 3D printers, plus a phone app in front of it.

- Library and host: [AaronFJung/Connect3Dp](https://github.com/AaronFJung/Connect3Dp)
- Phone app: [AaronFJung/Handy3Dp](https://github.com/AaronFJung/Handy3Dp)

This page does not republish either tree. Packet layouts are not included.

## The whole system

Each printer brand speaks a different protocol. Connect3Dp hides that behind one machine model. A host process keeps the connections open. The phone app talks to the host, not to each vendor SDK.

```mermaid
flowchart TB
  phone[Handy3D mk1]
  ws[Connect3Dp WebSocket host]
  lib[Lib3Dp]
  elegoo[Elegoo SDCP]
  bambu[Bambu Lab MQTT and FTP]
  creality[Creality K1C]
  phone --> ws
  ws --> lib
  lib --> elegoo
  lib --> bambu
  lib --> creality
```

**Lib3Dp** is the C# library. A connector subclasses `MachineConnection` and implements the protocol. The base class owns timeouts, scheduling, and state diffs. The connector overrides `Connect_Internal`, `Pause_Internal`, `Resume_Internal`, `Stop_Internal`, `PrintLocal_Internal`, and whatever else that brand can do. It reports what it supports with `MachineCapabilities`. The host only exposes operations the capability flags allow.

State changes go through `CommitState`. That applies the diff, runs scheduling, and notifies listeners. Callers get a `MachineOperationResult` instead of an exception for ordinary device failures.

**Connect3Dp.Host** is the ASP.NET Core process. It stores machine configs as JSON and print files on disk. Clients subscribe over a WebSocket. Topics cover configuration, live state, logs, pause, resume, stop, idle, and spool matching. A state change on a printer is broadcast to subscribed clients.

**Why this holds together:** the phone, and any later farm UI, only know `MachineStatus` and the shared topics. Adding a brand means a new connector, not a new app.

Brands in the library today: Elegoo Centauri Carbon, Bambu Lab, and Creality K1C.

## Elegoo

`ELEGOOMachineConnector` adapts Elegoo's SDCP protocol onto that shared model. The printer is a Centauri Carbon or Centauri Carbon 2, addressed by host name or IP. The connector opens a WebSocket, keeps a heartbeat, and polls status.

```mermaid
flowchart LR
  cfg[Nickname, model, serial, host]
  sock[WebSocket]
  poll[Status poll and heartbeat]
  map[Temperatures, fans, lights, print info]
  state[CommitState to MachineState]
  cfg --> sock --> poll --> map --> state
```

Commands the connector sends include start, pause, resume, and stop; file list, upload, and delete; history and history video; the video-stream toggle; lights; and fan speed. Incoming status values (`Printing`, `Suspended`, `Completed`, `Stopped`, and the rest of the SDCP status set) are mapped onto the shared `MachineStatus` enum, so the host treats an Elegoo like the other brands.

The protocol was recovered from the official web UI's network traffic. The command numbers live in `ELEGOOCmd`. They are not repeated here.

## Handy3D mk1

Expo app for a phone on the same network as the host. Display name in the app is Handy3D mk1. Package name in the tree is Farm3Dp.

```mermaid
flowchart TB
  home[Home: discovered and saved printers]
  settings[Settings]
  tabs[Printer tabs]
  home --> tabs
  settings --> home
  tabs --> status[Live status]
  tabs --> files[Files and start print]
  tabs --> history[History]
  tabs --> controls[Pause, resume, stop, lights]
  tabs --> filament[Filament slots and AMS match]
  status --> client[vendored Connect3Dp WebSocket client]
  files --> client
  controls --> client
  filament --> client
  client --> host[Connect3Dp host]
```

**Stack:** Expo 54, React Native 0.81, React 19, expo-router, NativeWind, React Navigation, AsyncStorage, lucide icons. The websocket client is the `connect3dp` package under `vendor/connect3dp`.

| Home | Printer |
|---|---|
| ![Handy3D home with a Centauri Carbon printing](docs/media/home-connected.png) | ![Live camera, job progress, temperatures, pause and stop](docs/media/printer-status.png) |

| Files on the printer | Filament |
|---|---|
| ![Local gcode files from Connect3Dp](docs/media/local-files.png) | ![AMS 2 Pro slots, colors, and humidity](docs/media/filament.png) |

The home screen groups printers by brand. A Centauri Carbon shows up under Elegoo once the phone is connected to the Connect3Dp host. Opening it shows the live camera, chamber light, model fan, the running job, nozzle / bed / chamber temperatures, and pause and stop.

Local files are the jobs Connect3Dp already has on the printer. Print history is the completed-job tab.

![Print history](docs/media/print-history.png)

The filament screen is the AMS: slot color, material, humidity, and heat. The job card shows which spool is active.

![Job card with nozzle, bed, chamber, and the active AMS slots](docs/media/print-progress.png)

Bambu Lab printers use the same list. A machine can show several jobs, each marked printing or finished.

![Bambu Lab jobs on the same phone app](docs/media/bambu-jobs.png)

Settings stores the Connect3Dp WebSocket address used for discovery. The default is `ws://localhost:5000/ws`.

![Discovery server address](docs/media/settings.png)

## License

This note is separate from the two source repos. All rights reserved.
