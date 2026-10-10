# Lustral Test Players

Prebuilt Unity players that [Lustral](https://github.com/LustralTeam/Lustral)'s end-to-end tests run against. Lustral's
CI downloads them from this repository's releases, so that it needs neither a Unity Editor nor a Unity license. The
players are minimal Unity games made by LustralTeam; they contain no mods and no part of Lustral.

## Releases

Each release, named by its date, contains one zip per player and `players.json`:

| File | Content |
| --- | --- |
| `<unity version>-<platform>.zip` | The player, extracted to `<unity version>/<platform>/` |
| `players.json` | The release, and each player's Unity version, platform, file name, size and SHA-256 hash |

Platforms are named as Lustral's release packages: `win-x64-mono` and `win-x64-il2cpp`, and `win-x64-mono-legacy` for a
Unity 2018 player that uses the legacy .NET 3.5 scripting runtime. Lustral names the release it tests against in its
`tests/testplayers.json`, and `tools/get-testplayers.ps1` downloads it.

## Unity Versions

| Unity | Why it is tested |
| --- | --- |
| 6000.6.5f1 | IL2CPP metadata v108; a class's interfaces in a single array; the player keeps Unity's player loop API |
| 6000.5.4f1 | IL2CPP metadata v107; a heavily stripped player without the player loop API |
| 2022.3.62f2 | IL2CPP metadata v31 |
| 2021.3.45f2 | The 2021.3.40 and later variant of IL2CPP metadata v31 |
| 2018.4.36f1 | The oldest supported generation: IL2CPP metadata v24, the experimental player loop API, .NET Standard 2.0; also a player with the legacy .NET 3.5 runtime, which Lustral does not support |

## Behavior

The players are controlled from the command line with `-lustral-test <scenario>`:

| Scenario | Behavior |
| --- | --- |
| `idle` (default) | Runs until it is closed. |
| `quit` | After 60 frames, clicks the GameObject named "Lustral e2e" if one exists (see below); after 120 frames, loads the scene `Second`; after 240 frames, quits normally. |
| `crash` | After 120 frames, crashes with an access violation through Unity's `Utils.ForceCrash`, which Unity's crash handler catches. |
| `throw-on-thread` | After 120 frames, throws an unhandled exception on a new thread. |

The click reaches a component that implements `IPointerClickHandler` in two ways, as a game's UI code would: the player
finds it with `GetComponent<IPointerClickHandler>()` and calls `OnPointerClick` with a click count of 1, then sends a
click through `ExecuteEvents.Execute` with a click count of 2. It logs the result to the player log.

Other properties that the tests rely on:

- **Scenes.** `Main` (index 0) contains the camera and the test driver, a MonoBehaviour named
  `Lustral.TestPlayer.TestDriver` with `Awake` and `Update` methods; `Second` (index 1) contains a camera.
- **Player settings.** Company "Lustral", product "Lustral Test Player", version 1.0, windowed at 640 × 360, running in
  the background, at 60 frames per second.
- **Unity UI.** The players include Unity UI (`com.unity.ugui`) for the click.
- **Asset bundle.** `e2e/lustral-e2e.bundle`, built with each player: a text asset `note.txt` ("Hello from an asset
  bundle"), a mesh `triangle`, a texture `atlas` with the sprites `left` and `right`, and a prefab `Lustral Cube` with a
  child `Lid`.
- **IL2CPP stripping.** The IL2CPP players keep `GameObject` and `TextAsset` entirely (link.xml) and are otherwise
  stripped as Unity does by default. They do not contain Unity's `AssetBundle` class.

## License

The players are built from LustralTeam's Unity project, which is not public, and are published for testing Lustral.
They contain Unity's runtime, which is subject to the [Unity Terms of Service](https://unity.com/legal/terms-of-service).
Lustral is not affiliated with Unity Technologies.
