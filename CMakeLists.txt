cmake_minimum_required(VERSION 3.9.6)
project(uWS C)

# enable C++ language
enable_language(C CXX)

set(COMMON_COMPILE_FLAGS -std=c11 -DLIBUS_USE_OPENSSL)
set(DEBUG_COMPILE_FLAGS ${COMMON_COMPILE_FLAGS} -ggdb3)

set(libsrc
        uSockets/src/bsd.c
	uSockets/src/context.c
	uSockets/src/loop.c
	uSockets/src/socket.c
	uSockets/src/crypto/openssl.c
	uSockets/src/crypto/openssl.c
	uSockets/src/crypto/sni_tree.cpp
	uSockets/src/eventing/epoll_kqueue.c
	uSockets/src/eventing/gcd.c
	uSockets/src/eventing/libuv.c
)

# This is the "object library" target: compiles the sources only once.
add_library(${PROJECT_NAME}ObjLib OBJECT ${libsrc})
target_include_directories(${PROJECT_NAME}ObjLib PUBLIC ${CMAKE_CURRENT_LIST_DIR}/uSockets/src)
target_compile_options(${PROJECT_NAME}ObjLib PRIVATE "$<$<CONFIG:DEBUG>:${DEBUG_COMPILE_FLAGS}>")
target_compile_options(${PROJECT_NAME}ObjLib PRIVATE "$<$<CONFIG:RELEASE>:${COMMON_COMPILE_FLAGS}>")
target_compile_options(${PROJECT_NAME}ObjLib PRIVATE "$<$<CONFIG:RELWITHDEBINFO>:${COMMON_COMPILE_FLAGS}>")

# Shared libraries need PIC.
# set_property(TARGET ${PROJECT_NAME}ObjLib PROPERTY POSITION_INDEPENDENT_CODE 1)

find_package(OpenSSL REQUIRED)
target_link_libraries(${PROJECT_NAME}ObjLib OpenSSL::SSL)

if(WIN32)
    target_link_libraries(${PROJECT_NAME}ObjLib uv_a)
    target_include_directories(${PROJECT_NAME}ObjLib PUBLIC ../libuv/include)
endif()

