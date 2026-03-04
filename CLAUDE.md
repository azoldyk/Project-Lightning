# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Project Lightning is a WPF desktop application (C#, .NET Framework 4.8) that delivers game fixes, enhancements, and optimizations. It targets Windows 10/11 and supports multiple game publishers (Ubisoft, EA, Rockstar, Denuvo, PlayStation, others).

## Build Commands




```bash
# Restore NuGet packages
nuget restore "Project Lightning.sln"

# Debug build
msbuild "Project Lightning.sln" /p:Configuration=Debug

# Release build
msbuild "Project Lightning.sln" /p:Configuration=Release
```

Output goes to `bin\Debug\` or `bin\Release\`. No test framework or CI/CD pipeline is configured.

## Architecture

### Navigation Model
`App.xaml` → `MainWindow` → Frame-based page navigation. MainWindow hosts a Frame that loads Pages, a collapsible sidebar, a video background (`MediaElement`), and a notification canvas. Navigation history is explicitly disabled.

### Key Layers
- **Pages/** — Main navigable views loaded into MainWindow's Frame. Each page receives a reference to MainWindow for navigation. The largest is `panelOnlineFix.xaml.cs` (~800 lines) with pagination (18 games/page).
- **Windows/** — Modal dialogs (error, confirmation, download progress, legal notices). All use WPF `ShowDialog()` with owner set to MainWindow.
- **UserControls/Cabecera.xaml** — Shared header bar with publisher navigation buttons. Pages subscribe to its routed events (`HomePresionado`, `UbisoftPresionado`, etc.).
- **Classes/** — Utilities: `Actualizador` (auto-update via GitHub), `NotificationManager` (toast notifications with animations), and three XAML value converters (`ScoreToColorConverter`, `NullToVisibilityConverter`, `ExpandTextConverter`).

### Data Flow
- **data.json** — Game catalog structured as `{ "PUBLISHER": { "APP_ID": { ...game metadata } } }`. This is the primary data source for game listings.
- **shop.json** — Store items with normal/donor pricing and image URLs.
- **Properties/Settings.settings** — Persisted user settings (DMCA version tracking, Steam path, etc.).

### External Dependencies
- **TokenProvider.dll** and **Microsoft.Content.Area._3D.dll** — External project DLLs referenced from sibling directories (`../TokenProvider/`, `../Microsoft.Content.Area._3D/`).
- **Steam API** — `steam_api.dll` and `steam_api64.dll` copied to output on build.
- **Key NuGet packages**: Newtonsoft.Json (JSON), SharpCompress (archives), WpfAnimatedGif (GIF support), System.Data.SQLite, ZstdSharp (Zstandard compression).

## Conventions

- **Namespace**: `Project_Lightning`
- **Naming**: Spanish names for many UI elements (e.g., `panelMenuPrincipal`, `framePrincipal`, `ventanaPrincipal`, `panelAjustes` = settings, `panelTienda` = store, `panelBiblioteca` = library). Code comments are in Spanish.
- **Inter-page communication**: Event handlers on `Cabecera` UserControl; pages hold a `MainWindow` reference for navigation calls.
- **Version tracking**: Assembly version in `Properties/AssemblyInfo.cs`; remote version in `latest-version.txt` checked by `Actualizador.cs`.
- **Resources**: Media files (videos, logos, icons, GIFs, fonts) live under `res/`. Video backgrounds and promos use `CopyToOutputDirectory=Always`.
