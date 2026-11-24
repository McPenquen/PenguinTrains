# Android Dev Guide
This document dives into how android works for games development and how its software and hardware work and how they are meaning to be worked with.

## Important mobile dev specific concerns and solutions
* Hardware differs wildly
    * System on Chip (SoC) - similar to motherboard combines several components into one circuit (CPU, GPU, Memory, Connectivity, I/O). Most popular: Qualcomm, MediaTek, Samsung, Google
    * RAM - 2-16GB
    * OS versions
    * Screen
* Thermal throtting & Mobile GPU
    * Mobile devices overheat easily - no proper cooling system
    * Mobile devices use Tile-Based Deferred Rendering - GPU rendering architecture splitting screen into smaller grid tiles and postpones pixel shading until visibility is resolved for each tile. It saves memory and power by keeping data in fast on-chip memory and skipping shading for hidden opaque pixels
    * Heavy overdraw, high resolution post-processing passes, dense micro triangles preoccupy the RAM bandwidth and so cause thermal throtting
* Frame pacing & Variable Refresh Rate
    * Mobile devices require frame synchronisation (with the Choreographer - *using VSync coordinates the timing of input, animations and drawing on display*, and SurfaceFlinger - android system compositor service that accepts graphic buffers from apps arranges them into layers and sends them to display)
    * The goal of synchronising frame pacing is to minimize fram stutter and input latency
* Aggressive Lifecycle & MEmory Contraints
    * Android manages memory by terminating BG processes when foreground app needs ram
    * Games need to handle OnPause, OnStop, OnTrimMemory
    * Blocking the UI thread for more than 5s triggers Application Not Responding (ANR)
* C++ vs Kotlin
    * high-performance game loops run in C++/C via Android NDK
    * Android OS features (permissions, soft keyboard, window insets, google play services - billing, play games services) are in Java/Kotlin
    * Java Native Interface (JNI) - is the bridge between the two languages. Moving data across JNI carelessly introduces thread-safety bugs, life cycle races and overhead
* Disrtibution size, asset delivery & play vitals
    * Games are big and cellular downloads could suffer, therefore games are submitted to android store in Android App Bundles (.aab)
    * Play Asset Delivery - system that hosts, manages and delivers large game assets (textures, shaders, models) using .aabs
        * splits assets into 3 delivery modes: 
            * install-time: installed during the install app stage
            * fast-follow: downloaded shortly after the app finishes downloading
            * on-demand: only downloaded when the app explicitly requests it
        * target device specific compression formats (ASTX vs ETC2)
        * Managest ANR thresholds in google play consoles android vitals

    
## Known Game Dev Approaches for Android
* One approach popular among developers is to use crossplatform engines: Unity, UE5, Godot, Defold, Cocos
    * Requires Java/Kotlin bridges to communicate to Google Play features and custom Android manifest permissions and similar
* Other developers could also develop their custom c++ engine using Android Game Development Kit
    * Core AGDK libraries:
        * Swappy - synchronizes OpenGL/Vulkan frame representeation with the display refresh rate to eliminate stutter
        * Tuning Fork - Correlates frame-time telematery with device models and in-game quality settings in google play console
        * Oboe - low latency audtio library wrapping AAudio and OpenSL
        * Paddleboat & GameTextInput - standardized c++ controller mapping and soft-keyboard handling
* Pure Kotlin/JAva + Android framework APIs
    * No c++, does not need JNI, all runs on Android run time
* Web/Hybrid wrappers - HTML5/WebGL/WebGPU/etc
    * browser based wrapper inside  of a lightweight native android shell (Capacitor or custom WebView)
    * developed in Type/JavaScript and communicates via translations to Java interfaces