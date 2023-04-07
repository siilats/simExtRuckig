set(CMAKE_OSX_ARCHITECTURES "x86_64" CACHE INTERNAL "" FORCE)
cmake_minimum_required(VERSION 3.15)

project(simExtRuckig)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_MACOSX_RPATH 1)

set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)
set(CMAKE_OSX_SYSROOT "/Library/Developer/CommandLineTools/SDKs/MacOSX13.1.sdk")
set(Qt5_DIR "/opt/homebrew/Cellar/qt@5/5.15.8_2/lib/cmake/Qt5")
set(COPPELIASIM_ROOT_DIR "/Applications/coppeliaSim.app/Contents/Resources")

set( BOOST_ROOT /usr/local/homebrew/Cellar/boost/1.81.0_1 CACHE PATH "Boost library path" )
set( Boost_NO_SYSTEM_PATHS on CACHE BOOL "Do not search system for Boost" )
set( Boost_NO_BOOST_CMAKE on CACHE BOOL "Do not search system for Boost" )

set( Boost_INCLUDE_DIRS ${BOOST_ROOT}/include )

include_directories(${Boost_INCLUDE_DIRS})

link_directories(/Applications/coppeliaSim.app/Contents/MacOS)

find_package(Boost REQUIRED)
find_package(Boost REQUIRED COMPONENTS regex)

if(NOT COPPELIASIM_INCLUDE_DIR)
    if(DEFINED COPPELIASIM_ROOT_DIR)
        set(COPPELIASIM_INCLUDE_DIR ${COPPELIASIM_ROOT_DIR}/programming/include)
    else()
        set(COPPELIASIM_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../include)
    endif()
endif()
list(APPEND CMAKE_MODULE_PATH ${COPPELIASIM_INCLUDE_DIR}/cmake)
find_package(CoppeliaSim 4.5.0.0 REQUIRED)

if(WIN32)
    add_compile_definitions(_USE_MATH_DEFINES)
endif()

include(ExternalProject)
ExternalProject_Add(libruckig-build
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ruckig
    BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/ruckig
    INSTALL_COMMAND ""
)

if(WIN32)
    set(RUCKIG_LIBRARY_PATH ${CMAKE_CURRENT_BINARY_DIR}/ruckig/Release)
    set(RUCKIG_SHARED_LIBRARY ${RUCKIG_LIBRARY_PATH}/ruckig.dll)
    set(RUCKIG_LINK_LIBRARY ${RUCKIG_LIBRARY_PATH}/ruckig.lib)
else()
    set(RUCKIG_LIBRARY_PATH ${CMAKE_CURRENT_BINARY_DIR}/ruckig)
    set(RUCKIG_SHARED_LIBRARY ${RUCKIG_LIBRARY_PATH}/${CMAKE_SHARED_LIBRARY_PREFIX}ruckig${CMAKE_SHARED_LIBRARY_SUFFIX})
    set(RUCKIG_LINK_LIBRARY ${RUCKIG_SHARED_LIBRARY})
endif()

coppeliasim_add_plugin(
    simExtRuckig
    LEGACY
    SOURCES
    simExtRuckig.cpp
)
add_dependencies(simExtRuckig libruckig-build)
target_include_directories(simExtRuckig PRIVATE ruckig/include)
target_link_libraries(simExtRuckig ${RUCKIG_LINK_LIBRARY})
coppeliasim_add_library(${RUCKIG_SHARED_LIBRARY})

file(GLOB LIBRARIES "/Applications/coppeliaSim.app/Contents/MacOS/*.dylib")
message("LIBRARIES = ${LIBRARIES}")
target_link_libraries(simExtRuckig ${LIBRARIES})