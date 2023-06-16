# MIT License
#
# Copyright (c) 2016-2022 The ZLMediaKit project authors. All Rights Reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#

include(GenerateExportHeader)
file(GLOB API_SRC_LIST
  include/*.h
  source/*.c
  source/*.cpp
  source/*.h)

set(LINK_LIBRARIES ${MK_LINK_LIBRARIES})

set(MK_LIBRARY_NAME mkapi)

if(IOS)
  add_library(${MK_LIBRARY_NAME} STATIC ${API_SRC_LIST})
  target_link_libraries(${MK_LIBRARY_NAME}
    PRIVATE ${LINK_LIBRARIES})
  return()
endif ()

set(COMPILE_DEFINITIONS ${MK_COMPILE_DEFINITIONS})

if (MSVC)
  list(APPEND COMPILE_DEFINITIONS GENERATE_EXPORT)
endif ()

if(ENABLE_API_STATIC_LIB)
  add_library(${MK_LIBRARY_NAME} STATIC ${API_SRC_LIST})
  list(APPEND COMPILE_DEFINITIONS MediaKitApi_STATIC)
else()
  add_library(${MK_LIBRARY_NAME} SHARED ${API_SRC_LIST})
endif()

target_compile_definitions(${MK_LIBRARY_NAME}
  PRIVATE ${COMPILE_DEFINITIONS})
target_include_directories(${MK_LIBRARY_NAME}
  PRIVATE
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>")

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
  target_link_libraries(${MK_LIBRARY_NAME} -Wl,--start-group ${LINK_LIBRARIES} -Wl,--end-group)
elseif(CMAKE_SYSTEM_NAME MATCHES "Android")
  target_link_libraries(${MK_LIBRARY_NAME} log -Wl,--start-group ${LINK_LIBRARIES} -Wl,--end-group)
else()
  target_link_libraries(${MK_LIBRARY_NAME} ${LINK_LIBRARIES})
endif()

generate_export_header(${MK_LIBRARY_NAME}
  EXPORT_MACRO_NAME API_EXPORT
  BASE_NAME MK_API
  STATIC_DEFINE MediaKitApi_STATIC
  EXPORT_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/mk_export.h")

file(GLOB API_HEADER_LIST include/*.h ${CMAKE_CURRENT_BINARY_DIR}/*.h)

install(FILES ${API_HEADER_LIST}
  DESTINATION ${INSTALL_PATH_INCLUDE}/mkapi)

install(TARGETS ${MK_LIBRARY_NAME}
  ARCHIVE DESTINATION ${INSTALL_PATH_LIB}
  LIBRARY DESTINATION ${INSTALL_PATH_LIB}
  RUNTIME DESTINATION ${INSTALL_PATH_RUNTIME})

include(GNUInstallDirs)

# Pkg-config
configure_file(mkapi.pc.in
    "${PROJECT_BINARY_DIR}/mkapi.pc" @ONLY)

install(FILES
"${PROJECT_BINARY_DIR}/mkapi.pc"
DESTINATION ${INSTALL_PATH_LIB}/pkgconfig)

if (ENABLE_TESTS)
  add_subdirectory(tests)
endif()