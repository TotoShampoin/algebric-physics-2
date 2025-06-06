cmake_minimum_required(VERSION 3.21)
project(physim-pga)

set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(VEN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/vendor)
set(OUT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/out)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUT_DIR})

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(BUILD_SHARED_LIBS OFF)

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS ${SRC_DIR}/*.cpp)

set(GLFWPP_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

# find_package(glfw3 REQUIRED)
find_package(OpenGL REQUIRED)
add_subdirectory(vendor/glad)
add_subdirectory(vendor/glm)
add_subdirectory(vendor/klein)
add_subdirectory(vendor/glfwpp)
include(vendor/imgui.cmake)
include(vendor/files_and_folders.cmake)

add_executable(${PROJECT_NAME} ${SOURCES})

target_include_directories(${PROJECT_NAME} PUBLIC ${SRC_DIR})

target_link_libraries(${PROJECT_NAME} PRIVATE glad glm imgui klein GLFWPP)

if(WIN32)
    target_link_libraries(${PROJECT_NAME} PUBLIC opengl32)
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL)
endif()

if(MINGW)
    # Get MinGW installation directory
    get_filename_component(MINGW_DIR ${CMAKE_CXX_COMPILER} DIRECTORY)
    
    # List of required DLLs
    set(MINGW_DLLS
        "libstdc++-6.dll"
        "libgcc_s_seh-1.dll"
        "libwinpthread-1.dll"
        "glfw3.dll"
    )
    
    # Copy each DLL to output directory
    foreach(DLL ${MINGW_DLLS})
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${MINGW_DIR}/${DLL}"
            $<TARGET_FILE_DIR:${PROJECT_NAME}>
        )
    endforeach()
endif()

Cool__target_copy_folder(${PROJECT_NAME} res)
