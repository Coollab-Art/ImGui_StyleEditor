cmake_minimum_required(VERSION 3.20)
project(ImStyleEd-tests)

# ---Create executable---
add_executable(${PROJECT_NAME} tests.cpp)
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_20)

# ---Set warning level---
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic -pedantic-errors -Wconversion -Wsign-conversion)
endif()

# ---Maybe enable warnings as errors---
set(WARNINGS_AS_ERRORS_FOR_IMSTYLEED OFF CACHE BOOL "ON iff you want to treat warnings as errors")

if(WARNINGS_AS_ERRORS_FOR_IMSTYLEED)
    if(MSVC)
        target_compile_options(${PROJECT_NAME} PRIVATE /WX)
    else()
        target_compile_options(${PROJECT_NAME} PRIVATE -Werror)
    endif()
endif()

# ---Include our library---
add_subdirectory(.. ${CMAKE_CURRENT_SOURCE_DIR}/build/ImStyleEd)
target_link_libraries(${PROJECT_NAME} PRIVATE ImStyleEd::ImStyleEd)

# ---Add exe_path---
include(FetchContent)
FetchContent_Declare(
    exe_path
    GIT_REPOSITORY https://github.com/Coollab-Art/exe_path
    GIT_TAG bcf81c8e1d48c056081e232fe297c73743e72420
)
FetchContent_MakeAvailable(exe_path)
target_link_libraries(${PROJECT_NAME} PRIVATE exe_path::exe_path)

# ---Add quick_imgui---
include(FetchContent)
FetchContent_Declare(
    quick_imgui
    GIT_REPOSITORY https://github.com/Coollab-Art/quick_imgui
    GIT_TAG 98f5a1acdc5ae28013120fec564422a88fa5b74c
)
FetchContent_MakeAvailable(quick_imgui)
target_include_directories(ImStyleEd PRIVATE ${quick_imgui_SOURCE_DIR}/lib) # Give our library access to Dear ImGui
target_link_libraries(${PROJECT_NAME} PRIVATE quick_imgui::quick_imgui)

# ---Ignore .vscode/settings.json in Git---
find_package(Git QUIET)

if(GIT_FOUND)
    get_filename_component(PARENT_DIR ${CMAKE_CURRENT_SOURCE_DIR} DIRECTORY)

    if(EXISTS "${PARENT_DIR}/.git")
        execute_process(COMMAND ${GIT_EXECUTABLE} update-index --assume-unchanged .vscode/settings.json
            WORKING_DIRECTORY ${PARENT_DIR}
            RESULT_VARIABLE ERRORS)

        if(NOT ERRORS EQUAL "0")
            message("Git assume-unchanged failed: ${ERRORS}")
        endif()
    else()
        message("No Git repository found.")
    endif()
else()
    message("Git executable not found.")
endif()
