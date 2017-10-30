cmake_minimum_required(VERSION 2.8)
project(CGoGN)

if(CMAKE_SYSTEM_NAME MATCHES "Darwin")
    set(_boost_COMPILER -clang-darwin42)
endif()

option(DISABLE_AUTO_LINK "Disable auto-link boost libraries" OFF)

find_package(Boost COMPONENTS regex thread system date_time chrono REQUIRED)
find_package(ZLIB REQUIRED)
find_package(LibXml2 REQUIRED)
find_package(Eigen3 REQUIRED)

# define includes of external libs
set (CGoGN_EXT_INCLUDES
    ${ZLIB_INCLUDE_DIRS}
    ${LIBXML2_INCLUDE_DIR}
    ${Boost_INCLUDE_DIR}
    ${EIGEN3_INCLUDE_DIR}
)

# define libs for external libs
set (CGoGN_EXT_LIBS
    ${ZLIB_LIBRARIES}
    ${LIBXML2_LIBRARIES}
    ${Boost_SYSTEM_LIBRARY}
    ${Boost_REGEX_LIBRARY}
    ${Boost_THREAD_LIBRARY}
    ${Boost_DATE_TIME_LIBRARY}
    ${Boost_CHRONO_LIBRARY}
)

if(NOT WIN32)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -fPIC")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Woverride-init -fPIC")
endif()

if(DISABLE_AUTO_LINK)
    add_definitions(-DBOOST_ALL_NO_LIB
                    -DBOOST_LINKING_PYTHON
                    )
else()
    add_definitions(-DBOOST_ALL_DYN_LINK
                    -DBOOST_LINKING_PYTHON
                    )
endif()

if(${CMAKE_BUILD_TYPE} STREQUAL Release)
    # ADD_DEFINITIONS(-DNOTOPOWARNING -DWITH_ZINRI)
else()
    add_definitions(-DBOOST_DEBUG_PYTHON
        -DBOOST_USE_DEBUG_PYTHON
        )
endif()

include_directories(
    include
    ${CGoGN_EXT_INCLUDES}
)

file(
    GLOB_RECURSE
    files_topology
    src/Topology/*.cpp
    include/Topology/*.hpp
    include/Topology/*.h
)

file(
    GLOB_RECURSE
    files_container
    src/Container/*.cpp
    include/Container/*.hpp
    include/Container/*.h
)

file(
    GLOB_RECURSE
    files_algo
    src/Algo/*.cpp
    src/Algo/*.c
    include/Algo/*.hpp
    include/Algo/*.h
)

file(
    GLOB        # WARNING NO MORE RECURSE TO AVOID TAKING QT FILES
    files_utils
    src/Utils/*.cpp
    include/Utils/*.hpp
    include/Utils/*.h
)
if(WIN32)
    add_library( cgogn STATIC ${files_topology} ${files_container} ${files_algo} ${files_utils} )
else()
    add_library( cgogn SHARED ${files_topology} ${files_container} ${files_algo} ${files_utils} )
endif()
target_link_libraries(cgogn ${CGoGN_EXT_LIBS})

install(DIRECTORY include DESTINATION include/cgogn)
install(TARGETS cgogn
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
)
