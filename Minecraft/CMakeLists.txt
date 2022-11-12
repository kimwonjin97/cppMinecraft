find_package(fmt CONFIG)
find_package(spdlog CONFIG)
find_package(CLI11 CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)
find_package(glfw3 REQUIRED)

add_library(glad ALIAS glad::glad)
set(TEST_CMAKE test_cmake)
message("HERE is ${TEST_CMAKE}")
add_library(${TEST_CMAKE} SHARED "")
target_sources(${TEST_CMAKE} PRIVATE test/test.cpp)


#add_library(${TARGET_ARGOS_COMMON} SHARED "")

add_executable(Minecraft main.cpp)

target_link_libraries(
        Minecraft
        PUBLIC
            project_options project_warnings ${TEST_CMAKE}
        PRIVATE
            CLI11::CLI11 fmt::fmt spdlog::spdlog glad glfw)
