cmake_minimum_required(VERSION 2.8)
cmake_policy(SET CMP0063 NEW)
cmake_policy(SET CMP0054 OLD)

project(ginan)

enable_language(Fortran)

set (CMAKE_Fortran_SOURCE_FILE_EXTENSIONS "f95;F90;f90;for;f;F")
set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/../../lib")
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/../../lib")
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/../../bin")

#this bit doesn't work on DARWIN (MAC)
#have to modify cmake configure file directly
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin") 
	message("found DARWIN")
endif ()
file(GLOB_RECURSE sources fortran/*.f90 fortran/*.F90 fortran/*.f fortran/*.f95 fortran/*.for fortran/*.F)
foreach (file ${sources})
	if ( ${CMAKE_SYSTEM_NAME} MATCHES "Darwin") 
		execute_process(COMMAND /usr/bin/basename    ${file} OUTPUT_VARIABLE fortranfile)
		#message("found DARWIN")
	else()
		execute_process(COMMAND /usr/bin/basename -z ${file} OUTPUT_VARIABLE fortranfile)
	endif()
	set_source_files_properties(fortran/${fortranfile} PROPERTIES LANGUAGE Fortran)
	#set_source_files_properties(fortran/${fortranfile} PROPERTIES COMPILE_FLAGS "-O3 -frecursive")
	#message("set fortran/${fortranfile} to fortran compile")
endforeach (file ${sources})

set(CMAKE_MODULE_PATH					"${CMAKE_SOURCE_DIR}/cmake")
set(CMAKE_CXX_VISIBILITY_PRESET     	hidden)
set(CMAKE_VERBOSE_MAKEFILE				OFF)

set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -std=c++2a")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -ggdb3")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -fpermissive")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-write-strings")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-format-overflow")
# 	set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-c++11-narrowing")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-unused-but-set-variable")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-sign-compare")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-stringop-truncation")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-unused-variable")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-switch")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-dangling-else")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-misleading-indentation")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-var-tracking-assignments")
set(CMAKE_CXX_FLAGS						"${CMAKE_CXX_FLAGS} -Wno-unknown-warning-option")

set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -g")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -frecursive")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -fall-intrinsics")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -Wall")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -fcheck=all")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -fbacktrace")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -Wno-unused-dummy-argument")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -Wno-conversion")
set (CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -Wno-surprising")


option(ENABLE_UNIT_TESTS        "ENABLE_UNIT_TESTS"         OFF)
option(BUILD_DOC                "BUILD_DOCUMENTATION"       ON)
option(ENABLE_PARALLELISATION   "ENABLE_PARALLELISATION"    ON)
option(ENABLE_OPTIMISATION      "ENABLE_OPTIMISATION"       ON)

if(ENABLE_UNIT_TESTS)
	message(STATUS "Setting unit tests      on")
endif()

if(ENABLE_OPTIMISATION)
	message(STATUS "Setting optimisation    on")
	set(CMAKE_CXX_FLAGS					"${CMAKE_CXX_FLAGS} -O3")
	set(CMAKE_C_FLAGS					"${CMAKE_C_FLAGS} -O3")
	set(CMAKE_Fortran_FLAGS				"${CMAKE_Fortran_FLAGS} -O3")
endif()

if(ENABLE_PARALLELISATION)
	message(STATUS "Setting parallelisation on")
	find_package(OpenMP REQUIRED)
	set (CMAKE_CXX_FLAGS				"${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
	set (CMAKE_C_FLAGS					"${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set (CMAKE_Fortran_FLAGS			"${CMAKE_Fortran_FLAGS} ${OpenMP_Fortran_FLAGS}")
endif()


# check if Doxygen is installed
if (BUILD_DOC)
	message(STATUS "Setting docs            on")
	find_package(Doxygen)
	if (DOXYGEN_FOUND)

		# set input and output files
		set(DOXYGEN_IN		doc_templates/Doxyfile.in)
		set(DOXYGEN_OUT		build/Doxyfile)
		set(DOXYGEN_LAYIN	doc_templates/DoxygenLayout.xml.in)
		set(DOXYGEN_LAYOUT	build/DoxygenLayout.xml)


		# request to configure the file
		configure_file(${DOXYGEN_IN}		${DOXYGEN_OUT}		@ONLY)
		configure_file(${DOXYGEN_LAYIN}		${DOXYGEN_LAYOUT}		@ONLY)


		# note the option ALL which allows to build the docs together with the application
		add_custom_target( docs
			ALL
			COMMAND				${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/${DOXYGEN_OUT}
			WORKING_DIRECTORY	${CMAKE_CURRENT_SOURCE_DIR}/build
			COMMENT				"Generating API documentation with Doxygen"
			VERBATIM )

		list(APPEND CTEST_CUSTOM_WARNING_EXCEPTION
		".*warning: ignoring loop annotation.*"
		".*warning: .*Consider increasing DOT_GRAPH_MAX_NODES."
		)
	else (DOXYGEN_FOUND)

		message("Doxygen need to be installed to generate the doxygen documentation")

	endif (DOXYGEN_FOUND)
endif()


#set debug if nothing else is set.
set(default_build_type "Debug")
if (NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE ${default_build_type})
endif()

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads REQUIRED)
if (CMAKE_USE_PTHREADS_INIT)
	set (CMAKE_CXX_FLAGS				"${CMAKE_CXX_FLAGS} -pthread")
	set (CMAKE_Fortran_FLAGS			"${CMAKE_Fortran_FLAGS} -pthread")
endif()

set(YAML_CPP_USE_STATIC_LIBS ON)
find_package(YAML_CPP 0.6.2 REQUIRED)

find_package(OpenSSL REQUIRED)

set(Boost_NO_SYSTEM_PATHS ON)
set(Boost_USE_STATIC_LIBS ON)
find_package(Boost 1.73.0 REQUIRED COMPONENTS log log_setup date_time filesystem system thread program_options serialization timer)

find_package(Eigen3 3.3.0)
include_directories(${EIGEN3_INCLUDE_DIRS})

find_package(mongocxx	REQUIRED)
find_package(bsoncxx	REQUIRED)

set(OPENBLAS_USE_STATIC_LIBS ON)
set(BLA_VENDOR OpenBLAS)

find_package(BLAS)
if(BLAS_FOUND)
	set(LAPACK_LIBRARIES "")
	message(STATUS "Found BLAS library:       " ${BLA_VENDOR})
else()
	set (BLA_VENDOR "")
	find_package(LAPACK REQUIRED)
	find_package(BLAS   REQUIRED)  
	message(STATUS "Found LAPACK and BLAS")
endif()

if (YAML_CPP_LIB)
	message(STATUS "Found YAML library:       " ${YAML_CPP_LIB})
else()
	message(STATUS "YAML was not found")
endif()

if (Eigen3_FOUND)
	message(STATUS "Found Eigen version:      " ${Eigen3_VERSION})
else()
	message(STATUS "Eigen was not found")
endif()

if (Boost_FOUND)
	message(STATUS "Found Boost version:      " ${Boost_VERSION})
else()
	message(STATUS "Boost was not found")
endif()

if (mongocxx_FOUND)
	message(STATUS "Found Mongocxx version:   " ${mongocxx_VERSION})
else()
	message(STATUS "Mongocxx was not found")
endif()

	message(STATUS "Found C++ compiler:       " ${CMAKE_CXX_COMPILER_VERSION})


IF(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../.git)
	FIND_PACKAGE(Git)
	IF(GIT_FOUND)
		EXECUTE_PROCESS(
		COMMAND ${GIT_EXECUTABLE} describe --tags --always
		WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
		OUTPUT_VARIABLE "GINAN_COMMIT_VERSION"
			OUTPUT_STRIP_TRAILING_WHITESPACE)

		EXECUTE_PROCESS(
				COMMAND ${GIT_EXECUTABLE} rev-parse --abbrev-ref HEAD
				WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
			OUTPUT_VARIABLE "GINAN_BRANCH_NAME"
				OUTPUT_STRIP_TRAILING_WHITESPACE)

		EXECUTE_PROCESS(
			COMMAND ${GIT_EXECUTABLE} log -1 --format=%cd --date=local
			WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
		OUTPUT_VARIABLE "GINAN_COMMIT_DATE"
		OUTPUT_STRIP_TRAILING_WHITESPACE)

		MESSAGE(STATUS "Git branch tag:           ${GINAN_COMMIT_VERSION}")
		MESSAGE(STATUS "Git branch:               ${GINAN_BRANCH_NAME}")
	ELSE(GIT_FOUND)
		SET(GINAN_COMMIT_VERSION 0)
		SET(GINAN_BRANCH_NAME "unknown branch")
		MESSAGE( STATUS "Git not found: ${GINAN_COMMIT_VERSION}" )
	ENDIF(GIT_FOUND)
ELSE()
	MESSAGE( STATUS "Git not found in ${CMAKE_CURRENT_SOURCE_DIR}/..")
ENDIF()


link_directories(/usr/lib64 ${CMAKE_BINARY_DIR}/../lib)



include(CMakeLists-C.txt)
include(CMakeLists-F.txt)

					 
message(STATUS "CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")
message(STATUS "CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "CMAKE_CXX_FLAGS:         ${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_Fortran_FLAGS:     ${CMAKE_Fortran_FLAGS}")

macro(print_all_variables)
	message(STATUS "print_all_variables------------------------------------------{")
	get_cmake_property(_variableNames VARIABLES)
	foreach (_variableName ${_variableNames})
		message(STATUS "${_variableName}=${${_variableName}}")
	endforeach()
	message(STATUS "print_all_variables------------------------------------------}")
endmacro()

#print_all_variables()
