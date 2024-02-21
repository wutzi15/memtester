cmake_minimum_required(VERSION 3.10)

project(memtester)

add_definitions(-DPOSIX -D_POSIX_C_SOURCE=200809L -D_FILE_OFFSET_BITS=64 )

option(TEST_NARROW_WRITES "run tests on narrow writes" ON)

if(TEST_NARROW_WRITES)
add_definitions(-DTEST_NARROW_WRITES)
endif(TEST_NARROW_WRITES)

add_executable(memtester memtester.c output.c tests.c)

add_custom_target(uninstall
    COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/memtester /usr/share/man/man8/memtester.8
    )

install(TARGETS memtester DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/memtester.8 DESTINATION /usr/share/man/man8)