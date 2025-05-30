CMAKE_MINIMUM_REQUIRED( VERSION 2.8 )
if(POLICY CMP0042)
   # policies not known to CMake 2.8 ...
   cmake_policy(SET CMP0042 NEW) # Mac OS -rpath behavior
endif(POLICY CMP0042)

# declare the project name
PROJECT(genfit2 LANGUAGES CXX)

# set project version
SET( ${PROJECT_NAME}_VERSION_MAJOR 2 )
SET( ${PROJECT_NAME}_VERSION_MINOR 2 )
SET( ${PROJECT_NAME}_VERSION_PATCH 0 )

include(GNUInstallDirs)

# install destinations can be passed via the command line:
# cmake -DCMAKE_INSTALL_LIBDIR:PATH=.... -DINCLUDE_OUTPUT_DIRECTORY:PATH=....
# otherwise, set default install destinations
IF(NOT DEFINED INCLUDE_OUTPUT_DIRECTORY)
	set( INCLUDE_OUTPUT_DIRECTORY ${CMAKE_INSTALL_INCLUDEDIR} )
ENDIF()

# define path to macros and settings
LIST( APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake )
# include the macros and settings
INCLUDE(settings)
INCLUDE(macros)
INCLUDE(genfit)

FIND_PACKAGE(ROOT REQUIRED COMPONENTS Geom)
IF(ROOT_VERSION VERSION_LESS 6.20.0)
	MESSAGE(FATAL_ERROR "GenFit requires a ROOT version > 6.20.00")
ENDIF()
set(CMAKE_CC_FLAGS "${CMAKE_CC_FLAGS} ${ROOT_CC_FLAGS}")


# either, the environment variable RAVEPATH has to be specified, and RAVE's RaveConfig.cmake will be used to determine everything we need
# or, Rave_LDFLAGS, Rave_INCLUDE_DIRS and Rave_CFLAGS have to be set via the command-line


IF(DEFINED ENV{RAVEPATH})
	MESSAGE(STATUS "Environment variable RAVEPATH is set. Try to build GFRave.")
	# get compiler flags from rave
	INCLUDE(FindPkgConfig)
	SET(ENV{PKG_CONFIG_PATH} $ENV{RAVEPATH})
	find_package(Rave REQUIRED)
	
	SET(RAVE True)
ELSEIF(DEFINED Rave_LDFLAGS)
	IF(DEFINED Rave_INCLUDE_DIRS)
		IF(DEFINED Rave_CFLAGS)
			MESSAGE(STATUS "Rave_LDFLAGS, Rave_INCLUDE_DIRS and Rave_CFLAGS are set. Try to build GFRave.")
			SET(RAVE True)
		ENDIF()
	ENDIF()
ELSE()
   	MESSAGE(STATUS "No RAVEPATH or Rave flags given. GFRave will not be built.")
ENDIF()

IF(DEFINED RAVE)
	# turn comma separated lists into space separated strings
	string (REPLACE ";" " " Rave_LDFLAGS_STR "${Rave_LDFLAGS}")
	string (REPLACE ";" " " Rave_INCLUDE_DIRS_STR "${Rave_INCLUDE_DIRS}")
	string (REPLACE ";" " " Rave_CFLAGS_STR "${Rave_CFLAGS}")

	SET (CMAKE_CXX_FLAGS ${Rave_DEFINITIONS} ${Rave_CFLAGS_STR})
	SET(GF_INC_DIRS
		./GFRave/include/ ${Rave_INCLUDE_DIRS_STR}
	)
	
	AUX_SOURCE_DIRECTORY( ./GFRave/src  library_sources )
ENDIF()

# Set GENFIT include directories
SET(GF_INC_DIRS
		./core/include/
		./eventDisplay/include/
		./fields/include/
		./finitePlanes/include/
		./fitters/include/
		./GBL/include/
		./measurements/include/
		./trackReps/include/
		./utilities/include/
		${GF_INC_DIRS}
)

INSTALL(DIRECTORY ${GF_INC_DIRS}
        DESTINATION ${INCLUDE_OUTPUT_DIRECTORY}
        PATTERN ".svn" EXCLUDE)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -Wpedantic -Wall -Wextra \
 -Wno-long-long -Wshadow -Werror=overloaded-virtual -Wunused-parameter -Waddress")

SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -O0 -g")
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3")
SET(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS} -O3 -g")


INCLUDE_DIRECTORIES(
		BEFORE
		${ROOT_INCLUDE_DIRS}
		SYSTEM ${EIGEN3_INCLUDE_DIR}
		${GF_INC_DIRS}
)

