# require 3.15 for GNUInstallDirs
cmake_minimum_required(VERSION 3.15...3.18)

project(QuaZip VERSION 1.2)

set(CMAKE_CXX_STANDARD 14)

set(QUAZIP_LIB_VERSION ${QuaZip_VERSION})
set(QUAZIP_LIB_SOVERSION 1.0.0)

option(BUILD_SHARED_LIBS "" ON)
option(QUAZIP_INSTALL "" ON)
option(QUAZIP_USE_QT_ZLIB "" OFF)
option(QUAZIP_ENABLE_TESTS "Build QuaZip tests" OFF)
set(QUAZIP_QT_MAJOR_VERSION 5 CACHE STRING "Qt version to use (4, 5 or 6), defaults to 5")

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE RELEASE)
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_DEBUG_POSTFIX d)

set(QUAZIP_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(QUAZIP_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(QUAZIP_LIB_FILE_NAME quazip${QuaZip_VERSION_MAJOR}-qt${QUAZIP_QT_MAJOR_VERSION})
set(QUAZIP_LIB_TARGET_NAME QuaZip)
set(QUAZIP_DIR_NAME QuaZip-Qt${QUAZIP_QT_MAJOR_VERSION}-${QUAZIP_LIB_VERSION})
set(QUAZIP_PACKAGE_NAME QuaZip-Qt${QUAZIP_QT_MAJOR_VERSION})

if(QUAZIP_QT_MAJOR_VERSION EQUAL 6)
	find_package(Qt6 REQUIRED COMPONENTS Core Core5Compat
                         OPTIONAL_COMPONENTS Network Test)
	set(QUAZIP_LIB_LIBRARIES Qt6::Core Qt6::Core5Compat)
	set(QUAZIP_TEST_QT_LIBRARIES Qt6::Core Qt6::Core5Compat Qt6::Network Qt6::Test)
	set(QUAZIP_PKGCONFIG_REQUIRES "zlib, Qt6Core")
elseif(QUAZIP_QT_MAJOR_VERSION EQUAL 5)
	find_package(Qt5 REQUIRED COMPONENTS Core
                         OPTIONAL_COMPONENTS Network Test)
	set(QUAZIP_LIB_LIBRARIES Qt5::Core)
	set(QUAZIP_TEST_QT_LIBRARIES Qt5::Core Qt5::Network Qt5::Test)
	set(QUAZIP_PKGCONFIG_REQUIRES "zlib, Qt5Core")
elseif(QUAZIP_QT_MAJOR_VERSION EQUAL 4)
	find_package(Qt4 4.5.0 REQUIRED COMPONENTS QtCore
                           OPTIONAL_COMPONENTS QtNetwork QtTest)
	set(QUAZIP_LIB_LIBRARIES Qt4::QtCore)
	set(QUAZIP_TEST_QT_LIBRARIES Qt4::QtCore Qt4::QtNetwork Qt4::QtTest)
	set(QUAZIP_PKGCONFIG_REQUIRES "zlib, QtCore")
else()
	message(FATAL_ERROR "Qt version ${QUAZIP_QT_MAJOR_VERSION} is not supported")
endif()


set(QUAZIP_QT_ZLIB_USED OFF)
if(QUAZIP_USE_QT_ZLIB)
    find_package(Qt${QUAZIP_QT_MAJOR_VERSION} OPTIONAL_COMPONENTS Zlib)
    if (Qt${QUAZIP_QT_MAJOR_VERSION}Zlib_FOUND)
        set(QUAZIP_LIB_LIBRARIES ${QUAZIP_LIB_LIBRARIES} Qt${QUAZIP_QT_MAJOR_VERSION}::Zlib)
        set(QUAZIP_QT_ZLIB_USED ON)
    endif()
endif()
if(NOT QUAZIP_QT_ZLIB_USED)
    find_package(ZLIB REQUIRED)
    set(QUAZIP_LIB_LIBRARIES ${QUAZIP_LIB_LIBRARIES} ZLIB::ZLIB)
endif()

add_subdirectory(quazip)

if(QUAZIP_ENABLE_TESTS)
    message(STATUS "Building QuaZip tests")
    enable_testing()
    add_subdirectory(qztest)
endif()
