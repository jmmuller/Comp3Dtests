#######################################################
##        General
#######################################################
cmake_minimum_required(VERSION 3.15)
project(Comp3D)
set(COMP3D_TARGET Comp3D)

set(TRANSLATIONS fr)

# strict C++17 required
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS OFF)

#######################################################
##        Options
#######################################################

option(ADD_PROJ_CC  "Add proj CC" ON)
option(ADD_PROJ_NTF "Add proj NTF" ON)
option(ADD_PROJ_UTM "Add Proj UTM" OFF)

option(USE_AUTO "Automatic mode" ON)
option(USE_SIM "Simulation" ON)

option(USE_GUI "Simultation" ON)
if (NOT ${USE_GUI})
  option(USE_RES "ressources" ON)
else()
  set(USE_RES ON CACHE BOOL "ressources" FORCE)
endif()

if (NOT ${USE_RES})
  option(USE_QT "QT" ON)
else()
  set(USE_QT ON CACHE BOOL "QT" FORCE)
endif()

set(COMP3D_DOC_DIR "${CMAKE_SOURCE_DIR}/doc_uni" CACHE PATH "Path to doc")
if ("${COMP3D_DOC_DIR}" STREQUAL "")
  set(COMP3D_DOC_DIR "${CMAKE_SOURCE_DIR}/doc_uni" CACHE PATH "Path to doc" FORCE)
endif()


#######################################################
##        Dependencies
#######################################################

# Eigen
find_package(Eigen3 REQUIRED NO_MODULE)

# Boost
set (BOOST_VERSION_MIN 1.60.0)
set (BOOST_COMPONENTS system filesystem date_time graph)
set (BOOST_COMPONENTS_FULL ${BOOST_COMPONENTS})
list (TRANSFORM BOOST_COMPONENTS_FULL PREPEND Boost::)
find_package(Boost ${BOOST_VERSION_MIN} REQUIRED COMPONENTS ${BOOST_COMPONENTS})

# Proj
set(PROJ_ROOT /usr/local/proj82)
find_package(PROJ REQUIRED)

# Qt
set (QT_COMPONENTS Widgets)
set (QT_COMPONENTS_FULL ${QT_COMPONENTS})
list (TRANSFORM QT_COMPONENTS_FULL PREPEND Qt5::)
find_package(Qt5 REQUIRED COMPONENTS ${QT_COMPONENTS} LinguistTools)
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
            ${CMAKE_CURRENT_SOURCE_DIR}/version.cmake)

#######################################################
##        Sources
#######################################################

set(SOURCES
    main.cpp
    lib_ex.cpp
    src/fformat.cpp
    src/filerefjson.cpp
    src/leastsquares.cpp
    src/misc_tools.cpp
    src/obs.cpp
    src/point.cpp
    src/coord.cpp
    src/projection.cpp
    src/station.cpp
    src/station_eq.cpp
    src/station_simple.cpp
    src/parameter.cpp
    src/project.cpp
    src/mathtools.cpp
    src/station_hz.cpp
    src/jsoncpp.cpp
    src/project_config.cpp
    src/datafile.cpp
    src/ellipsoid.cpp
    src/station_axis.cpp
    src/axisobs.cpp
    src/station_bascule.cpp
    src/matrixordering.cpp
    src/info.cpp
    src/varcovarmatrix.cpp
)

set(SOURCES_GUI
    gui/customtablewidgetitem.cpp
    gui/exportcoorddialog.cpp
    gui/infinityasctobasdialog.cpp
    gui/sinexdialog.cpp
    gui/maintables.cpp
    gui/mainwindow.cpp
    gui/calcthread.cpp
    gui/configdialog.cpp
    gui/errordialog.cpp
    gui/applytransfodialog.cpp
    gui/sightmatrixdialog.cpp
    gui/conversiondialog.cpp
    gui/dialogpreferences.cpp
    gui/framewidget.cpp
    gui/multisorttable.cpp
    gui/exportpointsdialog.cpp
)

set(HEADERS
    src/info.h
    src/fformat.h
    src/filerefjson.h
    src/leastsquares.h
    src/misc_tools.h
    src/obs.h
    src/point.h
    src/coord.h
    src/projection.h
    src/station.h
    src/station_eq.h
    src/station_simple.h
    src/parameter.h
    src/project.h
    src/mathtools.h
    src/station_hz.h
    src/json/json-forwards.h
    src/json/json.h
    src/compile.h
    src/project_config.h
    src/datafile.h
    src/ellipsoid.h
    src/station_axis.h
    src/axisobs.h
    src/station_bascule.h
    src/matrixordering.h
    src/uni_stream.h
    src/varcovarmatrix.h
    gui/customtablewidgetitem.h
    gui/exportcoorddialog.h
    gui/infinityasctobasdialog.h
    gui/sinexdialog.h
    gui/maintables.h
    gui/mainwindow.h
    gui/calcthread.h
    gui/configdialog.h
    gui/errordialog.h
    gui/applytransfodialog.h
    gui/sightmatrixdialog.h
    gui/conversiondialog.h
    gui/dialogpreferences.h
    gui/framewidget.h
    gui/multisorttable.h
    gui/exportpointsdialog.h
)


#######################################################
##        Translation
#######################################################
list (TRANSFORM TRANSLATIONS PREPEND "Comp3D_")
set(TS_FILES ${TRANSLATIONS})
list (TRANSFORM TS_FILES PREPEND "gui/translations/")
list (TRANSFORM TS_FILES APPEND ".ts")

set_source_files_properties(${TS_FILES} PROPERTIES OUTPUT_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/i18n")
qt5_add_translation(QM_FILES ${TS_FILES})

set (QM_FILES_QRC ${TRANSLATIONS})
list (TRANSFORM QM_FILES_QRC PREPEND "<file>i18n/")
list (TRANSFORM QM_FILES_QRC APPEND ".qm</file>\n")
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/cmake_qm.qrc" 
"
<RCC>
  <qresource>
  ${QM_FILES_QRC}
  </qresource>
</RCC>
")

#######################################################
##        Target
#######################################################

add_executable(${COMP3D_TARGET} ${SOURCES} ${SOURCES_GUI} ${HEADERS} ressource.qrc ${CMAKE_BINARY_DIR}/cmake_qm.qrc ${CMAKE_BINARY_DIR}/git_revision.cpp )

target_include_directories (${COMP3D_TARGET} PRIVATE src gui . ${PROJ_INCLUDE_DIRS})
target_link_libraries (${COMP3D_TARGET} PRIVATE COMP3D_compiler_flags ${BOOST_COMPONENTS_FULL} ${QT_COMPONENTS_FULL} ${PROJ_LIBRARIES} Eigen3::Eigen)

target_compile_definitions(${COMP3D_TARGET} PRIVATE
	COMP3D_DOC_DIR="${COMP3D_DOC_DIR}"
	$<$<BOOL:${ADD_PROJ_CC}>:ADD_PROJ_CC>
	$<$<BOOL:${ADD_PROJ_NTF}>:ADD_PROJ_NTF>
	$<$<BOOL:${ADD_PROJ_UTM}>:ADD_PROJ_UTM>
	$<$<BOOL:${USE_AUTO}>:USE_AUTO>
	$<$<BOOL:${USE_SIM}>:USE_SIM>
	$<$<BOOL:${USE_QT}>:USE_QT>
	$<$<BOOL:${USE_RES}>:USE_RES>
	$<$<BOOL:${USE_GUI}>:USE_GUI>
)
