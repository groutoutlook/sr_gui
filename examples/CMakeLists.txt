cmake_minimum_required(VERSION 4.0)


# Add backend sources to your library

find_package(glfw3 CONFIG REQUIRED)
add_executable(examples main.c common.c common.h)
target_link_libraries(examples PUBLIC 
sr_gui
glfw 
)
