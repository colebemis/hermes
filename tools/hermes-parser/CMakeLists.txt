# Copyright (c) Facebook, Inc. and its affiliates.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

if (EMSCRIPTEN)
  set(HERMES_LINK_COMPONENTS LLVHSupport)

  add_hermes_tool(hermes-parser-wasm
    hermes-parser-wasm.cpp
    HermesParserJSBuilder.cpp
    ${ALL_HEADER_FILES}
    )

  target_link_libraries(hermes-parser-wasm
    hermesAST
    hermesParser
  )

  add_custom_target(
    hermes-parser-js-library
    COMMAND ${CMAKE_C_COMPILER} -E -P -x c-header
    -I ${CMAKE_SOURCE_DIR}/include
    -o ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/HermesParserJSLibrary.js
    ${CMAKE_CURRENT_LIST_DIR}/HermesParserJSLibrary.js.template
  )

  add_dependencies(hermes-parser-wasm hermes-parser-js-library)

  target_link_options(hermes-parser-wasm PRIVATE "SHELL: -s NODERAWFS=0 -s WASM=1")
  target_link_options(hermes-parser-wasm PRIVATE "SHELL: -s ALLOW_MEMORY_GROWTH=1")
  target_link_options(hermes-parser-wasm PRIVATE
    "SHELL: -s EXTRA_EXPORTED_RUNTIME_METHODS=[cwrap,ccall]")
  target_link_options(hermes-parser-wasm PRIVATE "SHELL: -s EXPORT_NAME=hermes_parser_wasm")
  target_link_options(hermes-parser-wasm PRIVATE
    "SHELL: -s BINARYEN_ASYNC_COMPILATION=0 -s SINGLE_FILE=1 -s WASM_ASYNC_COMPILATION=0")
  target_link_options(hermes-parser-wasm PRIVATE
    --js-library ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/HermesParserJSLibrary.js)
  target_link_options(hermes-parser-wasm PRIVATE
    --pre-js ${CMAKE_CURRENT_LIST_DIR}/HermesParserJSUtils.js)
endif ()
