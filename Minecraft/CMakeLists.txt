find_package(fmt CONFIG)
find_package(spdlog CONFIG)
find_package(CLI11 CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)
find_package(glfw3 REQUIRED)
find_package(glm REQUIRED)
find_package(stb)

add_library(glad ALIAS glad::glad)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

file(GLOB_RECURSE externals
        ${CMAKE_CURRENT_SOURCE_DIR}/External/*.cpp)

file(GLOB_RECURSE headers
        ${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp)

file(GLOB_RECURSE sources
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)

add_definitions(-D__APPLE__)
add_executable(Minecraft ${externals} ${sources})

target_link_libraries(
        Minecraft
        PUBLIC
            project_options project_warnings
        PRIVATE
            CLI11::CLI11 fmt::fmt spdlog::spdlog glad glfw glm::glm stb::stb)

target_include_directories(Minecraft
        PUBLIC
            ${CMAKE_CURRENT_SOURCE_DIR}/include
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR})
