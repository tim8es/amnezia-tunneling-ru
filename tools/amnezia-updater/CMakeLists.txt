cmake_minimum_required(VERSION 3.21)
project(amnezia-list-updater VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt6 6.5 REQUIRED COMPONENTS Core Network Widgets Test)

add_library(amnezia_updater_core
    src/core.cpp
    src/core.h
)
target_link_libraries(amnezia_updater_core PUBLIC Qt6::Core Qt6::Network)
target_include_directories(amnezia_updater_core PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)

add_executable(amnezia-list-updater WIN32 MACOSX_BUNDLE src/main.cpp)
target_link_libraries(amnezia-list-updater PRIVATE amnezia_updater_core Qt6::Widgets)
set_target_properties(amnezia-list-updater PROPERTIES
    MACOSX_BUNDLE_BUNDLE_NAME "Amnezia List Updater"
    MACOSX_BUNDLE_GUI_IDENTIFIER "org.lib4u.amnezia-list-updater"
)

include(CTest)
if(BUILD_TESTING)
    add_executable(amnezia-updater-tests tests/test_core.cpp)
    target_link_libraries(amnezia-updater-tests PRIVATE amnezia_updater_core Qt6::Test)
    add_test(NAME amnezia-updater-tests COMMAND amnezia-updater-tests)
endif()
