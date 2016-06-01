cmake_minimum_required(VERSION 2.6)

project(lib)
set(EXECUTABLE_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/bin")
include_directories("${PROJECT_SOURCE_DIR}/include")

add_compile_options("-std=c++11")
add_compile_options("-Wall")
add_compile_options("-Wextra")

file(GLOB_RECURSE SRCS src/*)

if(SRCS)
  set(LIB ${PROJECT_NAME})
  set(LIBRARY_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/lib")
  add_library(${LIB} SHARED ${SRCS})
  install(TARGETS ${LIB} DESTINATION lib)
endif()

file(GLOB_RECURSE INCLUDES include/*)
install(FILES ${INCLUDES} DESTINATION include)

option(BOOST_TESTS
  "Enables/Disables Boost unit test framework" OFF)

if(BOOST_TESTS)
  enable_testing()

  find_package(Boost COMPONENTS unit_test_framework REQUIRED)
  include_directories(src test ${Boost_INCLUDE_DIRS})
  add_definitions(-DBOOST_TEST_DYN_LINK)

  file(GLOB_RECURSE TESTS test/*)
  foreach(TEST ${TESTS})
    get_filename_component(TEST_NAME ${TEST} NAME_WE)
    add_executable(${TEST_NAME}_test ${TEST})
    add_test(NAME ${TEST_NAME} COMMAND ${TEST_NAME}_test)
    target_link_libraries(${TEST_NAME}_test ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY})
    if(SRCS)
      target_link_libraries(${TEST_NAME}_test ${LIB})
    endif()
  endforeach()
endif(BOOST_TESTS)

find_package(Doxygen)
if(DOXYGEN_FOUND)
  add_custom_target(doc ${DOXYGEN_EXECUTABLE}
                    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
else(DOXYGEN_FOUND)
  message(STATUS "Doxygen not found - Documentation will not be generated")
endif(DOXYGEN_FOUND)
