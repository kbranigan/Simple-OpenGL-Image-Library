cmake_minimum_required (VERSION 2.6)
project (SOIL)

set (SOIL_BUILD_TESTS false CACHE BOOL "Build tests")

if(ANDROID)
  # Android requires GL ES 2.0 package automatically
  set(LIBRARIES GLESv2)
else()
  find_package (OpenGL REQUIRED)
  include_directories (${OPENGL_INCLUDE_DIR})
  set(LIBRARIES ${OPENGL_LIBRARIES})
endif()

file (GLOB SOIL_SOURCES src/*.c)
file (GLOB SOIL_HEADERS src/*.h)

add_library (SOIL ${SOIL_SOURCES} ${SOIL_HEADERS})
target_link_libraries (SOIL ${LIBRARIES})

install (TARGETS SOIL
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)
install (FILES ${SOIL_HEADERS} DESTINATION include/SOIL)

if (SOIL_BUILD_TESTS)
	add_executable (test_SOIL src/test_SOIL.cpp)
	target_link_libraries (test_SOIL SOIL)
	install (TARGETS SOIL test_SOIL RUNTIME DESTINATION bin)
endif ()