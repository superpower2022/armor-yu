cmake_minimum_required(VERSION 3.10)

project(tjsp_attack_2021)

set(CMAKE_CXX_STANDARD 14)  # for chrono
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "-Ofast -march=native -pthread")

set(CMAKE_CXX_EXTENSIONS OFF)

add_compile_options(-std=c++14)

#====================== OpenCV Config ============================
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

message(STATUS "OpenCV library status:")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

#========================= Threads ===============================
find_package(Threads REQUIRED)

#======================= ncnn ==============================
find_package(Vulkan)
set(ncnn_DIR ${tjsp_attack_2021_SOURCE_DIR}/ncnn/build/install/lib/cmake/ncnn)
find_package(ncnn)
find_package(OpenMP REQUIRED)
if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

#=========================== Boost ===============================
find_package(Boost REQUIRED COMPONENTS thread)
if (!Boost_FOUND)
    message("-- Boost NOT FOUND, Install with 'sudo apt install libboost-dev'")
endif ()

#========================== Serial ===============================
#set(serial_DIR /usr/local/share/serial/cmake/)
find_package(serial)
include_directories(${serial_INCLUDE_DIRS})

#======================== Dahua SDK ==============================
include_directories(/opt/DahuaTech/MVviewer/include)

#======================== MindVision SDK ==============================
add_library(mindvision SHARED IMPORTED)
set_target_properties(mindvision PROPERTIES 
    IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/sdk/mindvision/lib/x64/libMVSDK.so
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/sdk/mindvision/include)

#========================= Include ===============================
include_directories(source include)

add_executable(attack_mind
    main.cpp
    source/sort/sort.cpp
    source/imageshow.cpp
    source/base.cpp
    source/target.cpp
    source/attack.cpp
    source/communicator.cpp
    source/TfClassifier.cpp
    source/usbio/usbio.cpp
    source/usbio/utility.cpp
    source/windmill/Windmill.cpp
)
target_link_libraries(attack_mind ncnn ${OpenCV_LIBS} ${serial_LIBRARIES} ${Boost_LIBRARIES} usb-1.0 dlib)
target_link_libraries(attack_mind mindvision)
target_compile_definitions(attack_mind PRIVATE MINDVISION)

#========================= systemd ===============================
configure_file(daemon/armor-autostart.service.in daemon/armor-autostart.service)

add_custom_target(install-autostart
    COMMAND sh ${CMAKE_CURRENT_SOURCE_DIR}/daemon/install-armor-autostart.sh
)
