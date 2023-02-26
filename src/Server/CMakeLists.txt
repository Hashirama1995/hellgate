message("    -Server")

project(Server)

set(SERVER_NAME hgtServer)

file(GLOB SERVER_SOURCES *.cpp *.h)

add_executable(${SERVER_NAME}
    ${SERVER_SOURCES}
)

set(SERVER_LIBRARIES
    fmt::fmt-header-only
    protobuf::libprotobuf
    gRPC::grpc++
)
if(WIN32)
    set(SERVER_LIBRARIES ${SERVER_LIBRARIES}
        Winmm
        Psapi)
elseif(UNIX)
    set(SERVER_LIBRARIES ${SERVER_LIBRARIES}
        pthread
        stdc++fs)
endif()


target_link_libraries (${SERVER_NAME} PRIVATE ${SERVER_LIBRARIES})

#set_target_properties(${SERVER_NAME} PROPERTIES LINK_FLAGS ${CW_LINKER_OPTIONS})

if(WIN32)
    #include(pch)	
	# warning LNK4099: PDB 'grpc.pdb' was not found; linking object as if no debug info
	set_target_properties(${SERVER_NAME} PROPERTIES LINK_FLAGS "${CW_LINKER_OPTIONS} /ignore:4099")
endif()
if(UNIX)
    strip_debuginfo(${SERVER_NAME})
endif()

if(UNIX)
    message("is unix")
else()
    install(TARGETS ${SERVER_NAME} DESTINATION ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG}   CONFIGURATIONS Debug)
endif()