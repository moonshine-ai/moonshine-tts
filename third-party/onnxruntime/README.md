# ONNX Runtime (vendored headers / SDK layout)

- **Upstream repository:** https://github.com/microsoft/onnxruntime  
- **Contents:** C/C++ API headers under `include/`. Prebuilt shared libraries (e.g. `libonnxruntime.so`) are expected on the build machine or next to this tree per `cmake/moonshine-tts-onnxruntime.cmake`; they are not necessarily committed here.  
- **API:** `include/onnxruntime_c_api.h` defines `ORT_API_VERSION` for the bundled headers.  
- **License:** [LICENSE](LICENSE) (MIT, Microsoft Corporation). Release packages may ship additional third-party notices; see upstream releases for full attribution files if required for distribution.
