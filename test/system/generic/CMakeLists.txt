collect (PROJECT_LIB_TESTS main.c)
collect (PROJECT_LIB_TESTS alloc.c)
collect (PROJECT_LIB_TESTS irq.c)
collect (PROJECT_LIB_TESTS mutex.c)
collect (PROJECT_LIB_TESTS atomic.c)
collect (PROJECT_LIB_TESTS threads.c)

collect (PROJECT_LIB_TEST_ALL threads.c)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_MACHINE})
  add_subdirectory(${PROJECT_MACHINE})
endif (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_MACHINE})

