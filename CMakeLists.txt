cmake_minimum_required(VERSION 2.8)
project(boost CXX)

option (boost_cmake_debug "Enable debugging of the cmake configuration" OFF)

set(SUPPORTED_LIBRARIES
    atomic
    chrono
    # context => requres ASM
    date_time
    exception
    filesystem
    graph
    # graph_parallel => not checked
    # iostreames => warnings
    # locale => warnings system detection needed
    # math => not simple need to investigate
    # mpi => not checked
    program_options
    # python => not checked
    random
    # regex => warnings
    # serialization => lots of warnings
    signals
    smart_ptr
    system
    # test => some config needed
    thread
    timer
    # wave => not checked    
)

set(BOOST_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/boost_1_53_0")
set(BOOST_LIBS_ROOT "${BOOST_ROOT}/libs")

# include top level boost libraries directories
include_directories (${BOOST_ROOT})

# loop through the supported libraries
foreach (LibraryName ${SUPPORTED_LIBRARIES})

    # configure the target
    set (LibraryTarget      "boost_${LibraryName}")
    set (LibrarySourceDir   "${BOOST_LIBS_ROOT}/${LibraryName}/src")

    # collect up all of the source files into a seperate library
    file (GLOB LibraryFiles "${LibrarySourceDir}/*.cpp")

    # special case for date and time library
    if (LibraryName STREQUAL "date_time" )
        file (GLOB PosixTimeFiles "${LibrarySourceDir}/posix_time/*.cpp")
        file (GLOB GregorianFiles "${LibrarySourceDir}/gregorian/*.cpp")
        list (APPEND LibraryFiles ${PosixTimeFiles} ${GregorianFiles})
    endif ()

    # special case for the thread library
    if (LibraryName STREQUAL "thread")
        if (WIN32)
            set (LibraryFiles
                "${LibrarySourceDir}/future.cpp"
                "${LibrarySourceDir}/win32/thread.cpp"
            )
        elseif (UNIX)
            file (GLOB UnixFiles "${LibrarySourceDir}/pthread/*.cpp")
            list (APPEND LibraryFiles ${UnixFiles})
        endif ()
    endif ()

    if (boost_cmake_debug)
        message (STATUS "Library Name  : ${LibraryName}")
        message (STATUS "Library Target: ${LibraryTarget}")
        foreach (File ${LibraryFiles})
            message (STATUS "  -: ${File}")
        endforeach (File)
        message (STATUS "")
    endif (boost_cmake_debug)

    # add the library
    add_library (${LibraryTarget} ${LibraryFiles})

endforeach (LibraryName)