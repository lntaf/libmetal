collector_create (PROJECT_LIB_TESTS "${CMAKE_CURRENT_SOURCE_DIR}")
collector_create (PROJECT_LIB_TEST_ALL "${CMAKE_CURRENT_SOURCE_DIR}")

add_subdirectory (system)

collect (PROJECT_LIB_HEADERS metal-test.h)

collect (PROJECT_LIB_TESTS version.c)
collect (PROJECT_LIB_TESTS metal-test.c)

collect (PROJECT_LIB_HEADERS test_all.h)
collect (PROJECT_LIB_TEST_ALL test_all.c)

collector_list  (_hdirs PROJECT_INC_DIRS)
include_directories (${_hdirs} ${CMAKE_CURRENT_SOURCE_DIR})

collector_list  (_ldirs PROJECT_LIB_DIRS)
link_directories (${_ldirs})

collector_list (_deps PROJECT_LIB_DEPS)

collector_list (_srcs PROJECT_LIB_TESTS)

get_property (_ec_flgs GLOBAL PROPERTY "PROJECT_EC_FLAGS")

if (WITH_ZEPHYR)
  set (_tfiles "")
  foreach (f ${_srcs})
    list(APPEND _tfiles "${CMAKE_CURRENT_SOURCE_DIR}/${f}")
  endforeach(f)
  target_sources(app PRIVATE ${_tfiles})
  target_include_directories (app PRIVATE ${_hdirs} ${CMAKE_CURRENT_SOURCE_DIR})
  set (_eldflags "")
  foreach (d ${_ldirs})
    list (APPEND _eldflags "-L${d}")
  endforeach(d)
  add_dependencies(app metal)
  list (APPEND _deps -lmetal)
  target_link_libraries(app INTERFACE "${_eldflags}" ${_deps})
  target_compile_options (app PRIVATE -DNOT_HAVE_STRERROR)
else (WITH_ZEPHYR)
  if(WITH_SHARED_LIB)
    set (_lib ${PROJECT_NAME}-shared)
    add_executable (test-${_lib} ${_srcs})
    target_link_libraries (test-${_lib} ${_deps} ${_lib})
    install (TARGETS test-${_lib} RUNTIME DESTINATION bin)
    target_compile_options (test-${_lib} PUBLIC ${_ec_flgs})
    add_dependencies (test-${_lib}  ${PROJECT_NAME}-shared)
    if (WITH_TESTS_EXEC)
      add_test (test-${_lib} test-${_lib})
    endif (WITH_TESTS_EXEC)
  endif (WITH_SHARED_LIB)

  if (WITH_STATIC_LIB)
    get_property (_linker_options GLOBAL PROPERTY TEST_LINKER_OPTIONS)
    set (_lib ${PROJECT_NAME}-static)
    add_executable (test-${_lib} ${_srcs})
    target_link_libraries (test-${_lib} -Wl,-Map=test-${_lib}.map ${_linker_options} -Wl,--start-group ${_lib} ${_deps} -Wl,--end-group)
    install (TARGETS test-${_lib} RUNTIME DESTINATION bin)
    target_compile_options (test-${_lib} PUBLIC ${_ec_flgs})
    add_dependencies (test-${_lib}  ${PROJECT_NAME}-static)
    if (WITH_TESTS_EXEC)
      add_test (test-${_lib} test-${_lib})
    endif (WITH_TESTS_EXEC)
  endif (WITH_STATIC_LIB)

  collector_list (_headers PROJECT_HDR_TESTS)
  foreach (INCLUDE ${_headers})
    string (REGEX REPLACE "[^a-zA-Z0-9]+" "-" _f ${INCLUDE})
    configure_file (metal-header-template.c ${_f}.c)
    list (APPEND _flist "${CMAKE_CURRENT_BINARY_DIR}/${_f}.c")
  endforeach (INCLUDE)
  add_library (metal-headers STATIC ${_flist})

  collector_list(_src_all PROJECT_LIB_TEST_ALL)

  if (WITH_SHARED_LIB)
    get_property (_linker_options GLOBAL PROPERTY TEST_LINKER_OPTIONS)
    set (_lib ${PROJECT_NAME}-shared)
    add_executable(test-all-${_lib} ${_src_all})
    target_link_libraries(test-all-${_lib} ${PROJECT_NAME}-shared)
    target_compile_options(test-all-${_lib} PUBLIC ${_ec_flgs})
    add_dependencies(test-all-${_lib} ${PROJECT_NAME}-shared)
    install(TARGETS test-all-${_lib} RUNTIME DESTINATION bin)
    if (WITH_TESTS_EXEC)
      add_test(test-all-${_lib} test-all-${_lib})
    endif()
  endif()

  if (WITH_STATIC_LIB)
    get_property (_linker_options GLOBAL PROPERTY TEST_LINKER_OPTIONS)
    set (_lib ${PROJECT_NAME}-static)
    add_executable (test-all-${_lib} ${_src_all})
    target_link_libraries (test-all-${_lib} -Wl,-Map=test-all-${_lib}.map ${_linker_options} -Wl,--start-group ${_lib} ${_deps} -Wl,--end-group)
    install (TARGETS test-all-${_lib} RUNTIME DESTINATION bin)
    target_compile_options (test-all-${_lib} PUBLIC ${_ec_flgs})
    add_dependencies (test-all-${_lib}  ${PROJECT_NAME}-static)
    if (WITH_TESTS_EXEC)
      add_test (test-all-${_lib} test-all-${_lib})
    endif (WITH_TESTS_EXEC)
  endif (WITH_STATIC_LIB)

#  if (WITH_STATIC_LIB)
#    add_executable(test-all-static ${_src_all})
#    target_link_libraries(test-all-static ${PROJECT_NAME}-static)
#    target_compile_options(test-all-static PUBLIC ${_ec_flgs})
#    add_dependencies(test-all-static ${PROJECT_NAME}-static)
#    install(TARGETS test-all-static RUNTIME DESTINATION bin)
#    if (WITH_TESTS_EXEC)
#      add_test(test-all-static test-all-static)
#    endif()
#  endif()

endif (WITH_ZEPHYR)

