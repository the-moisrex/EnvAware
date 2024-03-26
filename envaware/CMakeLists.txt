set(HEADERS
        ${LIB_INCLUDE_DIR}/version.hpp
)

set(SOURCES
#        ${LIB_INCLUDE_DIR}/version.hpp
)

include(../cmake/common.cmake)

if (VERBOSE)
    set(HEADERS_SHORT "")
    foreach (source ${HEADERS})
        string(REPLACE "${PROJECT_SOURCE_DIR}/" "" SOURCE_SHORT ${source})
        list(APPEND HEADERS_SHORT ${SOURCE_SHORT})
    endforeach ()
    string(REPLACE ";" "\n                                    " HEADERS_SHORT_P "${HEADERS_SHORT}")
    message(STATUS "Header Files                   : ${HEADERS_SHORT_P}")
endif () # verbose

add_library(${LIB_NAME} STATIC)
add_library("${PROJECT_NAME}::${LIB_NAME}" ALIAS ${LIB_NAME})
target_sources(${LIB_NAME} PUBLIC
        FILE_SET HEADERS
        FILES ${HEADERS})

target_sources(${LIB_NAME} PRIVATE ${SOURCES})

target_link_libraries(${LIB_NAME}
#        PUBLIC fmt::fmt
)

set_target_properties(${LIB_NAME} PROPERTIES
        VERSION ${LIB_VERSION}
        SOVERSION ${LIB_SOVERSION}
        LINKER_LANGUAGE CXX
)


install(TARGETS ${LIB_NAME}
        EXPORT "${LIB_NAME}Config"
        FILE_SET HEADERS
        DESTINATION include/envaware
)
export(TARGETS ${LIB_NAME}
        NAMESPACE ${PROJECT_NAME}::${LIB_NAME}
        FILE "${CMAKE_CURRENT_BINARY_DIR}/${LIB_NAME}InternalConfig.cmake"
)
install(EXPORT "${LIB_NAME}Config"
        FILE "${LIB_NAME}Config.cmake"
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION "${CMAKE_INSTALL_DATADIR}/${LIB_NAME}"
)

setup_library(${LIB_NAME})
