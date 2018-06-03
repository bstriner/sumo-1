set(ENABLE_PYTHON_BINDINGS ON CACHE BOOL "Build Python Bindings")
set(ENABLE_JAVA_BINDINGS ON CACHE BOOL "Build Java Bindings")

if(SWIG_FOUND)
    include(${SWIG_USE_FILE})
    set(CMAKE_SWIG_FLAGS "")
    set_source_files_properties(libsumo.i PROPERTIES CPLUSPLUS ON)

    if(ENABLE_PYTHON_BINDINGS)
        set(SETUP_PY_IN "${CMAKE_CURRENT_SOURCE_DIR}/setup.py.in")
        set(SETUP_PY_TMP "${CMAKE_CURRENT_BINARY_DIR}/setup.py.tmp")
        set(SETUP_PY "${CMAKE_CURRENT_BINARY_DIR}/setup.py")
        set(PYLIBSUMO_TIMESTAMP "${CMAKE_CURRENT_BINARY_DIR}/build/timestamp")
        GET_DIRECTORY_PROPERTY(PYLIBSUMO_INCLUDES INCLUDE_DIRECTORIES)

        SET(PYSUMO_LIBS "")
        FOREACH(PYSUMO_LIB ${sumolibs})
            MESSAGE(STATUS "Target: ${PYSUMO_LIB}, $<TARGET_FILE:${PYSUMO_LIB}>")
            IF($<TARGET_FILE:${PYSUMO_LIB}>)
                SET(PYSUMO_LIBS "'$<TARGET_FILE:${PYSUMO_LIB}>', ${PYSUMO_LIBS}")
            ELSE()
                SET(PYSUMO_LIBS "'${PYSUMO_LIB}', ${PYSUMO_LIBS}")
            ENDIF()
        ENDFOREACH()

        configure_file(${SETUP_PY_IN} ${SETUP_PY_TMP})
        file(GENERATE OUTPUT ${SETUP_PY} INPUT ${SETUP_PY_TMP})

        add_custom_command(OUTPUT ${PYLIBSUMO_TIMESTAMP}
                           WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                           COMMAND ${PYTHON_EXECUTABLE} ${SETUP_PY} build
                           COMMAND ${CMAKE_COMMAND} -E touch ${PYLIBSUMO_TIMESTAMP}
                           DEPENDS ${sumolibs})
        add_custom_target(pylibsumo ALL DEPENDS ${PYLIBSUMO_TIMESTAMP})

        add_custom_target(install_pylibsumo
                          WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                          COMMAND ${PYTHON_EXECUTABLE} ${SETUP_PY} install
                          DEPENDS pylibsumo)
        install(
            CODE "execute_process(COMMAND ${PYTHON_EXECUTABLE} ${SETUP_PY} install
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})"
            COMPONENT pylibsumo)
    endif()

    if(ENABLE_JAVA_BINDINGS)
        find_package(Java)
        find_package(JNI)
        if(JAVA_FOUND AND JNI_FOUND)
            include(UseJava)
            include_directories(${JNI_INCLUDE_DIRS})
            set(CMAKE_SWIG_FLAGS -package de.dlr.ts.routingkernel.sumowrapper)
            set(CMAKE_SWIG_OUTDIR ${CMAKE_CURRENT_BINARY_DIR}/java)
            if (${CMAKE_VERSION} VERSION_GREATER_EQUAL 3.8.0)
                SWIG_ADD_LIBRARY(libsumo LANGUAGE java SOURCES libsumo.i)
            else()
                SWIG_ADD_MODULE(libsumo java libsumo.i)
            endif()
            if (MSVC)
                swig_link_libraries(libsumo ${sumolibs})
            else()
                set_source_files_properties(${swig_generated_file_fullname} PROPERTIES COMPILE_FLAGS "-Wno-strict-aliasing")
                swig_link_libraries(libsumo -Wl,--whole-archive ${sumolibs} -Wl,--no-whole-archive)
            endif()
    #        add_jar(libsumojni ${swig_extra_generated_files})
        else()
            message(WARNING "ENABLE_JAVA_BINDINGS is set but Java or JNI was not found.")
        endif()
    endif()
endif()

set(libsumostatic_STAT_SRCS
        Helper.cpp
        Helper.h
        InductionLoop.cpp
        InductionLoop.h
        Junction.cpp
        Junction.h
        Lane.cpp
        Lane.h
        LaneArea.cpp
        LaneArea.h
        MultiEntryExit.h
        MultiEntryExit.cpp
        Person.cpp
        Person.h
        POI.cpp
        POI.h
        Polygon.h
        Polygon.cpp
        Route.h
        Route.cpp
        Simulation.cpp
        Simulation.h
        Subscription.h
        TraCIDefs.h
        TrafficLight.cpp
        TrafficLight.h
        Vehicle.h
        Vehicle.cpp
        VehicleType.cpp
        VehicleType.h
        Edge.cpp
        Edge.h)

add_library(libsumostatic STATIC ${libsumostatic_STAT_SRCS})
set_property(TARGET libsumostatic PROPERTY PROJECT_LABEL "z_libsumostatic")
