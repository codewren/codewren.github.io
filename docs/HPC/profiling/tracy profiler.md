
# Tracy Profiler

https://github.com/wolfpld/tracy

### A real time, nanosecond resolution, remote telemetry, hybrid frame and sampling profiler for games and other applications.

Tracy supports profiling CPU (Direct support is provided for C, C++, Lua, Python and Fortran integration. At the same time, third-party bindings to many other languages exist on the internet, such as [Rust](https://github.com/nagisa/rust_tracy_client), [Zig](https://github.com/tealsnow/zig-tracy), [C#](https://github.com/clibequilibrium/Tracy-CSharp), [OCaml](https://github.com/imandra-ai/ocaml-tracy), [Odin](https://github.com/oskarnp/odin-tracy), etc.), GPU (All major graphic APIs: OpenGL, Vulkan, Direct3D 11/12, Metal, OpenCL, CUDA.), memory allocations, locks, context switches, automatically attribute screenshots to captured frames, and much more.

# Presentation :  An Introduction to Tracy Profiler in C++ - Marcos Slomp - CppCon 2023
https://www.youtube.com/watch?v=ghXk3Bk5F2U

An Introduction to Tracy Profiler in C++ - Marcos Slomp - CppCon 2023
https://github.com/CppCon/CppCon2023

Traditional profilers are prone to skew profiling results due to overhead and time resolution constraints. Moreover, results are commonly presented in some aggregated fashion (e.g., symbol tables, flame graphs, call graphs, etc.) which are unable to pin-point anomalies in a timeline. While insightful, this methodology obfuscates the "when and where" aspect of performance issues, which can lead to rabbit holes and your time wasted.

The Tracy profiler takes a different stance, putting the timeline in the front row. This helps on the identification of pathological cases, and on the performance analysis of specific portions of the program execution. Tracy also enables real-time performance analysis: you can interact with the profiler on-the-fly, as your program runs. While Tracy encourages manual code instrumentation through its minimally invasive, low-overhead, nanosecond resolution annotations, it still supports the more traditional, automatic call stack sampling approach; what's more, there is support for "selective" call stack sampling instrumentation. Even GPU activity can be instrumented and correlated alongside with the CPU timeline. Besides its performance profiling capabilities, Tracy also provides a variety of useful tracing utilities, such as plots, frame delimiters, memory allocation trackers, messages, and plenty more.

This talk will showcase Tracy and make a case as to why it has been an unparalleled tool to assist with research tech transfers at Adobe, as well as in production code, to locate, understand and optimize hot-spots. Tracy is also free and open source, so there's no excuse not to give it a try!