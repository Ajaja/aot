declare_cmake_min_version()

if (BUILD_COM) 
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}")
    find_package(IDL REQUIRED)
endif()


if (BUILD_COM)
    message ("compile COM-objects for Microsoft Windows")
    add_subdirectory (Lemmatizer)
    add_subdirectory (TestLemmatizerCOM)
endif()

if(BUILD_WINDOWS_GUI)
	        
    if(${CMAKE_SIZEOF_VOID_P} EQUAL 4)
        message ("BUILD_WINDOWS_GUI=1")

        #must be compiled 32bit, since we have only 32-bit tcl80.lib
        add_subdirectory (RossDev)
            
        # obsolete mdi tabs cannot be comppiled under 64
        # add_subdirectory (Morphwizard)
    endif()

    add_subdirectory (VisualSynan)
    add_subdirectory (Demo)
    add_subdirectory (Morphwizard)
endif()

