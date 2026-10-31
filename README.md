# CSE 489 Assignment 2 — Android Jetpack Demo

This is a complete Kotlin Android project demonstrating a navigation drawer, Jetpack fragments, dynamic broadcast receivers, pinch-to-zoom gestures, video playback, and audio playback.

## Run the project

1. Open the inner `CSE489_Assignment2` directory in Android Studio.
2. Allow Gradle sync to finish.
3. Run the `app` configuration on an API 24+ emulator or Android device with internet access.
4. Open the drawer with the toolbar menu icon and select a feature.

The remote image and audio samples require an active internet connection. Video is bundled in the APK so it works offline. No runtime permission is necessary because `INTERNET` is a normal manifest permission.

## Architecture and project structure

The app uses one activity and a fragment-based UI:

```text
app/src/main/
├── AndroidManifest.xml
├── java/com/example/cse489_assignment2/
│   ├── MainActivity.kt
│   ├── broadcast/
│   │   ├── BroadcastContract.kt
│   │   ├── BroadcastSelectionFragment.kt   # Spinner / first screen
│   │   ├── BroadcastDataFragment.kt        # Text input or battery / second screen
│   │   ├── BroadcastResultFragment.kt      # Received result / third screen
│   │   └── CustomMessageReceiver.kt
│   └── features/
│       ├── ImageScaleFragment.kt
│       ├── VideoFragment.kt
│       └── AudioFragment.kt
└── res/
    ├── layout/                              # Activity and fragment XML layouts
    └── menu/drawer_menu.xml                 # Four drawer destinations
```

`MainActivity` owns the `DrawerLayout`, `NavigationView`, toolbar, and fragment container. Jetpack `FragmentManager` handles the three-screen back stack, drawer destinations, titles, the hamburger icon, and Up navigation.

## Requirement mapping

### Broadcast Receiver

`BroadcastSelectionFragment` presents the two required values in a real Android `Spinner`. It passes a small enum value through a fragment argument.

For the custom path, `BroadcastDataFragment` validates the text and passes it to `BroadcastResultFragment`. The result fragment dynamically registers `CustomMessageReceiver` with `RECEIVER_NOT_EXPORTED`, sends a package-scoped custom broadcast, and displays the received extra. Registration occurs in `onStart` and cleanup in `onStop`, preventing receiver leaks.

For the battery path, `BroadcastDataFragment` dynamically listens for the sticky `Intent.ACTION_BATTERY_CHANGED` broadcast and computes the percentage from `BatteryManager.EXTRA_LEVEL` and `EXTRA_SCALE`. The third screen displays the permitted completion placeholder.

### Image Scale

An asynchronous `HttpsURLConnection` downloads the image without blocking the main thread. `ScaleGestureDetector` multiplies the `ImageView` scale for each pinch gesture and clamps it to 0.75×–5×. The current scale and a reset action are shown beneath the image.

### Video

`VideoFragment` plays the bundled `res/raw/demo_video.mp4` in `VideoView`, avoiding network and streaming-codec failures. `MediaController` supplies play, pause, seek, and scrub controls. The fragment saves the position across configuration changes, pauses when backgrounded, and calls `stopPlayback()` when its view is destroyed.

### Audio

`AudioFragment` prepares a streaming `MediaPlayer` asynchronously and exposes Play, Pause, and Stop. Stop invokes `MediaPlayer.stop()` and asynchronously prepares it again from the beginning. The player pauses when the fragment leaves the foreground and is always released in `onDestroyView`.

## Manifest and dependencies

`AndroidManifest.xml` declares only `android.permission.INTERNET`. Both receivers are dynamic, so no manifest `<receiver>` entry is needed. This is preferable for lifecycle-bound UI demonstrations and is compatible with modern Android broadcast restrictions.

Major dependencies are AndroidX AppCompat, Fragment, DrawerLayout, and Material Components. View Binding is enabled so Kotlin code never relies on unsafe `findViewById` casts.

## Demonstration checklist

1. Open **Broadcast Receiver**, choose **Custom broadcast receiver**, enter text, and verify it appears on screen three.
2. Return, choose **System battery notification receiver**, and verify the live percentage appears before continuing.
3. Open **Image Scale** and pinch with two fingers; use **Reset** to return to 1×.
4. Open **Video**, tap the player, and use the `MediaController` controls.
5. Open **Audio** and demonstrate Play, Pause, and Stop.

For a fully offline submission, also replace the image and audio URL constants with files in `res/drawable` and `res/raw`.
