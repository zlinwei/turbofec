cmake_minimum_required(VERSION 3.16)
project(turbofec)

enable_language(C CXX)
set(CMAKE_CXX_STANDARD 20)

include_directories(include)
aux_source_directory(src SRCS)

option(WITH_SSE "Use SSE" ON)
#foreach (key
#        IN ITEMS
#        NUMBER_OF_LOGICAL_CORES
#        NUMBER_OF_PHYSICAL_CORES
#        TOTAL_VIRTUAL_MEMORY
#        AVAILABLE_VIRTUAL_MEMORY
#        TOTAL_PHYSICAL_MEMORY
#        AVAILABLE_PHYSICAL_MEMORY
#        IS_64BIT
#        HAS_FPU
#        HAS_MMX
#        HAS_MMX_PLUS
#        HAS_SSE
#        HAS_SSE2
#        HAS_SSE_FP
#        HAS_SSE_MMX
#        HAS_AMD_3DNOW
#        HAS_AMD_3DNOW_PLUS
#        HAS_IA64
#        OS_NAME
#        OS_RELEASE
#        OS_VERSION
#        OS_PLATFORM
#        )
#    cmake_host_system_information(RESULT _${key} QUERY ${key})
#endforeach ()

configure_file(src/config.h.in src/config.h @ONLY)
if (${WITH_SSE})
    message(STATUS "USE SSE4.1")
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -msse4.1)
    set(CMAKE_C_FLAGS ${CMAKE_CXX_FLAGS} -msse4.1)
endif ()


add_subdirectory(include)

add_library(${CMAKE_PROJECT_NAME} SHARED ${SRCS})
target_compile_options(${CMAKE_PROJECT_NAME} INTERFACE -mmmx -msse -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 -msha -maes -mavx -mfma -mavx2 -mavx512f -mavx512cd -mavx512vl -mavx512bw -mavx512dq -mavx512ifma -mavx512vbmi)

if (${WITH_SSE})
    target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE HAVE_SSE3)
endif ()

target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>"
        "$<INSTALL_INTERFACE:include>"
        )

# install lib
set_property(TARGET ${CMAKE_PROJECT_NAME} PROPERTY DEBUG_POSTFIX d)
set_property(TARGET ${CMAKE_PROJECT_NAME}
        PROPERTY INTERFACE_INCLUDE_DIRECTORIES
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

install(TARGETS ${CMAKE_PROJECT_NAME}
        EXPORT ${CMAKE_PROJECT_NAME}-targets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include
        )

install(EXPORT ${CMAKE_PROJECT_NAME}-targets
        NAMESPACE turbofec::
        FILE turbofec-config.cmake
        DESTINATION lib/cmake/${CMAKE_PROJECT_NAME})



