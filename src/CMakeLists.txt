cmake_minimum_required(VERSION 3.0)
include(CheckLibraryExists)
include (CheckIncludeFiles)
include(CMakePrintHelpers)
include(TestBigEndian)
if (POLICY CMP0020)
    cmake_policy(SET CMP0020 NEW)
endif()
project(DNSViewer)

set (DNSViewer_VERSION_MAJOR 0)
set (DNSViewer_VERSION_MINOR 7)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

find_package(Qt4 COMPONENTS QtCore QtGui)
find_package(Qt5Widgets)
find_package(Qt5Gui)
find_package(Qt5Core)
set(UI_ADDED listwindow.ui)
set(RESOURCE_ADDED ../DNSViewer.qrc)
set(SRCS 
    ${RESOURCE_ADDED} ${UI_ADDED}
    listwindow.cpp main.cpp pcapthread.cpp ifcapimpl.cpp pcapimpl.cpp)
if (MSVC)
  set(SRCS ${SRCS} ../res.rc)
endif()

add_executable(dnsviewer ${SRCS})
if (Qt5Widgets_LIBRARIES AND Qt5Core_LIBRARIES AND Qt5Gui_LIBRARIES)
    set(QT_LIBRARIES ${Qt5Widgets_LIBRARIES} ${Qt5Core_LIBRARIES} ${Qt5Gui_LIBRARIES})
else()
    set(QT_LIBRARIES Qt4::QtCore Qt4::QtGui)
endif()
cmake_print_variables(QT_LIBRARIES)
target_link_libraries(dnsviewer ${QT_LIBRARIES})

#find wpcap/pcap
find_path(PCAP_INCLUDES pcap.h
    ${PCAP_ROOT_DIR}/Include
    ${PCAP_ROOT_DIR}/include )
include_directories(${PCAP_INCLUDES})
find_library(PCAP_LIBRARY
    NAMES wpcap pcap
    PATHS ${PCAP_ROOT_DIR}/Lib ${PCAP_ROOT_DIR}/lib)
check_library_exists(${PCAP_LIBRARY} pcap_open "" _HAS_PCAP_OPEN)   
get_filename_component(PCAP_LIB_NAME ${PCAP_LIBRARY} NAME_WE)
if ( ${PCAP_LIB_NAME} STREQUAL "wpcap" )
    set(HAVE_REMOTE 1)
    set(WPCAP 1)
    set(_HAS_WSOCK2_H 1)
    target_link_libraries(dnsviewer ws2_32)
endif()
target_link_libraries(dnsviewer ${PCAP_LIBRARY})

test_big_endian(_BIG_ENDIAN)

#dnsviewer.h
check_include_files(arpa/inet.h _HAS_ARPA_INET_H)
configure_file(${PROJECT_SOURCE_DIR}/dnsviewer.h.in
    ${PROJECT_BINARY_DIR}/dnsviewer.h)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

install(TARGETS dnsviewer DESTINATION bin)

#installer
include(InstallRequiredSystemLibraries)
configure_file(${PROJECT_SOURCE_DIR}/CPackOptions.cmake.in
    ${PROJECT_BINARY_DIR}/CPackOptions.cmake @ONLY)
set(CPACK_PROJECT_CONFIG_FILE "${PROJECT_BINARY_DIR}/CPackOptions.cmake")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "View DNS lookups")
SET(CPACK_PACKAGE_VENDOR "Justin Borodinsky")
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/../LICENSE")
SET(CPACK_PACKAGE_VERSION_MAJOR "${DNSViewer_VERSION_MAJOR}")
SET(CPACK_PACKAGE_VERSION_MINOR "${DNSViewer_VERSION_MINOR}")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "dnsviewer${DNSViewer_VERSION_MAJOR}.${DNSViewer_VERSION_MINOR}")
set(CPACK_PACKAGE_EXECUTABLES dnsviewer DNSViewer)
include(CPack)
