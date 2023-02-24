message("    -Client")

project(Client)

set(CLIENT_NAME hgtClient)

file(GLOB CLIENT_SOURCES *.cpp *.h)

add_executable(${CLIENT_NAME}
    ${CLIENT_SOURCES}
)

#set(CLIENT_LIBRARIES
    #fmt::fmt-header-only
#)
if(WIN32)
    set(CLIENT_LIBRARIES ${CLIENT_LIBRARIES}
        Winmm
        Psapi)
elseif(UNIX)
    set(CLIENT_LIBRARIES ${CLIENT_LIBRARIES}
        pthread
        stdc++fs)
endif()


target_link_libraries (${CLIENT_NAME} PRIVATE ${CLIENT_LIBRARIES})

#set_target_properties(${CLIENT_NAME} PROPERTIES LINK_FLAGS ${CW_LINKER_OPTIONS})

if(WIN32)
    #include(pch)	
	# warning LNK4099: PDB 'grpc.pdb' was not found; linking object as if no debug info
	set_target_properties(${CLIENT_NAME} PROPERTIES LINK_FLAGS "${CW_LINKER_OPTIONS} /ignore:4099")
endif()
if(UNIX)
    strip_debuginfo(${CLIENT_NAME})
endif()

if(UNIX)
    message("is unix")
else()
    install(TARGETS ${CLIENT_NAME} DESTINATION ${CMAKE_SOURCE_DIR}/${OUTDIR_DEBUG}   CONFIGURATIONS Debug)
endif()