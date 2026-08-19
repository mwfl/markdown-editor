cmake_minimum_required(VERSION 3.21)
project(markdown_editor VERSION 0.1.1 LANGUAGES CXX)

include(FetchContent)
include(GNUInstallDirs)

option(MARKDOWN_EDITOR_USE_LOCAL_MWFL "Use a local mwfl checkout" OFF)
set(MARKDOWN_EDITOR_MWFL_SOURCE_DIR "../mwfl" CACHE PATH "Local mwfl source directory")
set(MWFL_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(MWFL_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(MWFL_BUILD_SCINTILLA ON CACHE BOOL "" FORCE)
set(MWFL_BUILD_WEBVIEW2 ON CACHE BOOL "" FORCE)
if(MARKDOWN_EDITOR_USE_LOCAL_MWFL)
    add_subdirectory("${MARKDOWN_EDITOR_MWFL_SOURCE_DIR}" "${CMAKE_BINARY_DIR}/mwfl")
else()
    FetchContent_Declare(mwfl
        GIT_REPOSITORY https://github.com/mwfl/mwfl.git
        GIT_TAG v0.2.1)
    FetchContent_MakeAvailable(mwfl)
endif()
set(MD4C_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(MD4C_BUILD_MD2HTML_EXECUTABLE OFF CACHE BOOL "" FORCE)
FetchContent_Declare(md4c
    GIT_REPOSITORY https://github.com/mity/md4c.git
    GIT_TAG 729e6b8b320caa96328968ab27d7db2235e4fb47)
FetchContent_MakeAvailable(md4c)

add_executable(markdown-editor WIN32
    main.cpp markdown_renderer.cpp markdown_renderer.h
    markdown_syntax.cpp markdown_syntax.h
    markdown_editor.rc resource.h app-icon.ico app.manifest)
target_link_libraries(markdown-editor PRIVATE mwfl::app mwfl::scintilla mwfl::webview2 md4c-html)
target_include_directories(markdown-editor PRIVATE "${md4c_SOURCE_DIR}/src")
target_compile_features(markdown-editor PRIVATE cxx_std_20)
target_compile_definitions(markdown-editor PRIVATE UNICODE _UNICODE WIN32_LEAN_AND_MEAN NOMINMAX MWFL_APP_VERSION=L\"${PROJECT_VERSION}\")
if(MSVC)
    target_compile_options(markdown-editor PRIVATE /W4 /WX /permissive- /EHsc /utf-8)
endif()
set(MARKDOWN_EDITOR_SCINTILLA_DIR
    "${CMAKE_BINARY_DIR}/_deps/mwfl_scintilla_binary-src")
foreach(runtime IN ITEMS Scintilla.dll Lexilla.dll)
    add_custom_command(TARGET markdown-editor POST_BUILD
        COMMAND "${CMAKE_COMMAND}" -E copy_if_different
            "${MARKDOWN_EDITOR_SCINTILLA_DIR}/${runtime}"
            "$<TARGET_FILE_DIR:markdown-editor>/${runtime}")
endforeach()

enable_testing()
add_executable(markdown-renderer-tests markdown_renderer_test.cpp markdown_renderer.cpp)
target_include_directories(markdown-renderer-tests PRIVATE "${md4c_SOURCE_DIR}/src")
target_link_libraries(markdown-renderer-tests PRIVATE md4c-html)
target_compile_features(markdown-renderer-tests PRIVATE cxx_std_20)
target_compile_definitions(markdown-renderer-tests PRIVATE UNICODE _UNICODE WIN32_LEAN_AND_MEAN NOMINMAX)
if(MSVC)
    target_compile_options(markdown-renderer-tests PRIVATE /W4 /WX /permissive- /EHsc /utf-8)
endif()
add_test(NAME markdown-editor.renderer COMMAND markdown-renderer-tests)

install(TARGETS markdown-editor RUNTIME DESTINATION . COMPONENT Application)
install(FILES
    "${MARKDOWN_EDITOR_SCINTILLA_DIR}/Scintilla.dll"
    "${MARKDOWN_EDITOR_SCINTILLA_DIR}/Lexilla.dll"
    DESTINATION . COMPONENT Application)
install(FILES README.md LICENSE DESTINATION . COMPONENT Application)
install(FILES "${md4c_SOURCE_DIR}/LICENSE.md"
    DESTINATION licenses/md4c COMPONENT Application)
install(FILES "${MARKDOWN_EDITOR_SCINTILLA_DIR}/License.txt"
    DESTINATION licenses/scintilla-lexilla COMPONENT Application)
