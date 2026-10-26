cmake_minimum_required(VERSION 3.22)
message("Using toolchain file ${CMAKE_TOOLCHAIN_FILE}.")

########################################################################################################################
## Define project.
########################################################################################################################
project(
    FormListManipulator
    VERSION 1.7.0.0
    DESCRIPTION "Allows dynamic modification of FormLists."
    LANGUAGES CXX
)
set(PROJECT_FRIENDLY_NAME "Formlist Manipulator")
set(PROJECT_AUTHOR "MaskedRPGFan")
set(PROJECT_AUTHOR_EMAIL "maskedrpgfan@gmail.com")

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Plugin.hpp.in
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Plugin.hpp
    @ONLY
)

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.rc.in
    ${CMAKE_CURRENT_BINARY_DIR}/version.rc
    @ONLY
)

set(sources
		Src/main.cpp
		Src/Plugin.hpp
		Src/Manipulator/EventManager.hpp
		Src/Manipulator/Manipulator.hpp
		Src/Manipulator/RegisterFuncs.hpp
		Src/Utility/ExternalModules.hpp
		Src/Utility/LogInfo.hpp
		Src/Utility/Utility.hpp
		Src/Utility/Types/EntryType.hpp
		Src/Utility/Types/FormListType.hpp
		Src/Utility/Types/InfoType.hpp
		Src/Utility/Types/Keywords.hpp
		Src/Utility/Types/OperatingMode.hpp
		Src/Utility/Types/Types.hpp
        ${CMAKE_CURRENT_BINARY_DIR}/version.rc
	)

source_group(
        TREE ${CMAKE_CURRENT_SOURCE_DIR}
        FILES
        ${sources})


########################################################################################################################
## Configure target DLL.
########################################################################################################################
include(FetchContent)
set(BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(SKSE_SUPPORT_XBYAK ON CACHE BOOL "" FORCE)
set(SKSE_SUPPORT_PATCH_SAFETY OFF CACHE BOOL "" FORCE)
FetchContent_Declare(
	CommonLibSSE
	GIT_REPOSITORY https://github.com/alandtse/CommonLibSSE-NG.git
	GIT_TAG v13.1.0
	GIT_SHALLOW TRUE
)
FetchContent_MakeAvailable(CommonLibSSE)
include("${commonlibsse_SOURCE_DIR}/cmake/CommonLibSSE.cmake")
find_package(Boost CONFIG REQUIRED COMPONENTS regex)
find_package(unordered_dense CONFIG REQUIRED)
find_package(tsl-ordered-map CONFIG REQUIRED)
find_path(MERGEMAPPER_INCLUDE_DIRS "MergeMapperPluginAPI.h")
find_path(CLIB_UTIL_INCLUDE_DIRS "ClibUtil/utils.hpp")

add_commonlibsse_plugin(
	${PROJECT_NAME}
	AUTHOR "${PROJECT_AUTHOR}"
	EMAIL "${PROJECT_AUTHOR_EMAIL}"
	VERSION ${PROJECT_VERSION}
	SOURCES ${sources} 
	${MERGEMAPPER_INCLUDE_DIRS}/MergeMapperPluginAPI.cpp
	)
add_library("${PROJECT_NAME}::${PROJECT_NAME}" ALIAS "${PROJECT_NAME}")

target_include_directories(${PROJECT_NAME}
        PRIVATE
		${CMAKE_CURRENT_BINARY_DIR}/include
		${CMAKE_CURRENT_SOURCE_DIR}/src
		${MERGEMAPPER_INCLUDE_DIRS}
		${CLIB_UTIL_INCLUDE_DIRS}
        )

target_link_libraries(
	${PROJECT_NAME}
	PRIVATE
		unordered_dense::unordered_dense
		tsl::ordered_map
		Boost::regex
)

target_precompile_headers(${PROJECT_NAME}
        PRIVATE
        Src/Plugin.hpp)


