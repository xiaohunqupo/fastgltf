set(FASTGLTF_DEPS_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

include(FetchContent)

# glm
if(FASTGLTF_ENABLE_TESTS OR FASTGLTF_ENABLE_EXAMPLES)
    message(STATUS "fastgltf: Fetching glm")
    FetchContent_Declare(
        glm
        GIT_REPOSITORY https://github.com/g-truc/glm
        GIT_TAG 1.0.1
        GIT_SHALLOW TRUE
    )

    # glm breaks because it apparently can't properly detect constexpr/C++ version and
    # sets -Weverything which enables C++98 compatibility on Clang??
    option(GLM_ENABLE_CXX_17 "" ON)

    FetchContent_MakeAvailable(glm)

    add_library(glm::glm ALIAS glm)
    target_compile_definitions(glm PUBLIC GLM_ENABLE_EXPERIMENTAL=1)

    if (TARGET glm_static)
        set_target_properties(glm_static PROPERTIES EXCLUDE_FROM_ALL 1)
        add_library(glm::static ALIAS glm_static)
    endif()
    if (TARGET glm_shared)
        set_target_properties(glm_shared PROPERTIES EXCLUDE_FROM_ALL 1)
        add_library(glm::shared ALIAS glm_shared)
    endif()
endif()

# Catch2
if (FASTGLTF_ENABLE_TESTS)
    message(STATUS "fastgltf: Fetching catch2")
    FetchContent_Declare(
        catch2
        GIT_REPOSITORY https://github.com/catchorg/Catch2
        GIT_TAG v3.7.0
        GIT_SHALLOW TRUE
    )

    # We can set Catch2 config options like this. We want all the C++17 features.
    option(CATCH_CONFIG_CPP17_UNCAUGHT_EXCEPTIONS "" ON)
    option(CATCH_CONFIG_CPP17_STRING_VIEW "" ON)
    option(CATCH_CONFIG_CPP17_VARIANT "" ON)
    option(CATCH_CONFIG_CPP17_OPTIONAL "" ON)
    option(CATCH_CONFIG_CPP17_BYTE "" ON)
    option(CATCH_CONFIG_ENABLE_ALL_STRINGMAKERS "" ON)

    # Make sure that exceptions are never disabled, as some other library seems to disable them.
    option(CATCH_CONFIG_NO_DISABLE_EXCEPTIONS "" ON)

    if (MSVC)
        option(CATCH_CONFIG_WINDOWS_CRTDBG "" ON)
    endif()

    FetchContent_MakeAvailable(catch2)
endif()

# corrosion
if (FASTGLTF_ENABLE_GLTF_RS)
    message(STATUS "fastgltf: Fetching corrosion")
    FetchContent_Declare(
        corrosion
        GIT_REPOSITORY https://github.com/corrosion-rs/corrosion
        GIT_TAG v0.5
        GIT_SHALLOW TRUE
    )

    FetchContent_MakeAvailable(corrosion)
endif()

# glfw. We emulate parts of glfw's CMakeLists but don't want to include any examples or tests.
if(FASTGLTF_ENABLE_EXAMPLES)
    message(STATUS "fastgltf: Fetching glfw")
    FetchContent_Declare(
        glfw
        GIT_REPOSITORY https://github.com/glfw/glfw
        GIT_TAG 3.4
        GIT_SHALLOW TRUE
    )

    # copied from glfw's CMakeLists. We seriously don't want any examples/tests/docs.
    option(GLFW_BUILD_EXAMPLES "Build the GLFW example programs" OFF)
    option(GLFW_BUILD_TESTS "Build the GLFW test programs" OFF)
    option(GLFW_BUILD_DOCS "Build the GLFW documentation" OFF)
    option(GLFW_INSTALL "Generate installation target" OFF)

    FetchContent_MakeAvailable(glfw)

    set_target_properties(glfw PROPERTIES EXCLUDE_FROM_ALL 1)
    set_target_properties(update_mappings PROPERTIES EXCLUDE_FROM_ALL 1)

    add_library(glfw::glfw ALIAS glfw)
endif()

# glad
if (FASTGLTF_ENABLE_EXAMPLES)
    message(STATUS "fastgltf: Fetching glad")
    FetchContent_Declare(
        glad
        GIT_REPOSITORY https://github.com/Dav1dde/glad
        GIT_TAG glad2
        GIT_SHALLOW TRUE
        SOURCE_SUBDIR cmake
    )

    FetchContent_MakeAvailable(glad)

    # Let the glad script generate the glad headers
    glad_add_library(fg_glad_gl46 REPRODUCIBLE EXCLUDE_FROM_ALL LOADER API gl:core=4.6)
endif()

# imgui
if (FASTGLTF_ENABLE_EXAMPLES)
    message(STATUS "fastgltf: Fetching imgui")
    FetchContent_Declare(
        imgui
        GIT_REPOSITORY https://github.com/ocornut/imgui
        GIT_TAG v1.90.9
        GIT_SHALLOW TRUE
        SOURCE_SUBDIR doesnt-exist # This is so that MakeAvailable doesn't call add_subdirectory
    )

    FetchContent_GetProperties(imgui)
    if(NOT imgui_POPULATED)
        FetchContent_MakeAvailable(imgui)

        add_library(imgui)
        add_library(imgui::imgui ALIAS imgui)
        fastgltf_add_source_directory(TARGET imgui FOLDER "${imgui_SOURCE_DIR}")
        fastgltf_add_source_directory(TARGET imgui FOLDER "${imgui_SOURCE_DIR}/misc/cpp/")

        target_include_directories(imgui SYSTEM PUBLIC "${imgui_SOURCE_DIR}")
        target_include_directories(imgui SYSTEM PUBLIC "${imgui_SOURCE_DIR}/backends")
        target_include_directories(imgui SYSTEM PUBLIC "${imgui_SOURCE_DIR}/misc/cpp")

        target_compile_definitions(imgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)

        # We only need the GLFW implementation
        target_sources(imgui PUBLIC "${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.h" "${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp")
        target_sources(imgui PUBLIC "${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.h" "${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp")

        target_link_libraries(imgui PUBLIC glfw::glfw)
    endif()
endif()

# stb
if(FASTGLTF_ENABLE_EXAMPLES)
    message(STATUS "fastgltf: Fetching stb")
    FetchContent_Declare(
        stb
        GIT_REPOSITORY https://github.com/nothings/stb
        GIT_TAG master
        GIT_SHALLOW TRUE
        SOURCE_SUBDIR doesnt-exist # This is so that MakeAvailable doesn't call add_subdirectory
    )

    FetchContent_GetProperties(stb)
    if(NOT stb_POPULATED)
        FetchContent_MakeAvailable(stb)

        add_library(stb INTERFACE)
        target_include_directories(stb INTERFACE ${stb_SOURCE_DIR})
    endif()
endif()
