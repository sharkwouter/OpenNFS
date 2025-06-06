cmake_minimum_required(VERSION 3.15)
project(OpenNFS)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

include(CompileGLSLToSPIRV)

#[[Parameter Override]]
if (APPLE OR WIN32)
    set(USE_VULKAN OFF) #[[Avoid Vulkan on Mac until I add MoltenVK support. Windows too until Vulkan SDK on VSTS container]]
endif ()

#[[Static linking broken under Mac OSX (and UNIX too? + MSVC doesnt support this flag)]]
if (NOT (UNIX OR MSVC))
    set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} "-static")
endif ()

if (${DEV})
    if (APPLE)
        # Silence Fshtool warnings til I deal with them
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-pointer-sign -Wno-return-type")
    endif ()
    MESSAGE(STATUS "CXX Compiler flags are: " ${CMAKE_CXX_FLAGS})
    MESSAGE(STATUS "C Compiler flags are: " ${CMAKE_C_FLAGS})
endif ()

set(OpenGL_GL_PREFERENCE GLVND)
add_definitions(-DIMGUI_IMPL_OPENGL_LOADER_GLEW)

set(IMGUI_SOURCES
        include/imgui/imgui.cpp
        include/imgui/imgui_demo.cpp
        include/imgui/imgui_draw.cpp
        include/imgui/imgui_tables.cpp
        include/imgui/imgui_widgets.cpp
        include/imgui/backends/imgui_impl_glfw.cpp
        include/imgui/backends/imgui_impl_opengl3.cpp
        #[[include/ImGuiColorTextEdit/TextEditor.cpp]])
add_definitions(-DIMGUI_USER_CONFIG="${PROJECT_SOURCE_DIR}/src/Renderer/imgui_config.h")

set(SOURCE_FILES
        ${IMGUI_SOURCES}
        include/bmpread/bmpread.c
        lib/glew-cmake/src/glew.c
        resources/asset/icon.rc
        src/Camera/BaseCamera.cpp
        src/Camera/BaseCamera.cpp
        src/Camera/CarCamera.cpp
        src/Camera/FreeCamera.cpp
        src/Camera/HermiteCamera.cpp
        src/Config.cpp
        src/Input/InputManager.cpp
        src/Loaders/CarLoader.cpp
        src/Loaders/MusicLoader.cpp
        src/Loaders/TrackLoader.cpp
        src/Physics/AABB.cpp
        src/Physics/AABBTree.cpp
        src/Physics/Car.cpp
        src/Physics/Frustum.cpp
        src/Physics/PhysicsManager.cpp
        src/Race/OrbitalManager.cpp
        src/Race/RaceSession.cpp
        src/Race/RacerManager.cpp
        src/Race/Agents/CarAgent.cpp
        src/Race/Agents/PlayerAgent.cpp
        src/Race/Agents/RacerAgent.cpp
        src/Renderer/BulletDebugDrawer.cpp
        src/Renderer/CarRenderer.cpp
        src/Renderer/DebugRenderer.cpp
        src/Renderer/HermiteCurve.cpp
        src/Renderer/UIRenderer.cpp
        src/Renderer/Renderer.cpp
        src/Renderer/ShadowMapRenderer.cpp
        src/Renderer/SkyRenderer.cpp
        src/Renderer/GLTexture.cpp
        src/Renderer/TrackRenderer.cpp
        src/Scene/Entity.cpp
        src/Renderer/Models/GLCarModel.cpp
        src/Renderer/Models/GLModel.cpp
        src/Renderer/Models/GLQuad.cpp
        src/Renderer/Models/GLTrackModel.cpp
        src/Renderer/Models/GLBillboardModel.cpp
        src/Scene/Track.cpp
        src/Scene/GlobalLight.cpp
        src/Scene/Spotlight.cpp
        src/Shaders/BaseShader.cpp
        src/Shaders/BillboardShader.cpp
        src/Shaders/BulletShader.cpp
        src/Shaders/CarShader.cpp
        src/Shaders/DepthShader.cpp
        src/Shaders/FontShader.cpp
        src/Shaders/UIShader.cpp
        src/Shaders/ShaderSet.cpp
        src/Shaders/SkydomeShader.cpp
        src/Shaders/TrackShader.cpp
        src/Util/ImageLoader.cpp
        src/Util/ImageLoader.cpp
        src/Util/Logger.cpp
        src/Util/Utils.cpp
        src/main.cpp
        src/Renderer/MiniMapRenderer.cpp
        src/Shaders/MiniMapShader.cpp
        src/UI/UIElement.cpp
        src/UI/UIButton.cpp
        src/UI/UIImage.cpp
        src/UI/UITextField.cpp
        src/UI/UIManager.cpp
        src/UI/UIResource.cpp)

add_executable(OpenNFS ${SOURCE_FILES})

#[[IMGUI]]
include_directories(include/imgui)

#[[JSON/STB/IMGUI/TINYOBJLOADER/BMPREAD/ARGPARSE/MAGIC_ENUM]]
include_directories(include)

#[[FREETYPE]]
set(FT_DISABLE_HARFBUZZ ON CACHE BOOL "Disable HarfBuzz (used for improving auto-hinting of OpenType fonts).")
set(FT_DISABLE_BROTLI ON CACHE BOOL "Disable support of compressed WOFF2 fonts.")
add_subdirectory(lib/freetype2)
include_directories(lib/freetype2/include)
list(APPEND ONFS_LIBRARIES freetype)

#[[LibOpenNFS]]
add_subdirectory(lib/LibOpenNFS)
include_directories(lib/LibOpenNFS/include)
list(APPEND ONFS_LIBRARIES LibOpenNFS)

#[[G3Log]]
set(G3_SHARED_LIB OFF CACHE BOOL "Compile g3log as static library")
set(ADD_FATAL_EXAMPLE OFF CACHE BOOL "Don't bother compiling invalid code in g3log")
set(ADD_G3LOG_UNIT_TEST OFF CACHE BOOL "No need to test a released library")
add_subdirectory(lib/g3log)
list(APPEND ONFS_LIBRARIES g3log)

#[[Bullet Configuration]]
set(USE_MSVC_RUNTIME_LIBRARY_DLL ON CACHE BOOL "" FORCE)
set(BUILD_BULLET2_DEMOS OFF CACHE BOOL "" FORCE)
set(BUILD_CLSOCKET OFF CACHE BOOL "" FORCE)
set(BUILD_CPU_DEMOS OFF CACHE BOOL "" FORCE)
set(BUILD_ENET OFF CACHE BOOL "" FORCE)
set(BUILD_EXTRAS OFF CACHE BOOL "" FORCE)
set(BUILD_OPENGL3_DEMOS OFF CACHE BOOL "" FORCE)
set(BUILD_UNIT_TESTS OFF CACHE BOOL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
add_subdirectory(lib/bullet3)
include_directories(lib/bullet3/src)
list(APPEND ONFS_LIBRARIES BulletDynamics BulletCollision LinearMath Bullet3Common)

#[[GLEW Configuration]]
add_subdirectory(lib/glew-cmake)
list(APPEND ONFS_LIBRARIES libglew_static)

#[[GLFW Configuration]]
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_WAYLAND OFF CACHE BOOL "" FORCE)
add_subdirectory(lib/glfw)
list(APPEND ONFS_LIBRARIES glfw)

#[[OpenGL Configuration]]
find_package(OpenGL REQUIRED)
include_directories(${OPENGL_INCLUDE_DIRS})
list(APPEND ONFS_LIBRARIES ${OPENGL_LIBRARIES})

#[[Vulkan Configuration]]
if (${USE_VULKAN})
    find_package(Vulkan REQUIRED)
    message("VULKAN FOUND")
    include_directories(${Vulkan_INCLUDE_DIRS})
    list(APPEND ONFS_LIBRARIES ${Vulkan_LIBRARIES})
    CompileGLSLToSpirV(OpenNFS "${CMAKE_CURRENT_SOURCE_DIR}/shaders/vk" "${CMAKE_CURRENT_SOURCE_DIR}/shaders/vk")
    set(VULKAN_SOURCE_FILES src/Renderer/vkRenderer.cpp src/Renderer/vkRenderer.h)
    target_sources(OpenNFS PUBLIC ${VULKAN_SOURCE_FILES})
    add_definitions(-DVULKAN_BUILD)
endif ()

#[[Google Test Framework Configuration]]
set(gtest_force_shared_crt ON CACHE BOOL "Prevent overriding the parent project's compiler/linker setting")
add_subdirectory(lib/googletest)
enable_testing()
include_directories(lib/googletest/googletest/include)
# Add test cpp files
file(GLOB TEST_SRC_FILES ${CMAKE_CURRENT_SOURCE_DIR}/test/*.cpp)
list(REMOVE_ITEM SOURCE_FILES "src/main.cpp")
# From list of files we'll create tests test_name.cpp -> test_name
foreach (_test_file ${TEST_SRC_FILES})
    get_filename_component(_test_name ${_test_file} NAME_WE)
    add_executable(${_test_name} ${_test_file} ${SOURCE_FILES})
    target_link_libraries(${_test_name} gtest gtest_main ${ONFS_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
    add_test(${_test_name} ${_test_name})
    set_tests_properties(${_test_name} PROPERTIES TIMEOUT 10)
endforeach ()

#[[Link Up Ting]]
target_link_libraries(OpenNFS ${ONFS_LIBRARIES})