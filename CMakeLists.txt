cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
project("NanoGUI" LANGUAGES C CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/Modules")

# ccache
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    message(STATUS "Found CCache")
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

include(CMakeDependentOption)
# superbuild settings
include(SuperBuildDepends)

# cpp standard settings
superbuild_option(cpp17)
superbuild_option(modern-cpp)

# dependencies settings
superbuild_depend(eigen)
superbuild_depend(glfw)

# always generate PIC code, no matter static or shared library
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# project directories settings
set(NANOGUI_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(NANOGUI_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(NANOGUI_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(NANOGUI_BINARY_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/include)
# set(NANOGUI_TEST_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/test)

# some build options
option(NANOGUI_BUILD_EXAMPLE "Build NanoGUI example application" ON)
option(NANOGUI_BUILD_SHARED  "Build NanoGUI as a shared library" ON)

set(NANOGUI_EXTRA_DEFS "")
set(NANOGUI_EXTRA_LIBS "")

set(LIBNANOGUI_EXTRA_SOURCE "")

if(NANOGUI_BUILD_SHARED)
    list(APPEND NANOGUI_EXTRA_DEFS -DNANOGUI_SHARED -DNVG_SHARED)
endif()

list(APPEND NANOGUI_EXTRA_DEFS -DNANOGUI_BUILD -DNVG_BUILD -DNVG_STB_IMAGE_IMPLEMENTATION)

# Required core libraries on various platforms
if (APPLE)
    find_library(cocoa_library Cocoa)
    find_library(opengl_library OpenGL)
    find_library(corevideo_library CoreVideo)
    find_library(iokit_library IOKit)
    list(APPEND NANOGUI_EXTRA_LIBS ${cocoa_library} ${opengl_library} ${corevideo_library} ${iokit_library})
    list(APPEND LIBNANOGUI_EXTRA_SOURCE ${NANOGUI_SOURCE_DIR}/darwin.mm)
elseif(CMAKE_SYSTEM MATCHES "Linux" OR CMAKE_SYSTEM_NAME MATCHES "BSD")
    list(APPEND NANOGUI_EXTRA_LIBS GL Xxf86vm Xrandr Xinerama Xcursor Xi X11 pthread )
    if (NOT CMAKE_SYSTEM_NAME MATCHES "OpenBSD")
        list(APPEND NANOGUI_EXTRA_LIBS rt)
    endif()
    if(CMAKE_SYSTEM MATCHES "Linux")
        list(APPEND NANOGUI_EXTRA_LIBS dl)
    endif()
endif()

# Run simple cmake converter to put font files into the data segment
# Glob up resource files
file(GLOB resources "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.ttf")

# Concatenate resource files into a comma separated string
string(REGEX REPLACE "([^\\]|^);" "\\1," resources_string "${resources}")
string(REGEX REPLACE "[\\](.)" "\\1"     resources_string "${resources_string}")

# Create command line for running bin2c cmake script
set(bin2c_cmdline
    -DOUTPUT_C=nanogui_resources.cpp
    -DOUTPUT_H=nanogui_resources.h
    "-DINPUT_FILES=${resources_string}"
    -P "${CMAKE_CURRENT_SOURCE_DIR}/resources/bin2c.cmake"
)

# Run bin2c on resource files
add_custom_command(
    OUTPUT nanogui_resources.cpp nanogui_resources.h
    COMMAND ${CMAKE_COMMAND} ARGS ${bin2c_cmdline}
    DEPENDS ${resources}
    COMMENT "Running bin2c"
    PRE_BUILD VERBATIM
)

set(NANOGUI_PUBLIC_HEADERS
    ${NANOGUI_INCLUDE_DIR}/nanogui/serializer/core.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/serializer/opengl.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/serializer/sparse.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/button.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/checkbox.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/colorpicker.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/colorwheel.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/combobox.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/common.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/entypo.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/formhelper.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/glcanvas.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/glutil.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/graph.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/imagepanel.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/imageview.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/label.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/layout.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/messagedialog.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/nanogui.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/object.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/opengl.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/popup.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/popupbutton.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/progressbar.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/screen.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/slider.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/stackedwidget.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/tabheader.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/tabwidget.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/textbox.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/theme.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/toolbutton.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/vscrollpanel.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/widget.h
    ${NANOGUI_INCLUDE_DIR}/nanogui/window.h
    ${CMAKE_CURRENT_BINARY_DIR}/nanogui_resources.h
)

set(NANOGUI_PUBLIC_SOURCES
)

set(NANOGUI_PRIVATE_HEADERS
)

set(NANOGUI_PRIVATE_SOURCES
    ${NANOGUI_SOURCE_DIR}/button.cpp
    ${NANOGUI_SOURCE_DIR}/checkbox.cpp
    ${NANOGUI_SOURCE_DIR}/colorpicker.cpp
    ${NANOGUI_SOURCE_DIR}/colorwheel.cpp
    ${NANOGUI_SOURCE_DIR}/combobox.cpp
    ${NANOGUI_SOURCE_DIR}/common.cpp
    ${NANOGUI_SOURCE_DIR}/glcanvas.cpp
    ${NANOGUI_SOURCE_DIR}/glutil.cpp
    ${NANOGUI_SOURCE_DIR}/graph.cpp
    ${NANOGUI_SOURCE_DIR}/imagepanel.cpp
    ${NANOGUI_SOURCE_DIR}/imageview.cpp
    ${NANOGUI_SOURCE_DIR}/label.cpp
    ${NANOGUI_SOURCE_DIR}/layout.cpp
    ${NANOGUI_SOURCE_DIR}/messagedialog.cpp
    ${NANOGUI_SOURCE_DIR}/popup.cpp
    ${NANOGUI_SOURCE_DIR}/popupbutton.cpp
    ${NANOGUI_SOURCE_DIR}/progressbar.cpp
    ${NANOGUI_SOURCE_DIR}/screen.cpp
    ${NANOGUI_SOURCE_DIR}/serializer.cpp
    ${NANOGUI_SOURCE_DIR}/slider.cpp
    ${NANOGUI_SOURCE_DIR}/stackedwidget.cpp
    ${NANOGUI_SOURCE_DIR}/tabheader.cpp
    ${NANOGUI_SOURCE_DIR}/tabwidget.cpp
    ${NANOGUI_SOURCE_DIR}/textbox.cpp
    ${NANOGUI_SOURCE_DIR}/theme.cpp
    ${NANOGUI_SOURCE_DIR}/vscrollpanel.cpp
    ${NANOGUI_SOURCE_DIR}/widget.cpp
    ${NANOGUI_SOURCE_DIR}/window.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/nanogui_resources.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/ext/nanovg/nanovg.c
    ${LIBNANOGUI_EXTRA_SOURCE}
)

add_library(nanogui)
target_compile_definitions(nanogui
    PRIVATE ${NANOGUI_EXTRA_DEFS})
set_target_properties(nanogui PROPERTIES CXX_EXTENSIONS OFF) # -std=c++17 instead of -std=g++17
target_sources(nanogui
    PRIVATE
        ${NANOGUI_PUBLIC_HEADERS}
        ${NANOGUI_PUBLIC_SOURCES}
        ${NANOGUI_PRIVATE_HEADERS}
        ${NANOGUI_PRIVATE_SOURCES}
)
target_include_directories(nanogui
    PUBLIC
        ${NANOGUI_INCLUDE_DIR}
        ${NANOGUI_BINARY_INCLUDE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/ext/nanovg
    PRIVATE
        ${NANOGUI_SOURCE_DIR}
)
target_link_libraries(nanogui
    PUBLIC
        options::modern-cpp
        depends::eigen
        depends::glfw
        ${NANOGUI_EXTRA_LIBS}
)

set(NANOGUI_LIBRARY_DIR ${CMAKE_CURRENT_BINARY_DIR} CACHE INTERNAL "")


# Build example application if desired
if(NANOGUI_BUILD_EXAMPLE)
    add_executable(example1      utils/example1.cpp)
    add_executable(example2      utils/example2.cpp)
    add_executable(example3      utils/example3.cpp)
    add_executable(example4      utils/example4.cpp)
    add_executable(example_icons utils/example_icons.cpp)
    target_link_libraries(example1      PRIVATE nanogui)
    target_link_libraries(example2      PRIVATE nanogui)
    target_link_libraries(example3      PRIVATE nanogui)
    target_link_libraries(example4      PRIVATE nanogui)
    target_link_libraries(example_icons PRIVATE nanogui)

    # Copy icons for example application
    file(COPY resources/icons DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
endif()
