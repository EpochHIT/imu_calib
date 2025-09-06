cmake_minimum_required(VERSION 2.8.3)
project(code_utils)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++11")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -fPIC -fopenmp")

find_package(catkin REQUIRED
    roscpp
    std_msgs
    )

#set(EIGEN_INCLUDE_DIR "/usr/local/include/eigen3")
find_package(Eigen3 REQUIRED)

# Force OpenCV 4.5.4 exactly; allow passing a hint path via -DOpenCV_DIR_HINT
set(OpenCV_DIR_HINT "" CACHE PATH "Path to OpenCV 4.5.4 cmake config (e.g., /usr/local/lib/cmake/opencv4)")
if(OpenCV_DIR_HINT)
    set(OpenCV_DIR ${OpenCV_DIR_HINT})
endif()
find_package(OpenCV 4.5.4 EXACT REQUIRED COMPONENTS core imgproc imgcodecs highgui)

find_package(Ceres REQUIRED)

# libdw (elfutils) for backward.hpp stack trace resolver
# Prefer pkg-config if available, otherwise fall back to plain -ldw
find_package(PkgConfig QUIET)
if(PkgConfig_FOUND)
    pkg_check_modules(LIBDW QUIET libdw)
endif()

catkin_package(
    INCLUDE_DIRS include
    LIBRARIES polynomial cv_utils pnp
    CATKIN_DEPENDS roscpp std_msgs
#    DEPENDS system_lib
    )

include_directories(
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
    ${OpenCV_INCLUDE_DIRS}
    ${CERES_INCLUDE_DIRS}
     )
include_directories("include")

add_library(polynomial STATIC
    src/math_utils/Polynomial.cpp)
target_link_libraries(polynomial ${Boost_LIBRARIES} )

add_library(pnp
    src/cv_utils/dlt/dlt.cpp
    src/cv_utils/pnp/pnp.cpp
    src/cv_utils/pnp/linearpnp.cpp
    src/cv_utils/pnp/nonlinearpnp.cpp)
target_link_libraries(pnp ${Boost_LIBRARIES}  ${OpenCV_LIBS}  ${CERES_LIBRARIES})

add_library(cv_utils STATIC
    src/cv_utils.cc
    )
target_link_libraries(cv_utils ${Boost_LIBRARIES}  ${OpenCV_LIBS} )

add_executable(matIO_test   src/mat_io_test.cpp )
if(LIBDW_FOUND)
    target_link_libraries(matIO_test ${OpenCV_LIBS} ${LIBDW_LIBRARIES})
else()
    target_link_libraries(matIO_test ${OpenCV_LIBS} dw)
endif()

add_executable(sumpixel_test   src/sumpixel_test.cpp )
if(LIBDW_FOUND)
    target_link_libraries(sumpixel_test ${OpenCV_LIBS} ${LIBDW_LIBRARIES})
else()
    target_link_libraries(sumpixel_test ${OpenCV_LIBS} dw)
endif()
