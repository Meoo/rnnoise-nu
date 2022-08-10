cmake_minimum_required (VERSION 3.8.0)
project (rnnoise-nu C)

include(GNUInstallDirs)
include(CheckLibraryExists)

option(FIXED_POINT  "FIXED_POINT used" OFF)
if(MSVC)
  add_definitions(-D_USE_MATH_DEFINES)
  if(FIXED_POINT)
    add_definitions(-DFIXED_POINT)
  endif()
endif()

add_definitions(-DRNNOISE_BUILD)
if (BUILD_SHARED_LIBS)
  add_definitions(-DDLL_EXPORT)
endif()

include_directories(src include)

set(CMAKE_DEBUG_POSTFIX d)

file(STRINGS "configure.ac"
    _rnnoisenu_version_defines REGEX "RNNOISENU_(CURRENT|REVISION|AGE)_VERSION=([0-9]+)$")
foreach(ver ${_rnnoisenu_version_defines})
    if(ver MATCHES "RNNOISENU_(CURRENT|REVISION|AGE)_VERSION=([0-9]+)$")
        set(RNNOISENU_${CMAKE_MATCH_1}_VERSION "${CMAKE_MATCH_2}" CACHE INTERNAL "")
    endif()
endforeach()
set(RNNOISENU_VERSION ${RNNOISENU_MAJOR_VERSION}.${RNNOISENU_MINOR_VERSION}.${RNNOISENU_MICRO_VERSION})

set(SRC
  "src/celt_lpc.c"
  "src/denoise.c"
  "src/kiss_fft.c"
  "src/models.c"
  "src/models/bd.c"
  "src/models/cb.c"
  "src/models/lq.c"
  "src/models/mp.c"
  "src/models/sh.c"
  "src/pitch.c"
  "src/rnn.c"
  "src/rnn_data.c"
)

add_library(rnnoise-nu ${SRC})

set_property(TARGET rnnoise-nu PROPERTY C_STANDARD 90)

add_executable(rnnoise_demo "examples/rnnoise_demo.c")
target_link_libraries(rnnoise_demo rnnoise-nu)

install(
  TARGETS rnnoise-nu
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)

if(NOT DISABLE_INSTALL_HEADERS)
  install(DIRECTORY include/ DESTINATION include FILES_MATCHING PATTERN "*.h")
endif()

# pkgconfig file
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})
set(includedir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR})
set(LIBM )
check_library_exists(m ceil "" LIBMEXIST)
if(LIBMEXIST)
    list(APPEND LIBM -lm)
endif()
set(SPEEXLIB "${CMAKE_PROJECT_NAME}")
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(SPEEXLIB "${CMAKE_PROJECT_NAME}d")
endif()
configure_file(rnnoise-nu.pc.in rnnoise-nu.pc @ONLY)
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/rnnoise-nu.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
