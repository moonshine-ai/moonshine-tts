# Moonshine G2P (ONNX Runtime) — C++ library mirroring moonshine_onnx_g2p.py
#
# Full build (ONNX Runtime): default is cpp/third-party/onnxruntime
# (include/ + lib/linux/x86_64/libonnxruntime.so.1 on Linux x86_64).
#   cmake -S cpp -B cpp/build && cmake --build cpp/build
# Override tree: cmake -S cpp -B cpp/build -DONNXRUNTIME_ROOT=/other/ort
#
# Core-only unit tests (no ORT):
#   cmake -S cpp -B cpp/build-core -DMOONSHINE_TTS_BUILD_ONNX=OFF && cmake --build cpp/build-core
#
# Run tests (doctest) via CTest:
#   cmake --build cpp/build && (cd cpp/build && ctest --output-on-failure)
#
# From this directory (submodule): ``cmake -S . -B build && cmake --build build && ctest --output-on-failure``.
# Parity tests expect the parent monorepo layout (``../data``, ``../models``, Python G2P modules beside
# ``moonshine-tts/``); ref scripts live under ``moonshine-tts/tests/*.py``.
#
# Doctest layout follows https://github.com/moonshine-ai/moonshine/blob/main/core/moonshine-c-api-test.cpp
cmake_minimum_required(VERSION 3.18)
project(moonshine_tts_onnx LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(NOT WIN32)
  add_compile_options(-Wall -Wextra -pedantic)
endif()

include(FetchContent)
FetchContent_Declare(
  json
  GIT_REPOSITORY https://github.com/nlohmann/json.git
  GIT_TAG v3.11.3
)
FetchContent_MakeAvailable(json)

FetchContent_Declare(
  doctest
  GIT_REPOSITORY https://github.com/doctest/doctest.git
  GIT_TAG v2.4.11
)
FetchContent_MakeAvailable(doctest)

FetchContent_Declare(
  utf8proc
  GIT_REPOSITORY https://github.com/JuliaStrings/utf8proc.git
  GIT_TAG v2.9.0
)
FetchContent_GetProperties(utf8proc)
if(NOT utf8proc_POPULATED)
  FetchContent_Populate(utf8proc)
  add_library(utf8proc STATIC ${utf8proc_SOURCE_DIR}/utf8proc.c)
  set_source_files_properties(${utf8proc_SOURCE_DIR}/utf8proc.c PROPERTIES LANGUAGE C)
  target_include_directories(utf8proc PUBLIC ${utf8proc_SOURCE_DIR})
  target_compile_definitions(utf8proc PRIVATE UTF8PROC_EXPORTS PUBLIC UTF8PROC_STATIC)
endif()

enable_testing()

add_library(moonshine_tts_core STATIC
  src/builtin-cpp-data-root.cpp
  src/utf8-utils.cpp
  src/lang-specific/korean-numbers.cpp
  src/lang-specific/korean.cpp
  src/lang-specific/hindi-numbers.cpp
  src/lang-specific/hindi.cpp
  src/lang-specific/vietnamese.cpp
  src/lang-specific/spanish.cpp
  src/lang-specific/turkish.cpp
  src/lang-specific/ukrainian.cpp
  src/lang-specific/spanish-unicode.cpp
  src/lang-specific/spanish-unicode-tables.cpp
  src/lang-specific/german.cpp
  src/lang-specific/french.cpp
  src/lang-specific/french-compound-map.cpp
  src/lang-specific/dutch.cpp
  src/lang-specific/italian.cpp
  src/lang-specific/portuguese.cpp
  src/lang-specific/portuguese-rules.cpp
  src/lang-specific/russian.cpp
  src/lang-specific/chinese-numbers.cpp
  src/lang-specific/chinese.cpp
  src/lang-specific/japanese-kana-to-ipa.cpp
  src/lang-specific/english-hand-oov.cpp
  src/lang-specific/english-numbers.cpp
  src/lang-specific/french-oov.cpp
  src/text-normalize.cpp
  src/lang-specific/arabic-ipa.cpp
  src/lang-specific/cmudict-tsv.cpp
  src/lang-specific/heteronym-context.cpp
  src/ipa-postprocess.cpp
  src/json-config.cpp
  src/g2p-word-log.cpp
)
target_include_directories(moonshine_tts_core PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/src/lang-specific
)
target_include_directories(moonshine_tts_core PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/src
)
target_link_libraries(moonshine_tts_core PUBLIC nlohmann_json::nlohmann_json utf8proc)

add_executable(piper_ipa_normalize_cli tools/piper-ipa-normalize-cli.cpp)
target_link_libraries(piper_ipa_normalize_cli PRIVATE moonshine_tts_core)

add_executable(german_rule_g2p tools/german-rule-g2p-cli.cpp)
target_link_libraries(german_rule_g2p PRIVATE moonshine_tts_core)

add_executable(dutch_rule_g2p tools/dutch-rule-g2p-cli.cpp)
target_link_libraries(dutch_rule_g2p PRIVATE moonshine_tts_core)

add_executable(italian_rule_g2p tools/italian-rule-g2p-cli.cpp)
target_link_libraries(italian_rule_g2p PRIVATE moonshine_tts_core)

add_executable(korean_rule_g2p tools/korean-rule-g2p-cli.cpp)
target_link_libraries(korean_rule_g2p PRIVATE moonshine_tts_core)

add_executable(hindi_rule_g2p tools/hindi-rule-g2p-cli.cpp)
target_link_libraries(hindi_rule_g2p PRIVATE moonshine_tts_core)

add_executable(vietnamese_rule_g2p tools/vietnamese-rule-g2p-cli.cpp)
target_link_libraries(vietnamese_rule_g2p PRIVATE moonshine_tts_core)

add_executable(portuguese_rule_g2p tools/portuguese-rule-g2p-cli.cpp)
target_link_libraries(portuguese_rule_g2p PRIVATE moonshine_tts_core)

add_executable(dutch_g2p_batch tools/dutch-g2p-batch-cli.cpp)
target_link_libraries(dutch_g2p_batch PRIVATE moonshine_tts_core)

add_executable(french_g2p_batch tools/french-g2p-batch-cli.cpp)
target_link_libraries(french_g2p_batch PRIVATE moonshine_tts_core)

set(DOCTEST_INC "${doctest_SOURCE_DIR}")

function(moonshine_tts_add_test name)
  string(REPLACE "_" "-" name_dash "${name}")
  add_executable(${name} tests/${name_dash}.cpp)
  target_include_directories(${name} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/tests ${DOCTEST_INC})
  target_link_libraries(${name} PRIVATE ${ARGN})
  add_test(NAME ${name} COMMAND ${name})
endfunction()

moonshine_tts_add_test(utf8_utils_test moonshine_tts_core)
moonshine_tts_add_test(german_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(dutch_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(italian_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(portuguese_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(russian_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(chinese_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(korean_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(vietnamese_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(french_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(spanish_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(turkish_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(ukrainian_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(hindi_rule_g2p_test moonshine_tts_core)
moonshine_tts_add_test(text_normalize_test moonshine_tts_core)
moonshine_tts_add_test(heteronym_context_test moonshine_tts_core)
moonshine_tts_add_test(ipa_postprocess_test moonshine_tts_core)
moonshine_tts_add_test(cmudict_tsv_test moonshine_tts_core)
moonshine_tts_add_test(json_config_test moonshine_tts_core)
moonshine_tts_add_test(english_hand_oov_test moonshine_tts_core)

option(MOONSHINE_TTS_BUILD_ONNX "Build ONNX Runtime targets (uses cpp/third-party/onnxruntime by default)" ON)

if(MOONSHINE_TTS_BUILD_ONNX)
  include(${CMAKE_CURRENT_LIST_DIR}/cmake/moonshine-tts-onnxruntime.cmake)

  add_library(moonshine_tts_ort STATIC
  src/lang-specific/onnx-g2p-models.cpp
  src/lang-specific/korean-tok-pos-onnx.cpp
  src/lang-specific/japanese-tok-pos-onnx.cpp
  src/lang-specific/chinese-tok-pos-onnx.cpp
  src/lang-specific/japanese-onnx-g2p.cpp
  src/lang-specific/chinese-onnx-g2p.cpp
  src/lang-specific/arabic-diac-onnx.cpp
    src/moonshine-g2p.cpp
    src/rule-based-g2p-factory.cpp
    src/lang-specific/english.cpp
    src/lang-specific/japanese.cpp
    src/lang-specific/arabic.cpp
    src/moonshine-tts.cpp
    src/piper-tts.cpp
  )
  target_link_libraries(moonshine_tts_ort PUBLIC moonshine_tts_core)
  moonshine_tts_link_onnxruntime(moonshine_tts_ort)

  add_executable(moonshine_tts tools/moonshine-tts-cli.cpp)
  set_target_properties(moonshine_tts PROPERTIES OUTPUT_NAME "moonshine-tts")
  target_link_libraries(moonshine_tts PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(moonshine_tts)

  add_executable(moonshine_tts_g2p_cli tools/moonshine-g2p-cli.cpp)
  set_target_properties(moonshine_tts_g2p_cli PROPERTIES OUTPUT_NAME "moonshine-tts-g2p")
  target_link_libraries(moonshine_tts_g2p_cli PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(moonshine_tts_g2p_cli)

  add_executable(piper_phoneme_infer tools/piper-phoneme-infer-cli.cpp)
  target_link_libraries(piper_phoneme_infer PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(piper_phoneme_infer)

  add_executable(japanese_onnx_g2p tools/japanese-onnx-g2p-cli.cpp)
  target_link_libraries(japanese_onnx_g2p PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(japanese_onnx_g2p)

  add_executable(chinese_rule_g2p tools/chinese-rule-g2p-cli.cpp)
  target_link_libraries(chinese_rule_g2p PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(chinese_rule_g2p)

  add_executable(arabic_rule_g2p tools/arabic-rule-g2p-cli.cpp)
  target_link_libraries(arabic_rule_g2p PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(arabic_rule_g2p)

  moonshine_tts_add_test(onnx_g2p_smoke_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(onnx_g2p_smoke_test)

  moonshine_tts_add_test(korean_tok_pos_onnx_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(korean_tok_pos_onnx_test)

  moonshine_tts_add_test(japanese_tok_pos_onnx_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(japanese_tok_pos_onnx_test)

  moonshine_tts_add_test(chinese_tok_pos_onnx_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(chinese_tok_pos_onnx_test)

  moonshine_tts_add_test(japanese_onnx_g2p_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(japanese_onnx_g2p_test)

  moonshine_tts_add_test(arabic_rule_g2p_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(arabic_rule_g2p_test)

  moonshine_tts_add_test(english_rule_g2p_test moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(english_rule_g2p_test)

  target_link_libraries(russian_rule_g2p_test PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(russian_rule_g2p_test)
  target_compile_definitions(russian_rule_g2p_test PRIVATE MOONSHINE_TTS_WITH_G2P_CLASS=1)

  target_link_libraries(chinese_rule_g2p_test PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(chinese_rule_g2p_test)
  target_compile_definitions(chinese_rule_g2p_test PRIVATE MOONSHINE_TTS_WITH_G2P_CLASS=1)

  target_link_libraries(korean_rule_g2p_test PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(korean_rule_g2p_test)
  target_compile_definitions(korean_rule_g2p_test PRIVATE MOONSHINE_TTS_WITH_G2P_CLASS=1)

  target_link_libraries(vietnamese_rule_g2p_test PRIVATE moonshine_tts_ort)
  moonshine_tts_set_ort_rpath(vietnamese_rule_g2p_test)
  target_compile_definitions(vietnamese_rule_g2p_test PRIVATE MOONSHINE_TTS_WITH_G2P_CLASS=1)
endif()
