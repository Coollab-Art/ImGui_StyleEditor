cmake_minimum_required(VERSION 3.20)

set(WARNINGS_AS_ERRORS_FOR_IMSTYLEED OFF CACHE BOOL "ON iff you want to treat warnings as errors")

add_library(ImStyleEd)
add_library(ImStyleEd::ImStyleEd ALIAS ImStyleEd)
target_compile_features(ImStyleEd PUBLIC cxx_std_20)

# ---Add source files---
if(WARNINGS_AS_ERRORS_FOR_IMSTYLEED)
    target_include_directories(ImStyleEd PUBLIC include)
else()
    target_include_directories(ImStyleEd SYSTEM PUBLIC include)
endif()

file(GLOB_RECURSE SRC_FILES CONFIGURE_DEPENDS src/*.cpp)
target_sources(ImStyleEd PRIVATE ${SRC_FILES})

# ---Set warning level---
if(MSVC)
    target_compile_options(ImStyleEd PRIVATE /W4)
else()
    target_compile_options(ImStyleEd PRIVATE -Wall -Wextra -Wpedantic -pedantic-errors)
endif()

# ---Maybe enable warnings as errors---
if(WARNINGS_AS_ERRORS_FOR_IMSTYLEED)
    if(MSVC)
        target_compile_options(ImStyleEd PRIVATE /WX)
    else()
        target_compile_options(ImStyleEd PRIVATE -Werror)
    endif()
endif()

# ---glm library---
target_include_directories(ImStyleEd SYSTEM PRIVATE "lib/glm")
install(FILES "lib/glm/copying.txt" DESTINATION "license/glm")

# ---wants_dark_theme library---
add_subdirectory(lib/wants_dark_theme)
target_link_libraries(ImStyleEd PRIVATE wants_dark_theme::wants_dark_theme)

# ---nlohmann json library---
target_include_directories(ImStyleEd SYSTEM PRIVATE lib/json/single_include)
install(FILES "lib/json/LICENSE.MIT" DESTINATION "license/nlohmann_json")

# ---Boxer library---
# add_subdirectory(lib/Boxer) # Removed because it conflicts with Cool's version of the library
target_link_libraries(ImStyleEd PRIVATE Boxer)
install(FILES "lib/Boxer/LICENSE" DESTINATION "license/Boxer")