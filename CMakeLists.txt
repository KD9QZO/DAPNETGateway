#
# CMakeLists.txt for DAPNETGateway
#



###############################################################################
# PROJECT SETUP                                                               #
###############################################################################

project(DAPNETGateway)

cmake_minimum_required(VERSION 3.18)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release")
	message(STATUS "Build type not specified; defaulting to Release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")


include(GNUInstallDirs)

set(prefix ${CMAKE_INSTALL_PREFIX})
set(DAPNETGATEWAY_SHARE_DIR ${CMAKE_INSTALL_PREFIX}/share/DAPNETGateway)
set(DAPNETGATEWAY_ETC_DIR ${CMAKE_INSTALL_PREFIX}/etc)
set(SYSTEMD_DIR /etc/systemd/system)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

set(SOURCES
		Conf.cpp
		DAPNETGateway.cpp
		DAPNETNetwork.cpp
		Log.cpp
		POCSAGMessage.cpp
		POCSAGNetwork.cpp
		StopWatch.cpp
		TCPSocket.cpp
		Thread.cpp
		Timer.cpp
		UDPSocket.cpp
		Utils.cpp
		REGEX.cpp
)


######################
# GitVersion.h
######################

include(GitVersion)

configure_file(
		${CMAKE_CURRENT_SOURCE_DIR}/GitVersion.h.in
		${CMAKE_CURRENT_BINARY_DIR}/GitVersion.h
		@ONLY
)



###############################################################################
# COMPILER SPECIFIC SETUP                                                     #
###############################################################################

if(CMAKE_COMPILER_IS_GNUCC AND NOT WIN32)
	add_definitions(-Wall)
endif()

add_definitions(-DHAVE_LOG_H)

find_package(PkgConfig REQUIRED)

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)

find_package(Threads REQUIRED)
if(NOT THREADS_FOUND)
	message(FATAL_ERROR "pthreads is required to build DAPNETGateway")
endif()


include_directories(
		${CMAKE_SOURCE_DIR}
		${CMAKE_CURRENT_BINARY_DIR}
		${THREADS_PTHREADS_INCLUDE_DIR}
)

add_executable(DAPNETGateway ${SOURCES})

set_property(TARGET DAPNETGateway PROPERTY C_STANDARD 11)
set_property(TARGET DAPNETGateway PROPERTY CXX_STANDARD 14)

if(UNIX)
	target_link_libraries(DAPNETGateway m)
endif()
target_link_libraries(DAPNETGateway Threads::Threads)


##########################
# INSTALLATION
##########################

install(
		FILES ${CMAKE_SOURCE_DIR}/DAPNETGateway.ini
		DESTINATION "${DAPNETGATEWAY_ETC_DIR}"
)

install(
		FILES ${CMAKE_SOURCE_DIR}/systemd/dapnetgateway.service
		DESTINATION "${SYSTEMD_DIR}"
)
