# Theremin Emulator

A simple audio plugin that emulates a theremin. Move your mouse around the UI to control pitch and volume, similar to how you'd move your hands around a real theremin.

Available as a **Standalone** app, **VST3**, and **AU** plugin.

<img width="707" height="965" alt="Theremin Emulator Screenshot" src="https://github.com/user-attachments/assets/6fae57bb-54e5-4140-ba46-9f253f250994" />

## Building

Requires CMake 3.24+ and a C++20 compiler. JUCE is pulled in automatically via CPM.

```bash
cmake -B build
cmake --build build
```

Built plugins are copied to your system plugin folders automatically.

## Tech

- C++ / JUCE framework
- Sine wave synthesis
- Builds to VST3, AU, and Standalone formats
