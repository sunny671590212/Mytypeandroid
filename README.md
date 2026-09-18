# MyType V2

MyType is a native Android prototype for creating a personal handwriting set and using it in a custom IME.

## V2 features
- Persistent A–Z handwriting samples stored locally on-device.
- QWERTY live preview using saved glyph images.
- Space, delete, clear, HELLO demo.
- Native Android InputMethodService.
- QWERTY keyboard with shift, numbers/symbols, space, delete, enter.
- Saved handwriting glyphs displayed on alphabet keys when available.
- Settings shortcut and keyboard picker.

## Important platform limitation
Android IMEs normally send Unicode text to other apps; they cannot universally replace the receiving app's font with a private handwritten font. Therefore V2 keeps the real system keyboard text-compatible while providing a handwriting preview. A later release can add image/content insertion for compatible apps and a font-generation/export feature.

## Build
Open this folder as an Android project in Android Studio or another Android Gradle build environment and build `app`.
