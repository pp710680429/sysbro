cmake_minimum_required(VERSION 3.7)

if (NOT DEFINED VERSION)
    set(VERSION 1.2.2)
endif ()

project(sysbro)

#set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

if (${CMAKE_SYSTEM_PROCESSOR} MATCHES "sw_64")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mieee")
endif ()

if (NOT (${CMAKE_BUILD_TYPE} MATCHES "Debug"))
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

    # generate qm files
    execute_process(COMMAND bash "translate_generation.sh"
                    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
else ()
    set(CMAKE_CXX_COMPILER "clang++")
endif ()


# Sources files
file(GLOB SRCS "src/*.h" "src/*.cpp"
               "src/widgets/*.h" "src/widgets/*.cpp")

# Find the library
find_package(PkgConfig REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5DBus REQUIRED)
find_package(Qt5Concurrent REQUIRED)
find_package(Qt5Gui REQUIRED)
find_package(DtkWidget REQUIRED)
find_package(DtkCore REQUIRED)
find_package(Procps REQUIRED)

# Tell CMake to create the executable
add_executable(sysbro ${SRCS} src/resources.qrc)

target_link_libraries(sysbro
    ${DtkWidget_LIBRARIES}
    ${DtkCore_LIBRARIES}
    ${Qt5Widgets_LIBRARIES}
    ${Qt5DBus_LIBRARIES}
    ${Qt5Concurrent_LIBRARIES}
    procps
)

set(CMAKE_INSTALL_PREFIX /usr)

add_subdirectory("sysbro-delete-files")
add_subdirectory("sysbro-express")
add_subdirectory("sysbro-file-shredder")
add_subdirectory("sysbro-network-test")
add_subdirectory("sysbro-startup-apps")

# Install qm files
file(GLOB QM_FILES "src/translations/*.qm")
install(FILES ${QM_FILES}
        DESTINATION share/sysbro/translations)

# Install icon file
install(FILES src/resources/sysbro.svg DESTINATION share/icons/hicolor/scalable/apps)

# Install policy file
install(FILES src/com.sysbro.pkexec.delete-files.policy DESTINATION share/polkit-1/actions)

install(TARGETS sysbro DESTINATION bin)
install(FILES sysbro.desktop DESTINATION share/applications)