FIND_PACKAGE(Doxygen)
IF(DOXYGEN_FOUND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile @ONLY)

    ADD_CUSTOM_TARGET(doc
        ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/doc
        COMMENT "Generating API documentation with Doxygen" VERBATIM)
ELSE()
    MESSAGE( STATUS "Doxygen not installed or found -- API documentation can't be generated!" )
ENDIF(DOXYGEN_FOUND)

# add the list of source code files -- careful, when a new file is added!
AUX_SOURCE_DIRECTORY( ./core/src          library_sources )
AUX_SOURCE_DIRECTORY( ./eventDisplay/src  library_sources )
AUX_SOURCE_DIRECTORY( ./fields/src        library_sources )
AUX_SOURCE_DIRECTORY( ./finitePlanes/src  library_sources )
AUX_SOURCE_DIRECTORY( ./fitters/src       library_sources )
AUX_SOURCE_DIRECTORY( ./GBL/src           library_sources )
AUX_SOURCE_DIRECTORY( ./measurements/src  library_sources )
AUX_SOURCE_DIRECTORY( ./trackReps/src     library_sources )
AUX_SOURCE_DIRECTORY( ./utilities/src     library_sources )

# Dictionary generation.  For the time being, we list classes one-by-one.
SET(CORE_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsFinitePlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsFitterInfo.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsHMatrix.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsTrackRep.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/DetPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/FitStatus.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/IO.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/Material.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasuredStateOnPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementFactory.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementOnPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/MeasurementProducer.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/StateOnPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/AbsMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/ThinScatterer.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/Track.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackCand.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackCandHit.h
		${CMAKE_CURRENT_SOURCE_DIR}/core/include/TrackPoint.h
		${CMAKE_CURRENT_SOURCE_DIR}/finitePlanes/include/RectangularFinitePlane.h
)
ROOT_GENERATE_DICTIONARY(
	    coreRootDict
		"${CORE_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/core/src/coreLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/coreRootDict.cxx" "${library_sources}")

SET(EVENTDISPLAY_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/include/EventDisplay.h
)
ROOT_GENERATE_DICTIONARY(
	    eventDisplayRootDict
		"${EVENTDISPLAY_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/eventDisplay/src/eventDisplayLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/eventDisplayRootDict.cxx" "${library_sources}")

SET(FITTERS_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/AbsKalmanFitter.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFittedStateOnPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitStatus.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/ReferenceStateOnPlane.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitterRefTrack.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitter.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/KalmanFitterInfo.h
		${CMAKE_CURRENT_SOURCE_DIR}/fitters/include/DAF.h
)
ROOT_GENERATE_DICTIONARY(
	    fittersRootDict
		"${FITTERS_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/fitters/src/fittersLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/fittersRootDict.cxx" "${library_sources}")

SET(GBL_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GFGbl.h
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GblFitter.h
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/ICalibrationParametersDerivatives.h
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GblFitterInfo.h
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GblFitStatus.h
		${CMAKE_CURRENT_SOURCE_DIR}/GBL/include/GblData.h
)
ROOT_GENERATE_DICTIONARY(
	    GBLRootDict
        "${GBL_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/GBL/src/GBLLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/GBLRootDict.cxx" "${library_sources}")

