# 找不到作者了，暂时记录一下

# Awesome-Gamedev

## Contents
- [Renderer](#renderer)
- [Engine](#Engine)
- [Physically-based (Photorealistic) Rendering](#physically-based-photorealistic-rendering)
- [Graphics Example](#graphics-example)
- [DirectX](#directx)
- [Vulkan](#vulkan)
- [OpenGL](#OpenGL)
- [DXR](#dxr)
- [Small Gadget](#small-gadget)
- [C++](#c++)
- [Scripting](#Scripting)
- [Awesome Github Contributors](#awesome-github-contributors)
- [Other Awesome List](#other-awesome-list)


## Renderer
* [bgfx](https://github.com/bkaradzic/bgfx) - Cross-platform, graphics API agnostic, "Bring Your Own Engine/Framework" style rendering library. https://bkaradzic.github.io/bgfx/overview.html
* [The-Forge](https://github.com/ConfettiFX/The-Forge) - The Forge Cross-Platform Rendering Framework PC, Linux, Ray Tracing, macOS / iOS, Android, XBOX, PS4
* [oryol](https://github.com/floooh/oryol) - A small, portable and extensible C++ 3D coding framework
* [bsf](https://github.com/GameFoundry/bsf) - Modern C++14 library for the development of real-time graphical applications https://www.bsframework.io
* [LLGL](https://github.com/LukasBanana/LLGL) - Low Level Graphics Library (LLGL) is a thin abstraction layer for the modern graphics APIs OpenGL, Direct3D, Vulkan, and Metal
* [Falcor](https://github.com/NVIDIAGameWorks/Falcor) - Real-Time Rendering Framework https://developer.nvidia.com/falcor
* [RayEngine](https://github.com/stuffbydavid/RayEngine) - Rendering engine for CPU/GPU accelerated ray tracing
* [visionaray](https://github.com/szellmann/visionaray) - A C++ based, cross platform ray tracing library https://vis.uni-koeln.de/visionaray.html
* [minko](https://github.com/aerys/minko) - 3D framework for web, desktop and mobile devices. http://minko.io
* [kaleido3d](https://github.com/DsoTsin/kaleido3d) - Next Generation Renderer for Cross Platform Engine Development

## Engine
* [lumberyard](https://github.com/aws/lumberyard) - Amazon Lumberyard is a free AAA game engine deeply integrated with AWS and Twitch – with full source.
* [WickedEngine](https://github.com/turanszkij/WickedEngine) - C++ game engine focusing on modern rendering techniques. https://turanszkij.wordpress.com/
* [AtomicGameEngine](https://github.com/AtomicGameEngine/AtomicGameEngine) - The Atomic Game Engine is a multi-platform 2D and 3D engine with a consistent API in C++, C#, JavaScript, and TypeScript
* [xenko](https://github.com/xenko3d/xenko) - Xenko Game Engine https://xenko.com
* [BansheeEngine](https://github.com/BearishSun/BansheeEngine) - Modern C++14 game engine with Vulkan support, fully featured editor and C# scripting http://www.banshee3d.com
* [libgdx](https://github.com/libgdx/libgdx) - Desktop/Android/HTML5/iOS Java game development framework http://www.libgdx.com/
* [EtherealEngine](https://github.com/volcoma/EtherealEngine) C++ Game Engine and Editor
* [Directus3D](https://github.com/PanosK92/Directus3D) - Directus3D Game Engine
* [Serious-Engine](https://github.com/Croteam-official/Serious-Engine) - An open source version of a game engine developed by Croteam for the classic Serious Sam games.
* [DiligentCore](https://github.com/DiligentGraphics/DiligentCore) - This repository implements core functionality of Diligent Engine
* [DiligentEngine](https://github.com/DiligentGraphics/DiligentEngine) - Master repository for Diligent Engine project http://diligentgraphics.com/diligent-engine/
* [oxygine-framework](https://github.com/oxygine/oxygine-framework) - Oxygine is C++ engine and framework for 2D games on iOS, Android, Windows, Linux and Mac http://oxygine.org/
* [GamePlay](https://github.com/gameplay3d/GamePlay) - Open-source, cross-platform, C++ game engine for creating 2D/3D games. http://www.gameplay3d.io
* [LumixEngine](https://github.com/nem0/LumixEngine) - 3D Game Engine https://github.com/nem0/lumixengine_data
* [Torque3D](https://github.com/GarageGames/Torque3D) - MIT Licensed Open Source version of Torque 3D from GarageGames http://torque3d.org
* [Torque2D](https://github.com/GarageGames/Torque2D) - MIT Licensed Open Source version of Torque 2D game engine from GarageGames
* [Torque6](https://github.com/andr3wmac/Torque6) - Torque 6 is an MIT licensed 3D engine loosely based on Torque2D. Taking the best of Torque2D and adding a modern 3D rendering engine it aims to be a contender in the free and open-source 3D engine category.
* [gkEngine](https://github.com/gameknife/gkEngine) - A cross-platform game engine with cutting-edge real-time rendering tech & fantastic speed. http://gameknife.github.io

## Physically-based (Photorealistic) Rendering
* [IntegrateDFG](https://github.com/knarkowicz/IntegrateDFG) - Simple program which integrates DFG for split sum IBL approximation and saves calculated LUT as CSV or 2D DDS texture (RGBA_F32 and RG_16F). For more details see "Real Shading in Unreal Engine 4" by Brian Karis.
* [Flux](https://github.com/JulianThijssen/Flux) - A real-time physically based rendering engine written in C++ and OpenGL
* [PBR](https://github.com/Nadrin/PBR) - An implementation of physically based shading model & image based lighting in various graphics APIs.
* [pbrt-v3](https://github.com/mmp/pbrt-v3) - ource code for pbrt, the renderer described in the third edition of "Physically Based Rendering: From Theory To Implementation", by Matt Pharr, Wenzel Jakob, and Greg Humphreys. http://pbrt.org
* [Filament](https://github.com/google/filament) - Filament is a real-time physically based rendering engine for Android, Windows, Linux, macOS and WASM/WebGL
* [laugh_engine](https://github.com/jian-ru/laugh_engine) - A Vulkan implementation of real-time PBR renderer


## Graphics Example
* [HelloVulkan](https://github.com/GPUOpen-LibrariesAndSDKs/HelloVulkan) - Introductory Vulkan sample
* [MaskedOcclusionCulling](https://github.com/GameTechDev/MaskedOcclusionCulling) - Example code for the research paper "Masked Software Occlusion Culling"; implements an efficient alternative to the hierarchical depth buffer algorithm. https://software.intel.com/en-us/articles/masked-software-occlusion-culling
* [precomputed_atmospheric_scattering](https://github.com/ebruneton/precomputed_atmospheric_scattering) - This project provides a new implementation of our EGSR 2008 paper "Precomputed Atmospheric Scattering"
* [BakingLab](https://github.com/TheRealMJP/BakingLab) - A D3D11 application for experimenting with Spherical Gaussian lightmaps
* [AndroidPhysicallyBasedRendering](https://github.com/hak/AndroidPhysicallyBasedRendering) - Physically based renderer demo in Android
* [Spectral-BRDF-Explorer](https://github.com/chicio/Spectral-BRDF-Explorer) - OpenGL application inspired by Walt Disney Animation Studios BRDF Viewer. A BRDF Viewer that support standard RGB and spectral data (tristimulus values) color calculation. http://fabrizioduroni.it/
* [OpenGL-ES-3.0-Deferred-Rendering](https://github.com/GameTechDev/OpenGL-ES-3.0-Deferred-Rendering) - OpenGL ES 3.0 Deferred Renderer
* [GraphicsSamples](https://github.com/NVIDIAGameWorks/GraphicsSamples) - GameWorks cross-platform graphics API samples from NVIDIA
* [GPU-Pro-7](https://github.com/wolfgangfengel/GPU-Pro-7) - Source code repository for the Book GPU Pro 7

## DirectX
* [d3d12book](https://github.com/d3dcoder/d3d12book) - Sample code for the book "Introduction to 3D Game Programming with DirectX 12"
* [DirectX-Graphics-Samples](https://github.com/Microsoft/DirectX-Graphics-Samples) - This repo contains the DirectX Graphics samples that demonstrate how to build graphics intensive applications on Windows.
* [DXUT](https://github.com/Microsoft/DXUT) - DXUT is a "GLUT"-like framework for Direct3D 11.x Win32 desktop applications; primarily samples, demos, and prototypes. https://blogs.msdn.microsoft.com/chuckw/2013/09/14/dxut-for-win32-desktop-update/
* [FX11](https://github.com/Microsoft/FX11) - Effects for Direct3D 11 (FX11) is a management runtime for authoring HLSL shaders, render state, and runtime variables together. https://blogs.msdn.microsoft.com/chuckw/2012/10/23/effects-for-direct3d-11-update/
* [UVAtlas](https://github.com/Microsoft/UVAtlas) - UVAtlas isochart texture atlas https://blogs.msdn.microsoft.com/chuckw/2014/11/14/uvatlas-return-of-the-isochart/
* [DirectXMath](https://github.com/Microsoft/DirectXMath) - DirectXMath is an all inline SIMD C++ linear algebra library for use in games and graphics apps https://blogs.msdn.microsoft.com/chuckw/2012/03/26/introducing-directxmath/


## Vulkan
* [VulkanMemoryAllocator](https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator) - Easy to integrate Vulkan memory allocation library
* [PracticalVulkan](https://github.com/GameTechDev/PracticalVulkan) - Repository with code samples for "API without Secrets: The Practical Approach to Vulkan" series of articles.
* [pumex](https://github.com/pumexx/pumex) - Vulkan library oriented on high speed rendering
* [Vookoo](https://github.com/andy-thomason/Vookoo) - A set of utilities for taking the pain out of Vulkan in header only modern C++
* [VkHLF](https://github.com/nvpro-pipeline/VkHLF) - Experimental High Level Framework for Vulkan
* [vkDOOM3](https://github.com/DustinHLand/vkDOOM3) - Vulkan DOOM 3 port based on DOOM 3 BFG Edition
* [vkQuake](https://github.com/Novum/vkQuake) - Vulkan Quake port based on QuakeSpasm
* [MoltenVK](https://github.com/KhronosGroup/MoltenVK) - MoltenVK is an implementation of the high-performance, industry-standard Vulkan graphics and compute API, that runs on Apple's Metal graphics framework, bringing Vulkan to iOS and macOS.
* [Vulkan-Forward-Plus-Renderer](https://github.com/WindyDarian/Vulkan-Forward-Plus-Renderer) - Forward+ renderer in Vulkan using Compute Shader. An Upenn CIS565 final project.
* [VulkanSamples](https://github.com/LunarG/VulkanSamples) - Vulkan Samples from LunarG

## OpenGL
* [opengl-es-sdk-for-android](https://github.com/ARM-software/opengl-es-sdk-for-android) - OpenGL ES SDK for Android from ARM-Software
* [opengles3-book](https://github.com/danginsburg/opengles3-book) - OpenGL ES 3.0 Programming Guide Sample Code
* [OpenGL](https://github.com/McNopper/OpenGL) - Examples, OpenGL 3 and 4 with GLSL http://nopper.tv
* [iOS-OpenGLES-Stuff](https://github.com/jlamarche/iOS-OpenGLES-Stuff) - Various scripts, utils, and code examples for OpenGL ES programming for iOS
* [awesome-opengl](https://github.com/eug/awesome-opengl) - A curated list of awesome OpenGL libraries, debuggers and resources.
* [glfw](https://github.com/glfw/glfw) - A multi-platform library for OpenGL, OpenGL ES, Vulkan, window and input https://www.glfw.org/

## DXR
* [DxrTutorials](https://github.com/NVIDIAGameWorks/DxrTutorials) DxrTutorials from NVIDIA

## Small Gadget
* [imgui](https://github.com/ocornut/imgui) - Dear ImGui: Bloat-free Immediate Mode Graphical User interface for C++ with minimal dependencies
* [MetricsGui](https://github.com/GameTechDev/MetricsGui) - Library of ImGui controls for displaying performance metrics.
* [ImGuizmo](https://github.com/CedricGuillemet/ImGuizmo) - Immediate mode 3D gizmo for scene editing and other controls based on Dear Imgui
* [ImWindow](https://github.com/thennequin/ImWindow) - Window and GUI system based on Dear ImGui from OCornut
* [ImGuiColorTextEdit](https://github.com/BalazsJako/ImGuiColorTextEdit) - Colorizing text editor for ImGui
* [UnrealImGui](https://github.com/segross/UnrealImGui) - Unreal plug-in that integrates Dear ImGui framework into Unreal Engine 4
* [HLSLcc](https://github.com/Unity-Technologies/HLSLcc) - DirectX shader bytecode cross compiler from Unity
* [Anvil](https://github.com/GPUOpen-LibrariesAndSDKs/Anvil) - Anvil is a cross-platform framework for Vulkan
* [SPIRV-Reflect](https://github.com/chaoticbob/SPIRV-Reflect) - SPIRV-Reflect is a lightweight library that provides a C/C++ reflection API for SPIR-V shader bytecode in Vulkan applications.
* [ozz-animation](https://github.com/guillaumeblanc/ozz-animation) - Open source c++ skeletal animation library and toolset http://guillaumeblanc.github.io/ozz-animation/
* [slang](https://github.com/shader-slang/slang) - Making it easier to work with shaders
* [unreal.js-core](https://github.com/ncsoft/Unreal.js-core) & [unreal.js](https://github.com/ncsoft/Unreal.js) - Unreal.js plugin submodule
* [im3d](https://github.com/john-chapman/im3d) - Immediate mode rendering and 3d gizmos.
* [USD](https://github.com/PixarAnimationStudios/USD) - Universal Scene Description http://www.openusd.org
* [OpenSubdiv](https://github.com/PixarAnimationStudios/OpenSubdiv) - An Open-Source subdivision surface library. http://graphics.pixar.com/opensubdiv
* [Source-2-Decompiler](https://github.com/Dingf/Source-2-Decompiler) - A decompiler for the Source 2 file formats
* [SFML](https://github.com/SFML/SFML) - Simple and Fast Multimedia Library http://www.sfml-dev.org/
* [nuklear](https://github.com/vurtun/nuklear) - A single-header ANSI C gui library
* [XShaderCompiler](https://github.com/LukasBanana/XShaderCompiler) - Shader cross compiler to translate HLSL (Shader Model 4 and 5) to GLSL
* [FasTC](https://github.com/GammaUNC/FasTC) - A fast texture compressor for various formats
* [ppsspp](https://github.com/hrydgard/ppsspp) - A PSP emulator for Android, Windows, Mac and Linux, written in C++. Want to contribute? Join us on Discord at https://discord.gg/sdCwGGQ or in #ppsspp on freenode (IRC) or just send pull requests / issues. For discussion use the forums on ppsspp.org. https://www.ppsspp.org
* [Savvy](https://github.com/lotsopa/Savvy) - A Generic and Flexible Shader Cross Compiler Library/Tool
* [cashgenUE](https://github.com/midgen/cashgenUE) - Runtime Procedural Terrain Generator for UnrealEngine
* [ProceduralMeshes](https://github.com/SiggiG/ProceduralMeshes) - Procedural mesh examples for Unreal 4
* [fw-public](https://github.com/johnhable/fw-public) - Public source code from Filmic Worlds, LLC. http://www.filmicworlds.com
* [lullaby](https://github.com/google/lullaby) - A collection of C++ libraries designed to help teams develop virtual and augmented reality experiences( Google )
* [guetzli](https://github.com/google/guetzli) - Perceptual JPEG encoder( Google )
* [FastNoiseSIMD](https://github.com/Auburns/FastNoiseSIMD) - C++ SIMD Noise Library
* [libsimdpp](https://github.com/p12tic/libsimdpp) - Portable header-only zero-overhead C++ low level SIMD library
* [Simd](https://github.com/ermig1979/Simd) - C++ image processing library with using of SIMD: SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AVX, AVX2, AVX-512, VMX(Altivec) and VSX(Power7), NEON for ARM. http://ermig1979.github.io/Simd



## C++
* [rttr](https://github.com/rttrorg/rttr) - C++ Reflection Library http://www.rttr.org
* [fmt](https://github.com/fmtlib/fmt) - A modern formatting library http://fmtlib.net
* [Catch2](https://github.com/catchorg/Catch2) - A modern, C++-native, header-only, test framework for unit-tests, TDD and BDD - using C++11, C++14, C++17 and later (or C++03 on the Catch1.x branch) https://discord.gg/4CWS9zD
* [sanitizers](https://github.com/google/sanitizers) - AddressSanitizer, ThreadSanitizer, MemorySanitizer From Google
* [brofiler](https://github.com/bombomby/brofiler) - C++ Profiler For Games http://brofiler.com
* [orbitprofiler](https://github.com/pierricgimmig/orbitprofiler) - C/C++ Performance Profiler https://orbitprofiler.com/
* [metareflect](https://github.com/Leandros/metareflect) - Metareflect is a lightweight reflection system for C++, based on LLVM and Clangs libtooling. https://arvid.io
* [Cinder](https://github.com/cinder/Cinder) - Cinder is a community-developed, free and open source library for professional-quality creative coding in C++. http://libcinder.org
* [enet](https://github.com/lsalzman/enet) - ENet reliable UDP networking library
* [magnum](https://github.com/mosra/magnum) - Lightweight and modular C++11/C++14 graphics middleware for games and data visualization https://magnum.graphics/
* [JUCE](https://github.com/WeAreROLI/JUCE) - The JUCE cross-platform C++ framework https://juce.com

## Scripting
* [sol2](https://github.com/ThePhD/sol2) Sol v2.0 - a C++ <-> Lua API wrapper with advanced features and top notch performance - is here, and it's great! Documentation: http://sol2.rtfd.io/
* [luvit](https://github.com/luvit/luvit) - Lua + libUV + jIT = pure awesomesauce https://luvit.io/


## [Awesome Github Contributors](#awesome-github-contributors)
* [GameTechDev](https://github.com/GameTechDev) - Intel Game Dev https://software.intel.com/gamedev
* [Arm-Software](https://github.com/ARM-software)
* [GPUOpen](https://github.com/GPUOpen-LibrariesAndSDKs) - AMD Game Dev http://gpuopen-librariesandsdks.github.io/
* [NVIDIAGameWorks](https://github.com/NVIDIAGameWorks) - NVIDIA Technologies for game and application developers https://developer.nvidia.com/gameworks-source-github
* [nvpro-pipeline](https://github.com/nvpro-pipeline)

## Other Awesome List
* [awesome-graphics](https://github.com/ericjang/awesome-graphics) - Curated list of computer graphics tutorials and resources
* [graphics-resources](https://github.com/mattdesl/graphics-resources) - a list of graphic programming resources
* [AwesomeCppGameDev](https://github.com/Cmdu76/AwesomeCppGameDev) - (Needed to sort my github stars, so this might not be as awesome as expected)
* [junkyard_gfx](https://github.com/raizam/junkyard_gfx) - A collection of open source c/c++ libraries for gamedev
* [awesome-c](https://github.com/aleksandar-todorovic/awesome-c) - Continuing the development of awesome-c list on GitHub
* [awesome-compilers](https://github.com/aalhour/awesome-compilers) - curated list of awesome resources, learning materials, tools, frameworks, platforms, technologies and source code projects in the field of Compilers, Interpreters and Runtimes. This list has a bias towards education.
* [open-source-ios-apps](https://github.com/dkhamsing/open-source-ios-apps) - Collaborative List of Open-Source iOS Apps
* [Awesome-ARKit](https://github.com/olucurious/Awesome-ARKit) - 
A curated list of awesome ARKit projects and resources. Feel free to contribute!
* [awesome-vulkan](https://github.com/vinjn/awesome-vulkan) - Awesome Vulkan ecosystem
