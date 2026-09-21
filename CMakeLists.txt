cmake_minimum_required(VERSION 3.16)
project(CS488)

set(CMAKE_CXX_STANDARD 14)

# On macOS, building GLEW shared may try to link the removed AGL framework.
# For this assignment we only need the static GLEW library.
set(glew-cmake_BUILD_SHARED OFF CACHE BOOL "" FORCE)
set(glew-cmake_BUILD_STATIC ON CACHE BOOL "" FORCE)
set(ONLY_LIBS ON CACHE BOOL "" FORCE)

add_subdirectory(external/glew)
add_subdirectory(external/glfw)

file(GLOB SRC_FILES
        src/*.h
        src/*.cpp)

add_executable(CS488
        ${SRC_FILES})

target_include_directories(CS488 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/external/glfw/include)
target_include_directories(CS488 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/external/glew/include)

if(APPLE)
    add_dependencies(CS488 libglew_static)
    target_compile_definitions(CS488 PRIVATE GLEW_STATIC)
    target_link_libraries(CS488
            glfw
            ${CMAKE_CURRENT_BINARY_DIR}/external/glew/lib/libglew.a
            "-framework OpenGL")
else()
    target_link_libraries(CS488 glfw libglew_static)
endif()