SET(MEASUREMENTS_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixPhi.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixU.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixUnit.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixV.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/HMatrixUV.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/FullMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/PlanarMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/ProlateSpacepointMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/SpacepointMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WireMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WireMeasurementNew.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WirePointMeasurement.h
		${CMAKE_CURRENT_SOURCE_DIR}/measurements/include/WireTrackCandHit.h
)
ROOT_GENERATE_DICTIONARY(
		measurementsRootDict
		${MEASUREMENTS_DICTIONARY_SOURCES}
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/measurements/src/measurementsLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/measurementsRootDict.cxx" "${library_sources}")

SET(TRACKREPS_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/RKTrackRep.h
		${CMAKE_CURRENT_SOURCE_DIR}/trackReps/include/MplTrackRep.h
)
ROOT_GENERATE_DICTIONARY(
	    trackRepsRootDict
		"${TRACKREPS_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/trackReps/src/trackRepsLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/trackRepsRootDict.cxx" "${library_sources}")

SET(UTILITIES_DICTIONARY_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/HelixTrackModel.h
		${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/MeasurementCreator.h
		${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/mySpacepointDetectorHit.h
		${CMAKE_CURRENT_SOURCE_DIR}/utilities/include/mySpacepointMeasurement.h
)
ROOT_GENERATE_DICTIONARY(
	    utilitiesRootDict
		"${UTILITIES_DICTIONARY_SOURCES}"
		LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/utilities/src/utilitiesLinkDef.h"
)
SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/utilitiesRootDict.cxx" "${library_sources}")

foreach( dict coreRootDict eventDisplayRootDict fittersRootDict GBLRootDict measurementsRootDict trackRepsRootDict utilitiesRootDict)
  list(APPEND ROOT_PCM_FILES ${CMAKE_CURRENT_BINARY_DIR}/lib${dict}_rdict.pcm)
endforeach()

if(DEFINED RAVE)
	SET(GFRAVE_DICTIONARY_SOURCES
			${CMAKE_CURRENT_SOURCE_DIR}/GFRave/include/GFRaveTrackParameters.h
			${CMAKE_CURRENT_SOURCE_DIR}/GFRave/include/GFRaveVertex.h
	)
	ROOT_GENERATE_DICTIONARY(
		    GFRaveRootDict
			"${GFRAVE_DICTIONARY_SOURCES}"
			LINKDEF "${CMAKE_CURRENT_SOURCE_DIR}/GFRave/src/GFRaveLinkDef.h"
	)
	SET(library_sources "${CMAKE_CURRENT_BINARY_DIR}/GFRaveRootDict.cxx" "${library_sources}")
        list(APPEND ROOT_PCM_FILES ${CMAKE_CURRENT_BINARY_DIR}/libGFRaveRootDict_rdict.pcm)
endif()


# install
ADD_SHARED_LIBRARY( ${PROJECT_NAME} ${library_sources})
INSTALL(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
INSTALL(FILES ${ROOT_PCM_FILES} DESTINATION ${CMAKE_INSTALL_LIBDIR})

# link ROOT libraries
if(DEFINED RAVE)
	TARGET_LINK_LIBRARIES(
			${PROJECT_NAME}
			ROOT::Core ROOT::Physics ROOT::Eve
			${Rave_LIB}
			${Rave_LDFLAGS_STR}
	)
else()
	TARGET_LINK_LIBRARIES(${PROJECT_NAME} ROOT::Core ROOT::Physics ROOT::Eve)
endif()

INCLUDE(CTest)

ADD_CUSTOM_TARGET( tests )

ADD_GENFIT_TEST( fitterTests               test/fitterTests/main.cc)
ADD_GENFIT_TEST( minimalFittingExample     test/minimalFittingExample/main.cc)
ADD_GENFIT_TEST( minimalFittingExample2    test/minimalFittingExample/main2.cc)
ADD_GENFIT_TEST( measurementFactoryExample test/measurementFactoryExample/main.cc)
ADD_GENFIT_TEST( streamerTest              test/streamerTest/main.cc)
ADD_GENFIT_TEST( unitTests                 test/unitTests/main.cc)
IF(DEFINED RAVE)
  ADD_GENFIT_TEST( vertexingTest           test/vertexingTest/main.cc)
  ADD_GENFIT_TEST( vertexingTestRead       test/vertexingTest/read.cc)
ENDIF()

IF(BUILD_TESTING)

	# Run these automatically via ctest since they do not need a GUI. Create the
	# necessary geometry first via a dummy test
	ADD_TEST(NAME createGeometry COMMAND root -l -b -q ${CMAKE_CURRENT_LIST_DIR}/test/makeGeom.C)

	ADD_TEST(NAME streamerTest COMMAND streamerTest)
	SET_PROPERTY(TEST streamerTest PROPERTY DEPENDS createGeometry)

	ADD_TEST(NAME unitTests COMMAND unitTests)
	SET_PROPERTY(TEST unitTests PROPERTY DEPENDS createGeometry)

    ENABLE_TESTING()


	FIND_PACKAGE(GTest REQUIRED)
	ADD_EXECUTABLE(gtests
			gtest/TestConstField.cpp
			gtest/TestMaterialEffects.cpp
			gtest/TestRKTrackRep.cpp
			gtest/TestMaterial.cpp
			)
	TARGET_LINK_LIBRARIES(gtests ${GTEST_BOTH_LIBRARIES} ${ROOT_LIBS} ${PROJECT_NAME})  # gtest gtest_main
	MESSAGE(STATUS  ${GTEST_INCLUDE_DIRS})
	TARGET_INCLUDE_DIRECTORIES(gtests PUBLIC ${GTEST_INCLUDE_DIRS})
	GTEST_DISCOVER_TESTS(gtests)
	INSTALL(TARGETS gtests DESTINATION bin)
ENDIF()

# generate and install following configuration files
#GENERATE_PACKAGE_CONFIGURATION_FILES( genfitConfig.cmake genfitConfigVersion.cmake )
