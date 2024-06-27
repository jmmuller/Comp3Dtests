#######################################################
##        General
#######################################################
cmake_minimum_required(VERSION 3.15)
project(Comp3D_tests)
set(COMP3D_TARGET Comp3D_tests)

# strict C++17 required
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS OFF)

#######################################################
##        Options
#######################################################

set(ADD_PROJ_CC ON)
set(ADD_PROJ_NTF ON)
set(ADD_PROJ_UTM ON)
set(USE_QT ON)
set(USE_RES ON)

#######################################################
##        Dependencies
#######################################################

# Eigen
find_package(Eigen3 REQUIRED NO_MODULE)

# Boost
set (BOOST_VERSION_MIN 1.74.0)
set (BOOST_COMPONENTS system filesystem date_time graph)
set (BOOST_COMPONENTS_FULL ${BOOST_COMPONENTS})
list (TRANSFORM BOOST_COMPONENTS_FULL PREPEND Boost::)
find_package(Boost ${BOOST_VERSION_MIN} REQUIRED COMPONENTS ${BOOST_COMPONENTS})

# Proj
set(PROJ_ROOT /usr/local/proj82)
find_package(PROJ 8.2.1 REQUIRED)

# Qt
set (QT_COMPONENTS Widgets Test)
set (QT_COMPONENTS_FULL ${QT_COMPONENTS})
list (TRANSFORM QT_COMPONENTS_FULL PREPEND Qt5::)
find_package(Qt5 REQUIRED COMPONENTS ${QT_COMPONENTS})
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)


#######################################################
##        Compilation toolchain
#######################################################

# ccache
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
endif(CCACHE_FOUND)

# compilers specific options : warnings, MSVC specific defines
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
set(gnu_cxx  "$<COMPILE_LANG_AND_ID:CXX,GNU>")

add_library(COMP3D_compiler_flags INTERFACE)
target_compile_options(COMP3D_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Werror>>"
    "$<${msvc_cxx}:$<BUILD_INTERFACE:/W3;/D_USE_MATH_DEFINES;/D_HAS_STD_BYTE=0;/wd4244;/wd4267;/wd4305;/wd4661;/D_CRT_SECURE_NO_WARNINGS>>"
    "$<$<CONFIG:Debug>:$<$<CXX_COMPILER_ID:MSVC>:/bigobj>>"
    "$<$<CONFIG:RelWithDebInfo>:$<$<CXX_COMPILER_ID:MSVC>:/bigobj>>"
)

#######################################################
##     Versionning
#######################################################
ADD_CUSTOM_COMMAND(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/git_revision.cpp
    COMMAND ${CMAKE_COMMAND} -P
            ${CMAKE_CURRENT_SOURCE_DIR}/../version.cmake)

#######################################################
##        Sources
#######################################################

set(SOURCES
    ../src/fformat.cpp
    ../src/filerefjson.cpp
    ../src/leastsquares.cpp
    ../src/misc_tools.cpp
    ../src/obs.cpp
    ../src/point.cpp
    ../src/coord.cpp
    ../src/projection.cpp
    ../src/station.cpp
    ../src/station_eq.cpp
    ../src/station_simple.cpp
    ../src/parameter.cpp
    ../src/project.cpp
    ../src/mathtools.cpp
    ../src/station_hz.cpp
    ../src/jsoncpp.cpp
    ../src/project_config.cpp
    ../src/datafile.cpp
    ../src/ellipsoid.cpp
    ../src/station_axis.cpp
    ../src/axisobs.cpp
    ../src/station_bascule.cpp
    ../src/matrixordering.cpp
    ../src/info.cpp
    ../src/varcovarmatrix.cpp
    tests_coord.cpp
    tests_main.cpp
    tests_basc.cpp
    compcompare.cpp
    tests_nonreg.cpp
    tests_stab.cpp
)


#######################################################
##        Target
#######################################################

add_executable(${COMP3D_TARGET} ${SOURCES} ../ressource.qrc ${CMAKE_BINARY_DIR}/../git_revision.cpp )

target_include_directories (${COMP3D_TARGET} PRIVATE ../src . ${PROJ_INCLUDE_DIRS})
target_link_libraries (${COMP3D_TARGET} PRIVATE COMP3D_compiler_flags ${BOOST_COMPONENTS_FULL} ${QT_COMPONENTS_FULL} ${PROJ_LIBRARIES} Eigen3::Eigen)

target_compile_definitions(${COMP3D_TARGET} PRIVATE
	COMP3D_DOC_DIR="${COMP3D_DOC_DIR}"
	$<$<BOOL:${ADD_PROJ_CC}>:ADD_PROJ_CC>
	$<$<BOOL:${ADD_PROJ_NTF}>:ADD_PROJ_NTF>
	$<$<BOOL:${ADD_PROJ_UTM}>:ADD_PROJ_UTM>
	$<$<BOOL:${USE_QT}>:USE_QT>
	$<$<BOOL:${USE_RES}>:USE_RES>
)


