cmake_minimum_required(VERSION 3.26)
project(imgui)

option(IMGUI_SHARED "" ON)
option(IMGUI_STATIC "" OFF)
option(IMGUI_FREETYPE "" OFF)

#option(IMGUI_ALLEGRO "" OFF)
#option(IMGUI_ANDROID "" OFF)
#option(IMGUI_DX9 "" OFF)
#option(IMGUI_DX10 "" OFF)
#option(IMGUI_DX11 "" ON)
#option(IMGUI_DX12 "" OFF)
#option(IMGUI_GLFW "" OFF)
#option(IMGUI_GLUT "" OFF)
#option(IMGUI_METAL "" OFF)
#option(IMGUI_OPENGL2 "" OFF)
#option(IMGUI_OPENGL3 "" OFF)
#option(IMGUI_OSX "" OFF)
#option(IMGUI_SDL2 "" OFF)
option(IMGUI_SDL3 "" OFF)
#option(IMGUI_SDL2_RENDERER "" OFF)
option(IMGUI_SDL3_RENDERER "" OFF)
#option(IMGUI_VULKAN "" ON)
#option(IMGUI_WGPU "" OFF)
#option(IMGUI_WIN32 "" OFF)

set(SOURCE_FILES
        imgui.cpp
        imgui_demo.cpp
        imgui_draw.cpp
        imgui_tables.cpp
        imgui_widgets.cpp
)

#set(BACKEND_FILES
#        backends/imgui_impl_allegro5.cpp
#        backends/imgui_impl_android.cpp
#        backends/imgui_impl_dx9.cpp
#        backends/imgui_impl_dx10.cpp
#        backends/imgui_impl_dx11.cpp
#        backends/imgui_impl_dx12.cpp
#        backends/imgui_impl_glfw.cpp
#        backends/imgui_impl_glut.cpp
#        backends/imgui_impl_metal.mm
#        backends/imgui_impl_opengl2.cpp
#        backends/imgui_impl_opengl3.cpp
#        backends/imgui_impl_osx.mm
#        backends/imgui_impl_sdl2.cpp
#        backends/imgui_impl_sdlrenderer2.cpp
#        backends/imgui_impl_sdl3.cpp
#        backends/imgui_impl_sdlrenderer3.cpp
#        backends/imgui_impl_vulkan.cpp
#        backends/imgui_impl_wgpu.cpp
#        backends/imgui_impl_win32.cpp
#)

if(IMGUI_SDL3_RENDERER)
    set(IMGUI_SDL3 ON)
endif()

if(IMGUI_SDL3)
    find_package(SDL3 QUIET)
    message(STATUS "Looking for SDL3...")
    if(SDL3_FOUND)
        message(STATUS "Found SDL3: ${SDL3_INCLUDE_DIR}")
        list(APPEND BACKEND_FILES
                backends/imgui_impl_sdl3.cpp
        )
        set(SDL3_FOUND TRUE)
    elseif(TARGET SDL3::SDL3)
        message(STATUS "Found SDL3: ${SDL3_INCLUDE_DIR}")
        list(APPEND BACKEND_FILES
                backends/imgui_impl_sdl3.cpp
        )
        set(SDL3_FOUND TRUE)
    else()
        message(WARNING "SDL3 not found, SDL3 backend will not be included")
    endif()
endif()

if(IMGUI_SDL3_RENDERER)
    if(SDL3_FOUND)
        list(APPEND BACKEND_FILES
            backends/imgui_impl_sdlrenderer3.cpp
        )
    endif()
endif()

if (IMGUI_SHARED)
    add_library(imgui-shared SHARED
            ${SOURCE_FILES}
            ${BACKEND_FILES}
            misc/cpp/imgui_stdlib.cpp
    )
    set_target_properties(imgui-shared PROPERTIES
            OUTPUT_NAME "imgui"
            WINDOWS_EXPORT_ALL_SYMBOLS TRUE
    )

    target_include_directories(imgui-shared PUBLIC
            ${CMAKE_CURRENT_SOURCE_DIR}
    )

    if(SDL3_FOUND)
        target_link_libraries(imgui-shared PUBLIC SDL3::SDL3)
    endif()

    if (IMGUI_FREETYPE)
        target_sources(imgui-shared
                misc/freetype/imgui_freetype.cpp
        )
    endif ()

    add_library(imgui::imgui-shared ALIAS imgui-shared)
endif ()

if (IMGUI_STATIC)
    add_library(imgui-static STATIC
            ${SOURCE_FILES}
            ${BACKEND_FILES}
            misc/cpp/imgui_stdlib.cpp
    )
    set_target_properties(imgui-static PROPERTIES OUTPUT_NAME "imgui")

    target_include_directories(imgui-static PUBLIC
            ${CMAKE_CURRENT_SOURCE_DIR}
    )

    if(SDL3_FOUND)
        target_link_libraries(imgui-static PUBLIC SDL3::SDL3)
    endif()

    if (IMGUI_FREETYPE)
        target_sources(imgui-static
                misc/freetype/imgui_freetype.cpp
        )
    endif ()
    target_include_directories(imgui-static PUBLIC)

    add_library(imgui::imgui-static ALIAS imgui-static)
endif ()

if (TARGET imgui::imgui-shared)
    add_library(imgui::imgui ALIAS imgui-shared)
else ()
    add_library(imgui::imgui ALIAS imgui-static)
endif ()

