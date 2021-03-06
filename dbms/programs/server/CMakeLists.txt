add_library (clickhouse-server-lib ${LINK_MODE}
    HTTPHandler.cpp
    InterserverIOHTTPHandler.cpp
    MetricsTransmitter.cpp
    NotFoundHandler.cpp
    PingRequestHandler.cpp
    ReplicasStatusHandler.cpp
    RootRequestHandler.cpp
    Server.cpp
    TCPHandler.cpp
    )

target_link_libraries (clickhouse-server-lib PRIVATE clickhouse_common_io daemon clickhouse_storages_system clickhouse_functions clickhouse_aggregate_functions clickhouse_table_functions ${Poco_Net_LIBRARY})
if (USE_POCO_NETSSL)
    target_link_libraries (clickhouse-server-lib PRIVATE ${Poco_NetSSL_LIBRARY} ${Poco_Crypto_LIBRARY})
endif ()

target_include_directories (clickhouse-server-lib PUBLIC ${ClickHouse_SOURCE_DIR}/libs/libdaemon/include)

if (CLICKHOUSE_SPLIT_BINARY)
    add_executable (clickhouse-server clickhouse-server.cpp)
    target_link_libraries (clickhouse-server PRIVATE clickhouse-server-lib)
    install (TARGETS clickhouse-server ${CLICKHOUSE_ALL_TARGETS} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT clickhouse)
endif ()

if (GLIBC_COMPATIBILITY)
    set (GLIBC_MAX_REQUIRED 2.4 CACHE INTERNAL "")
    # temporary disabled. to enable - change 'exit 0' to 'exit $a'
    add_test(NAME GLIBC_required_version COMMAND bash -c "readelf -s ${CMAKE_CURRENT_BINARY_DIR}/../clickhouse-server | perl -nE 'END {exit 0 if $a} ++$a, print if /\\x40GLIBC_(\\S+)/ and pack(q{C*}, split /\\./, \$1) gt pack q{C*}, split /\\./, q{${GLIBC_MAX_REQUIRED}}'")

    #add_test(NAME GLIBC_required_version COMMAND bash -c "readelf -s ${CMAKE_CURRENT_BINARY_DIR}/../clickhouse-server | grep '@GLIBC' | grep -oP 'GLIBC_[\\d\\.]+' | sort | uniq | sort --version-sort --reverse | perl -lnE 'warn($_), exit 1 if $_ gt q{GLIBC_${GLIBC_MAX_REQUIRED}}'") # old
endif ()

install (
    FILES config.xml users.xml
    DESTINATION  ${CLICKHOUSE_ETC_DIR}/clickhouse-server
    COMPONENT clickhouse)
