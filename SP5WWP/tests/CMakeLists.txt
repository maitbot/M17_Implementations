option(UNITY_TEST_URL "Location of Unity Test" "undefined")

if(EXISTS ${UNITY_TEST_URL}/CMakeLists.txt)
    # Use locally unpacked source
    set(UNITY_ROOT ../unity)
elseif(EXISTS ../unity/CMakeLists.txt)
    # Use the unpacked source
    set(UNITY_ROOT ${UNITY_TEST_URL}/unity)

elseif(NOT (${UNITY_TEST_URL} STREQUAL "undefined"))

  include(FetchContent)
  FetchContent_Declare( Unity
    URL ${UNITY_TEST_URL}
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )

  FetchContent_GetProperties(Unity)
  if(NOT Unity_POPULATED)
    FetchContent_Populate(Unity)
  endif()
  set(UNITY_ROOT ${CMAKE_BINARY_DIR}/_deps/unity-src)

else()
    # Convenience copy from https://github.com/ThrowTheSwitch/Unity/releases
    message("Not building Unity Tests")
    message("-- maybe configure with -DUNITY_TEST_URL=https://github.com/ThrowTheSwitch/Unity/archive/refs/tags/v2.5.2.tar.gz")
    return()
endif()

add_executable(unit_tests unit_tests.c ${UNITY_ROOT}/src/unity.c)
target_include_directories(unit_tests PRIVATE ${UNITY_ROOT}/src)
target_link_libraries(unit_tests PRIVATE ${lm17})
add_test(NAME unit_tests COMMAND unit_tests)
