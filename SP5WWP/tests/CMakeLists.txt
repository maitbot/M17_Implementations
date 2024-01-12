find_package(unity)

if(unity_FOUND)
 add_executable(unit_tests unit_tests.c)
 target_link_libraries(unit_tests PRIVATE ${lm17} unity::framework)
 add_test(NAME unit_tests COMMAND unit_tests)
endif()

find_program(GRCC NAMES grcc)

if(EXISTS ${GRCC})
message("Got GRCC: " ${GRCC})
add_custom_command(
    COMMENT "Build Python flowgraphs"
    DEPENDS ${CMAKE_SOURCE_DIR}/grc/m17_streamer.grc
    COMMAND ${GRCC} ${CMAKE_SOURCE_DIR}/grc/m17_streamer.grc
    COMMAND mkfifo ../stream_fifo
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/m17_streamer.py )
add_custom_target(m17_streamer ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/m17_streamer.py)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/test_streaming.sh "#!/bin/sh -v\n")
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/test_streaming.sh "./m17_streamer.py &\n")
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/test_streaming.sh "cat ../stream_fifo | ../m17-coder-sym | ../m17-decoder-sym\n")
file(CHMOD ${CMAKE_CURRENT_BINARY_DIR}/test_streaming.sh PERMISSIONS OWNER_READ OWNER_EXECUTE)

add_test(NAME stream_test COMMAND ${CMAKE_CURRENT_BINARY_DIR}/test_streaming.sh)
set_property(TEST stream_test PROPERTY ENVIRONMENT "QT_QPA_PLATFORM=offscreen")
set_property(TEST stream_test PROPERTY TIMEOUT 5)
set_property(TEST stream_test PROPERTY WILLFAIL)
endif()
