cmake_minimum_required(VERSION 3.10)
project(smina)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake/Modules)

file(GLOB sminasrc
		"src/lib/*.h"
		"src/lib/*.cpp"
		"src/lib/CommandLine2/*.cpp"
)
list(APPEND sminasrc "${CMAKE_CURRENT_BINARY_DIR}/version.cpp")

file(GLOB sminaserver
		"src/server/*.h"
		"src/server/*.cpp"
)

# Boost >= 1.75 and current Eigen require C++14 or newer.
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# On macOS the architecture is deliberately left to CMake's default (the host
# arch).  Homebrew's Boost/OpenBabel are single-arch, so forcing a universal
# build here would fail to link; pass -DCMAKE_OSX_ARCHITECTURES=... if you have
# universal dependencies available.

message(${CMAKE_CURRENT_BINARY_DIR})
# Add a custom command that produces version.cpp, plus
# a dummy output that's not actually produced, in order
# to force version.cmake to always be re-run before the build
add_custom_command(
		OUTPUT  ${CMAKE_CURRENT_BINARY_DIR}/_version.cpp
		BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/version.cpp
		COMMAND ${CMAKE_COMMAND} -P
		${CMAKE_SOURCE_DIR}/CMake/version.cmake)
add_custom_command(
		OUTPUT  ${CMAKE_CURRENT_BINARY_DIR}/version.cpp
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/_version.cpp)
set_source_files_properties(${CMAKE_CURRENT_BINARY_DIR}/_version.cpp PROPERTIES SYMBOLIC TRUE)

add_library(libsmina
		${sminasrc}
)
set_target_properties(libsmina PROPERTIES OUTPUT_NAME "smina")
target_include_directories(libsmina PUBLIC src/lib/ src/lib/CommandLine2)

# boost/timer.hpp and boost/progress.hpp (used by parallel_progress.h) error out
# on Boost >= 1.76 unless deprecation is explicitly opted into.
target_compile_definitions(libsmina PUBLIC BOOST_TIMER_ENABLE_DEPRECATED)

set(SMINA_BOOST_COMPONENTS
		iostreams
		timer
		system
		regex
		thread
		serialization
		filesystem
		program_options
		date_time
)

# Prefer the CMake package config Boost itself installs (>= 1.70).  It is the
# only option on CMake >= 3.30, where FindBoost is deprecated, and it is also
# the only finder that knows Boost.System is header-only in recent releases --
# the legacy FindBoost module reports "missing: system" against those.
find_package(Boost CONFIG QUIET COMPONENTS ${SMINA_BOOST_COMPONENTS})
if(NOT Boost_FOUND)
	# Older installs ship no BoostConfig.cmake; fall back to the find module.
	find_package(Boost MODULE REQUIRED COMPONENTS ${SMINA_BOOST_COMPONENTS})
endif()

foreach(component IN LISTS SMINA_BOOST_COMPONENTS)
	target_link_libraries(libsmina PUBLIC Boost::${component})
endforeach()
message("-- Found Boost ${Boost_VERSION} include files at ${Boost_INCLUDE_DIRS}")

# Boost's package config pulls this in for Boost::thread, but the FindBoost
# fallback does not, and the executables below link Threads::Threads.
find_package(Threads REQUIRED)

find_package(Eigen3 REQUIRED)
target_link_libraries(libsmina PUBLIC Eigen3::Eigen)
message("-- Found Eigen3 ${Eigen3_VERSION}")

#flexible docking implementation requires features of openbabel3
find_package(OpenBabel3 REQUIRED)
target_include_directories(libsmina PUBLIC ${OPENBABEL3_INCLUDE_DIR})
target_link_libraries(libsmina PUBLIC ${OPENBABEL3_LIBRARIES})
message("-- Found OpenBabel3 include files at ${OPENBABEL3_INCLUDE_DIR}")
message("-- Found OpenBabel3 libraries at ${OPENBABEL3_LIBRARIES}")

# libsmina's include directories are PUBLIC, so every target below inherits
# them (and Boost/Eigen/OpenBabel's) by linking against it.
add_executable(smina src/main/main.cpp)
target_link_libraries(smina libsmina Threads::Threads)

add_executable(server ${sminaserver})
target_link_libraries(server libsmina Threads::Threads)

add_executable(fromsmina src/fromsmina/fromsmina.cpp)
target_link_libraries(fromsmina libsmina)

add_executable(tosmina src/tosmina/tosmina.cpp)
target_link_libraries(tosmina libsmina)
