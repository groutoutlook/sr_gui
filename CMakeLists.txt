cmake_minimum_required(VERSION 4.0)

project(sr_gui)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 23)
if (PROJECT_IS_TOP_LEVEL)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin-lib")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin-etc")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")
    set(BUILD_EXAMPLES ON CACHE BOOL "Build the example executable")
endif()

# Common source and header files
set(${PROJECT_NAME}_SOURCES
    sr_gui_internal.h
)

set(${PROJECT_NAME}_HEADERS
    sr_gui.h
)

# Platform-specific source files
if (WIN32)
    list(APPEND ${PROJECT_NAME}_SOURCES sr_gui_win.cpp)
    set_source_files_properties(sr_gui_win.cpp PROPERTIES LANGUAGE CXX)
elseif (APPLE)
    list(APPEND ${PROJECT_NAME}_SOURCES sr_gui_mac.m)
    set_source_files_properties(sr_gui_mac.m PROPERTIES LANGUAGE OBJC)
elseif (UNIX AND NOT APPLE)
    list(APPEND ${PROJECT_NAME}_SOURCES sr_gui_lin.c)
    set_source_files_properties(sr_gui_lin.c PROPERTIES LANGUAGE C)
endif()

# Add main static library
add_library(${PROJECT_NAME} STATIC ${${PROJECT_NAME}_SOURCES} ${${PROJECT_NAME}_HEADERS})
target_include_directories(${PROJECT_NAME} PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}")
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_23)

# Compiler warnings
if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W3)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra)
endif()

# Configuration-specific settings
if (CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_definitions(${PROJECT_NAME} PRIVATE NDEBUG)
    target_compile_options(${PROJECT_NAME} PRIVATE -O2)
elseif (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(${PROJECT_NAME} PRIVATE DEBUG)
    target_compile_options(${PROJECT_NAME} PRIVATE -g)
endif()

# Linux-specific dependencies (libnotify, gtk+-3.0)
if (UNIX AND NOT APPLE)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(LIBNOTIFY REQUIRED libnotify)
    pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
    target_include_directories(${PROJECT_NAME} PRIVATE ${LIBNOTIFY_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS})
    target_compile_options(${PROJECT_NAME} PRIVATE ${LIBNOTIFY_CFLAGS_OTHER} ${GTK3_CFLAGS_OTHER})
    target_link_libraries(${PROJECT_NAME} PRIVATE ${LIBNOTIFY_LIBRARIES} ${GTK3_LIBRARIES})
elseif (WIN32)
	target_link_libraries(${PROJECT_NAME} PRIVATE User32 Comdlg32 Comctl32 Shell32 runtimeobject)
	target_compile_definitions(${PROJECT_NAME} PRIVATE _UNICODE UNICODE)
endif()

# sr_gui_cli static library
add_library(sr_gui_cli STATIC
    sr_gui.h
    sr_gui_internal.h
    sr_gui_cli.c
)
target_include_directories(sr_gui_cli PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}")
set_source_files_properties(sr_gui_cli.c PROPERTIES LANGUAGE C)

# Compiler warnings for sr_gui_cli
if (MSVC)
    target_compile_options(sr_gui_cli PRIVATE /W3)
else()
    target_compile_options(sr_gui_cli PRIVATE -Wall -Wextra)
endif()

# Configuration-specific settings for sr_gui_cli
if (CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_definitions(sr_gui_cli PRIVATE NDEBUG)
    target_compile_options(sr_gui_cli PRIVATE -O2)
elseif (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(sr_gui_cli PRIVATE DEBUG)
    target_compile_options(sr_gui_cli PRIVATE -g)
endif()

# Example subdirectory
if (BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

# Custom clean target
add_custom_target(clean-all
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_CURRENT_SOURCE_DIR}/bin"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_CURRENT_SOURCE_DIR}/bin-lib"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_CURRENT_SOURCE_DIR}/bin-etc"
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_BINARY_DIR}"
    COMMENT "Cleaning build directories"
)