cmake_minimum_required(VERSION 3.10)

project(mcachefs
    VERSION 0.6.0
    DESCRIPTION "Fuse caching mount point"
    LANGUAGES C
)

INCLUDE(CheckIncludeFiles)

IF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
	SET(CMAKE_BUILD_TYPE RelWithDebInfo)
ENDIF(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FILE_OFFSET_BITS=64")

add_executable(mcachefs
    crc64table.h
    list.h
    mcachefs.c
    mcachefs-cleanup-backing.c
    mcachefs-config.c
    mcachefs-config.h
    mcachefs-file.c
    mcachefs-file.h
    mcachefs-file-ts.c
    mcachefs.h
    mcachefs-hash.c
    mcachefs-hash.h
    mcachefs-io.c
    mcachefs-io.h
    mcachefs-journal.c
    mcachefs-journal.h
    mcachefs-log.h
    mcachefs-lowlevel.c
    mcachefs-metadata.c
    mcachefs-metadata.h
    mcachefs-mutex.c
    mcachefs-mutex.h
    mcachefs-transfer.c
    mcachefs-transfer.h
    mcachefs-types.h
    mcachefs-util.c
    mcachefs-util.h
    mcachefs-vops.c
    mcachefs-vops.h
)

option(WITH_XATTR "Enable support for extended attributes" ON)

IF (WITH_XATTR)
	CHECK_INCLUDE_FILES("sys/xattr.h" HAVE_LIBC_XATTR)
	CHECK_INCLUDE_FILES("attr/xattr.h" HAVE_LIBATTR_XATTR)

	IF (HAVE_LIBC_XATTR)
		add_definitions(-DLIBC_XATTR)
	ELSEIF(HAVE_LIBATTR_XATTR)
		add_definitions(-DLIBATTR_XATTR)
	ENDIF()

	IF (NOT HAVE_LIBC_XATTR AND NOT HAVE_LIBATTR_XATTR)
		add_definitions(-DDISABLE_XATTR)
	ENDIF()
ELSE (WITH_XATTR)
	add_definitions(-DDISABLE_XATTR)
ENDIF (WITH_XATTR)

find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBFUSE3 IMPORTED_TARGET fuse3)
IF(${LIBFUSE3_FOUND})
    execute_process(COMMAND bash "-c" "echo -n ${LIBFUSE3_VERSION}|cut -d '.' -f 1,2 |sed s/'\\.'//g|tr -d '\n'" OUTPUT_VARIABLE LIBFUSE_VERSION2)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DFUSE_USE_VERSION=${LIBFUSE_VERSION2}")
    target_link_libraries(siafs PUBLIC ${LIBFUSE3_LIBRARIES})
ENDIF(${LIBFUSE3_FOUND})

install(FILES ../extra-scripts/mount.mcachefs DESTINATION sbin)
install(TARGETS mcachefs RUNTIME DESTINATION bin)
