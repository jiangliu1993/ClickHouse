# 'clickhouse' binary is a multi purpose tool,
# that contain multiple execution modes (client, server, etc.)
# each of them is built and linked as a separate library, defined below.

option (ENABLE_CLICKHOUSE_ALL "Enable all tools" ON)
option (ENABLE_CLICKHOUSE_SERVER "Enable clickhouse-server" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_CLIENT "Enable clickhouse-client" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_LOCAL "Enable clickhouse-local" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_BENCHMARK "Enable clickhouse-benchmark" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_PERFORMANCE "Enable clickhouse-performance-test" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_EXTRACT_FROM_CONFIG "Enable clickhouse-extract-from-config" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_COMPRESSOR "Enable clickhouse-compressor" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_COPIER "Enable clickhouse-copier" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_FORMAT "Enable clickhouse-format" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_OBFUSCATOR "Enable clickhouse-obfuscator" ${ENABLE_CLICKHOUSE_ALL})
option (ENABLE_CLICKHOUSE_ODBC_BRIDGE "Enable clickhouse-odbc-bridge" ${ENABLE_CLICKHOUSE_ALL})

configure_file (config_tools.h.in ${CMAKE_CURRENT_BINARY_DIR}/config_tools.h)

add_subdirectory (server)
add_subdirectory (client)
add_subdirectory (local)
add_subdirectory (benchmark)
add_subdirectory (performance-test)
add_subdirectory (extract-from-config)
add_subdirectory (compressor)
add_subdirectory (copier)
add_subdirectory (format)
add_subdirectory (clang)
add_subdirectory (obfuscator)
add_subdirectory (odbc-bridge)

if (CLICKHOUSE_SPLIT_BINARY)
    set (CLICKHOUSE_ALL_TARGETS clickhouse-server clickhouse-client clickhouse-local clickhouse-benchmark clickhouse-performance-test
            clickhouse-extract-from-config clickhouse-compressor clickhouse-format clickhouse-copier clickhouse-odbc-bridge)

    if (USE_EMBEDDED_COMPILER)
        list (APPEND CLICKHOUSE_ALL_TARGETS clickhouse-clang clickhouse-lld)
    endif ()

    set_target_properties(${CLICKHOUSE_ALL_TARGETS} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ..)

    add_custom_target (clickhouse-bundle ALL DEPENDS ${CLICKHOUSE_ALL_TARGETS})
    add_custom_target (clickhouse ALL DEPENDS clickhouse-bundle)
else ()
    if (USE_EMBEDDED_COMPILER)
        # before add_executable !
        link_directories (${LLVM_LIBRARY_DIRS})
    endif ()
    add_executable (clickhouse main.cpp)
    target_link_libraries (clickhouse clickhouse_common_io)
    target_include_directories (clickhouse BEFORE PRIVATE ${COMMON_INCLUDE_DIR})
    target_include_directories (clickhouse PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

    if (ENABLE_CLICKHOUSE_SERVER)
        target_link_libraries (clickhouse clickhouse-server-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_CLIENT)
        target_link_libraries (clickhouse clickhouse-client-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_LOCAL)
        target_link_libraries (clickhouse clickhouse-local-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_BENCHMARK)
        target_link_libraries (clickhouse clickhouse-benchmark-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_PERFORMANCE)
        target_link_libraries (clickhouse clickhouse-performance-test-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_COPIER)
        target_link_libraries (clickhouse clickhouse-copier-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_EXTRACT_FROM_CONFIG)
        target_link_libraries (clickhouse clickhouse-extract-from-config-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_COMPRESSOR)
        target_link_libraries (clickhouse clickhouse-compressor-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_FORMAT)
        target_link_libraries (clickhouse clickhouse-format-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_OBFUSCATOR)
        target_link_libraries (clickhouse clickhouse-obfuscator-lib)
    endif ()
    if (USE_EMBEDDED_COMPILER)
        target_link_libraries (clickhouse clickhouse-compiler-lib)
    endif ()
    if (ENABLE_CLICKHOUSE_ODBC_BRIDGE)
        target_link_libraries (clickhouse clickhouse-odbc-bridge-lib)
    endif()

    set (CLICKHOUSE_BUNDLE)
    if (ENABLE_CLICKHOUSE_SERVER)
        add_custom_target (clickhouse-server ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-server DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-server DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-server)
    endif ()
    if (ENABLE_CLICKHOUSE_CLIENT)
        add_custom_target (clickhouse-client ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-client DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-client DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-client)
    endif ()
    if (ENABLE_CLICKHOUSE_LOCAL)
        add_custom_target (clickhouse-local ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-local DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-local DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-local)
    endif ()
    if (ENABLE_CLICKHOUSE_BENCHMARK)
        add_custom_target (clickhouse-benchmark ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-benchmark DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-benchmark DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-benchmark)
    endif ()
    if (ENABLE_CLICKHOUSE_PERFORMANCE)
        add_custom_target (clickhouse-performance-test ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-performance-test DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-performance-test DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-performance-test)
    endif ()
    if (ENABLE_CLICKHOUSE_COPIER)
        add_custom_target (clickhouse-copier ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-copier DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-copier DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-copier)
    endif ()
    if (ENABLE_CLICKHOUSE_EXTRACT_FROM_CONFIG)
        add_custom_target (clickhouse-extract-from-config ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-extract-from-config DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-extract-from-config DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-extract-from-config)
    endif ()
    if (ENABLE_CLICKHOUSE_COMPRESSOR)
        add_custom_target (clickhouse-compressor ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-compressor DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-compressor DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-compressor)
    endif ()
    if (ENABLE_CLICKHOUSE_FORMAT)
        add_custom_target (clickhouse-format ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-format DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-format DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-format)
    endif ()
    if (ENABLE_CLICKHOUSE_COPIER)
        add_custom_target (clickhouse-obfuscator ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-obfuscator DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-obfuscator DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-obfuscator)
    endif ()
    if (ENABLE_CLICKHOUSE_ODBC_BRIDGE)
        add_custom_target (clickhouse-odbc-bridge ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-odbc-bridge DEPENDS clickhouse)
        install (FILES ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-odbc-bridge DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
        list(APPEND CLICKHOUSE_BUNDLE clickhouse-odbc-bridge)
    endif ()


    # install always because depian package want this files:
    add_custom_target (clickhouse-clang ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-clang DEPENDS clickhouse)
    add_custom_target (clickhouse-lld ALL COMMAND ${CMAKE_COMMAND} -E create_symlink clickhouse clickhouse-lld DEPENDS clickhouse)
    list(APPEND CLICKHOUSE_BUNDLE clickhouse-clang clickhouse-lld)

    install (TARGETS clickhouse RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)

    install (FILES
       ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-clang
       ${CMAKE_CURRENT_BINARY_DIR}/clickhouse-lld
       DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)

    add_custom_target (clickhouse-bundle ALL DEPENDS ${CLICKHOUSE_BUNDLE})

endif ()

if (TARGET clickhouse-server AND TARGET copy-headers)
    add_dependencies(clickhouse-server copy-headers)
endif ()
