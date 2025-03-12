if(BUILD_UNIT_TESTS)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUNIT_TEST")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DUNIT_TEST")
elseif(NOT DEBUG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNDEBUG")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG")
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${CMAKE_SOURCE_DIR}/src)

add_library(OBJ_LIB OBJECT ${SOURCES})

target_link_libraries(OBJ_LIB PUBLIC ${DYNAMIC_LIB})
set_property(TARGET ${OBJ_LIB} PROPERTY POSITION_INDEPENDENT_CODE 1)
if (BUILD_SHARED_LIBS)
add_library(braft-shared SHARED $<TARGET_OBJECTS:OBJ_LIB>)
else()
add_library(braft-static STATIC $<TARGET_OBJECTS:OBJ_LIB>)
endif()

if (BUILD_SHARED_LIBS)
target_include_directories(braft-shared PUBLIC $<INSTALL_INTERFACE:include>)
target_link_libraries(braft-shared PUBLIC ${DYNAMIC_LIB})
else()
target_include_directories(braft-static PUBLIC $<INSTALL_INTERFACE:include>)
target_link_libraries(braft-static PUBLIC ${DYNAMIC_LIB})
endif()

if (NOT BUILD_SHARED_LIBS)
SET_TARGET_PROPERTIES(braft-static PROPERTIES OUTPUT_NAME braft CLEAN_DIRECT_OUTPUT 1)
else()
SET_TARGET_PROPERTIES(braft-shared PROPERTIES OUTPUT_NAME braft CLEAN_DIRECT_OUTPUT 1)
endif()

if (NOT BUILD_SHARED_LIBS)
install(TARGETS braft-static EXPORT unofficial-braftTargets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib${LIBSUFFIX}
        ARCHIVE DESTINATION lib${LIBSUFFIX}
        )
else()
install(TARGETS braft-shared EXPORT unofficial-braftTargets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib${LIBSUFFIX}
        ARCHIVE DESTINATION lib${LIBSUFFIX}
        )
endif()

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/unofficial-braft-config.cmake"
[[include(CMakeFindDependencyMacro)
find_dependency(ZLIB)
find_dependency(gflags CONFIG)
find_dependency(glog CONFIG)
find_dependency(Protobuf CONFIG)
file(GLOB TARGET_FILES "${CMAKE_CURRENT_LIST_DIR}/unofficial-braftTargets.cmake")
foreach (TARGET_FILE ${TARGET_FILES})
    include("${TARGET_FILE}")
endforeach()
]])
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/unofficial-braft-config.cmake" DESTINATION share/unofficial-braft)

install(EXPORT unofficial-braftTargets
    NAMESPACE unofficial::braft::
    DESTINATION share/unofficial-braft
)
