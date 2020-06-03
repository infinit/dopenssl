cmake_minimum_required(VERSION 3.10)

cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

project(dopenssl DESCRIPTION "A deterministic implementation of some of the OpenSSL functionalities")

enable_language(C)
enable_testing()

option(TEST "Enable tests" OFF)

# Must use GNUInstallDirs to install libraries into correct locations on all platforms.
include(GNUInstallDirs)

set(SRC_DIR "src/dopenssl")

set(HEADER_FILES
	${SRC_DIR}/all.h
	${SRC_DIR}/bn.h
	${SRC_DIR}/rand.h
	${SRC_DIR}/rsa.h
)

include_directories(${SRC_DIR}/..)

set(SRC_FILES
	${SRC_DIR}/bn.c
	${SRC_DIR}/rand.c
	${SRC_DIR}/rsa.c
)

add_library(dopenssl STATIC ${SRC_FILES})

find_package(OpenSSL)
if (OPENSSL_FOUND)
	include_directories(${OPENSSL_INCLUDE_DIRS})
	target_link_libraries (dopenssl ${OPENSSL_LIBRARIES})
endif (OPENSSL_FOUND)

set_target_properties(dopenssl PROPERTIES OUTPUT_NAME "dopenssl")
target_include_directories(dopenssl PUBLIC
	${SRC_DIR}
)

install(FILES ${HEADER_FILES} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/dopenssl)

# 'make install' to the correct locations (provided by GNUInstallDirs).
install(TARGETS dopenssl
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

if(TEST)
	set(TEST_DIR "tests/dopenssl")
	add_executable(bn ${TEST_DIR}/bn.c)
	add_executable(rand ${TEST_DIR}/rand.c)
	add_executable(rsa ${TEST_DIR}/rsa.c)

	target_include_directories(bn PRIVATE ${TEST_DIR})
	target_include_directories(rand PRIVATE ${TEST_DIR})
	target_include_directories(rsa PRIVATE ${TEST_DIR})

	target_link_libraries(bn dopenssl crypto)
	target_link_libraries(rand dopenssl crypto)
	target_link_libraries(rsa dopenssl crypto)
endif()
