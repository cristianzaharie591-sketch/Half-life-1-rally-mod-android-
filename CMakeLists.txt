cmake_minimum_required(VERSION 3.12)
project(HLRally_Xash3D C CXX)

# Configurare pentru compilare pe 32-bit (standard pentru Xash3D legacy)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -m32")
endif()

# Definiții globale cerute de HLSDK și Xash3D pentru Linux
add_definitions(-DLINUX -D_LINUX -DXASH3D -D_POSIX_C_SOURCE=200809L)
set(CMAKE_CXX_STANDARD 11)

# Directoarele cu headere (Include paths)
include_directories(
    ${CMAKE_SOURCE_DIR}/common
    ${CMAKE_SOURCE_DIR}/pm_shared
    ${CMAKE_SOURCE_DIR}/engine
    ${CMAKE_SOURCE_DIR}/dlls
    ${CMAKE_SOURCE_DIR}/cl_dll
)

# 1. COMPILARE SERVER (hl.so)
file(GLOB_RECURSE SERVER_SRCS 
    "${CMAKE_SOURCE_DIR}/dlls/*.cpp"
    "${CMAKE_SOURCE_DIR}/game_shared/*.cpp"
)
add_library(hl SHARED ${SERVER_SRCS})
target_compile_options(hl PRIVATE -w) 

# 2. COMPILARE CLIENT (client.so)
file(GLOB_RECURSE CLIENT_SRCS 
    "${CMAKE_SOURCE_DIR}/cl_dll/*.cpp"
    "${CMAKE_SOURCE_DIR}/pm_shared/*.c"
    "${CMAKE_SOURCE_DIR}/pm_shared/*.cpp"
)
add_library(client SHARED ${CLIENT_SRCS})
target_compile_options(client PRIVATE -w)

# 3. CREAREA AUTOMATĂ A STRUCTURII DE FOLDERE SOLICITATE
set(STRUCT_DIR "${CMAKE_SOURCE_DIR}/LINUX .SO FILES AND FOLDERS")

add_custom_command(TARGET client POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/cldll"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/common"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/dll"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/engine"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/game_shared"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${STRUCT_DIR}/pm_shared"
    
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:client> "${STRUCT_DIR}/cldll/client.so"
)

add_custom_command(TARGET hl POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:hl> "${STRUCT_DIR}/dll/hl.so"
)
