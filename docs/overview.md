# Theremin Emulator - Overview

## What It Is

A software theremin synthesizer built with the JUCE framework in C++20. It produces a sine wave whose frequency and amplitude are controlled in real time via on-screen sliders. Available as a **VST3 plugin**, **AU plugin**, and **standalone application**.

- **Company**: hooklineAudio
- **Plugin Code**: `Reg0` / **Manufacturer Code**: `Zah1`
- **Version**: 0.0.1

## Project Structure

```
Theremin/
  CMakeLists.txt
  README.md
  Source/
    PluginProcessor.h/.cpp   # Audio engine and parameter management
    PluginEditor.h/.cpp      # GUI
    SineWave.h/.cpp          # Sine wave oscillator
  Resources/
    theremin.jpg             # Background image for the UI
  deliverables/
    ThereminEmulator.zip     # Compiled plugin
```

## Architecture

The plugin follows JUCE's standard processor/editor split:

### SineWave (signal generator)

Generates a sine wave with per-sample phase accumulation. Each audio channel maintains its own phase accumulator to stay numerically stable (phase wraps at 2pi).

Three `SmoothedValue` parameters prevent audible clicks:

| Parameter | Smoothing Type   | Ramp Time |
|-----------|------------------|-----------|
| Frequency | Multiplicative   | 50 ms     |
| Amplitude | Linear           | 10 ms     |
| Power     | Linear           | 250 ms    |

### AudioPluginAudioProcessor (host interface)

Manages the plugin's three parameters via `AudioProcessorValueTreeState`:

| ID     | Range          | Default | Scale |
|--------|----------------|---------|-------|
| freqHz | 20 - 15,000 Hz | 220 Hz  | Log (skew centre 1000 Hz) |
| amp    | 0.0 - 1.0     | 1.0     | Linear |
| play   | on / off       | on      | Boolean |

Parameters are read through atomic floats/bools so the audio thread never blocks.

### AudioPluginAudioProcessorEditor (GUI)

A 710 x 947 window with a theremin photograph as the background. Controls:

- **Frequency slider** (horizontal) - shows the current Hz value as an integer.
- **Amplitude slider** (vertical, left side).
- **Play button** (toggle) - navajowhite when on, maroon when off.

All controls are wired to the processor's parameter state through JUCE attachments so automation and preset recall work automatically.

## Audio Processing Flow

1. Host calls `prepareToPlay()` - SineWave is initialised with the sample rate and channel count.
2. Each audio block:
   - Atomic parameter values are loaded (lock-free).
   - Target values are set on the SineWave smoother.
   - `SineWave::process()` fills the buffer sample-by-sample: `output = power * amplitude * sin(phase)`.
3. `ScopedNoDenormals` is active during processing to avoid CPU denormal slowdowns.

## Build

Requires **CMake 3.24+**. JUCE is fetched automatically via CPM.

```bash
cmake -B build -S .
cmake --build build
```

Linked JUCE modules: `juce_audio_basics`, `juce_audio_devices`, `juce_audio_utils`, `juce_audio_processors`, `juce_core`, `juce_graphics`, `juce_gui_basics`, `juce_gui_extra`, `juce_data_structures`, `juce_dsp`, `juce_analytics`.

## Design Notes

- **No MIDI** - the plugin is controlled entirely through its parameters/UI.
- **IS_SYNTH is false** in the CMake config despite generating audio.
- Stereo output is a duplicated mono signal (both channels share the same sine wave, with independent phase accumulators).
- The frequency slider uses a logarithmic skew so the musically useful range is easier to navigate.
