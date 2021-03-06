# ft-index only supports x86-64 and cmake-2.8.9+
IF(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" AND
    NOT CMAKE_VERSION VERSION_LESS "2.8.9" AND
   NOT WITHOUT_TOKUDB AND NOT WITHOUT_TOKUDB_STORAGE_ENGINE)
CHECK_CXX_SOURCE_COMPILES(
"
struct a {int b; int c; };
struct a d = { .b=1, .c=2 };
int main() { return 0; }
" TOKUDB_OK)
ENDIF()

IF(NOT TOKUDB_OK)
  RETURN()
ENDIF()

IF(DEFINED ENV{TOKUDB_VERSION})
    SET(TOKUDB_VERSION $ENV{TOKUDB_VERSION})
    ADD_DEFINITIONS("-DTOKUDB_VERSION=\"${TOKUDB_VERSION}\"")
ENDIF()

IF(DEFINED ENV{TOKUDB_PATCHES})
    SET(TOKUDB_PATCHES $ENV{TOKUDB_PATCHES})
    ADD_DEFINITIONS("-DTOKUDB_PATCHES=${TOKUDB_PATCHES}")
ENDIF()

ADD_SUBDIRECTORY(ft-index)

# TODO: clean up includes in ft-index
INCLUDE_DIRECTORIES(ft-index)
INCLUDE_DIRECTORIES(ft-index/include)
INCLUDE_DIRECTORIES(ft-index/portability)
INCLUDE_DIRECTORIES(ft-index/toku_include)
INCLUDE_DIRECTORIES(ft-index/util)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}/ft-index)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}/ft-index/buildheader)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}/ft-index/toku_include)

SET(TOKUDB_PLUGIN_DYNAMIC "ha_tokudb")
SET(TOKUDB_SOURCES ha_tokudb.cc)
MYSQL_ADD_PLUGIN(tokudb ${TOKUDB_SOURCES} STORAGE_ENGINE MODULE_ONLY
    LINK_LIBRARIES tokufractaltree_static tokuportability_static ${ZLIB_LIBRARY} stdc++)
SET_PROPERTY(TARGET tokudb APPEND PROPERTY LINK_FLAGS_RELEASE "-flto -fuse-linker-plugin")
SET_PROPERTY(TARGET tokudb APPEND PROPERTY LINK_FLAGS_RELWITHDEBINFO "-flto -fuse-linker-plugin")
