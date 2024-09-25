cmake_minimum_required(VERSION 3.15)

project(Lua LANGUAGES C VERSION 5.4.7)

option(LUA_USES_POSIX "Use posix as standard" ON)

set(LUA_CORE_SRCS src/lapi.c
    src/lcode.c
    src/lctype.c
    src/ldebug.c
    src/ldo.c
    src/ldump.c
    src/lfunc.c
    src/lgc.c
    src/llex.c
    src/lmem.c
    src/lobject.c
    src/lopcodes.c
    src/lparser.c
    src/lstate.c
    src/lstring.c
    src/ltable.c
    src/ltm.c
    src/lundump.c
    src/lvm.c
    src/lzio.c)
set(LUA_LIB_SRCS src/lauxlib.c
    src/lbaselib.c
    src/lcorolib.c
    src/ldblib.c
    src/liolib.c
    src/lmathlib.c
    src/loadlib.c
    src/loslib.c
    src/lstrlib.c
    src/ltablib.c
    src/lutf8lib.c
    src/linit.c)

set(MATH_LINKING)

if(UNIX AND NOT APPLE)
    set(MATH_LINKING m)
endif()

add_library(lua_shared SHARED ${LUA_CORE_SRCS} ${LUA_LIB_SRCS})
target_include_directories(lua_shared PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src> $<INSTALL_INTERFACE:./include>)
if(WIN32)
    target_compile_definitions(lua_shared PUBLIC LUA_BUILD_AS_DLL)
endif()
target_link_libraries(lua_shared PUBLIC ${MATH_LINKING})

if(UNIX AND NOT APPLE AND LUA_USES_POSIX)
    target_compile_definitions(lua_shared PUBLIC LUA_USE_POSIX)
endif()

add_library(lua_static STATIC ${LUA_CORE_SRCS} ${LUA_LIB_SRCS})
target_include_directories(lua_static PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src> $<INSTALL_INTERFACE:./include>)
target_link_libraries(lua_static PUBLIC ${MATH_LINKING})

if(UNIX AND NOT APPLE AND LUA_USES_POSIX)
    target_compile_definitions(lua_static PUBLIC LUA_USE_POSIX)
endif()

add_executable(lua src/lua.c)
target_include_directories(lua PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>)
target_link_libraries(lua PUBLIC lua_static)

add_executable(luac src/luac.c)
target_include_directories(luac PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>)
target_link_libraries(luac PUBLIC lua_static)

set(LUA_GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(LUA_VERSION_CONFIG "${LUA_GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(LUA_PROJECT_CONFIG "${LUA_GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
set(LUA_TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(LUA_CONFIG_INSTALL_DIR "lib/cmake/${PROJECT_NAME}")
set(LUA_NAMESPACE "Lua::")
set(LUA_VERSION ${PROJECT_VERSION})

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${LUA_VERSION_CONFIG}" VERSION ${LUA_VERSION} COMPATIBILITY SameMajorVersion
)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.in" "${LUA_PROJECT_CONFIG}" @ONLY)
# Install cmake config files
install(FILES "${LUA_PROJECT_CONFIG}" "${LUA_VERSION_CONFIG}"
        DESTINATION "${LUA_CONFIG_INSTALL_DIR}")
install(
    EXPORT "${LUA_TARGETS_EXPORT_NAME}"
    NAMESPACE "${LUA_NAMESPACE}"
    DESTINATION "${LUA_CONFIG_INSTALL_DIR}")

install(TARGETS lua_shared lua_static lua luac
        EXPORT ${LUA_TARGETS_EXPORT_NAME} 
		RUNTIME DESTINATION "bin"
        ARCHIVE DESTINATION "lib"
        LIBRARY DESTINATION "lib")

install(FILES "${PROJECT_SOURCE_DIR}/src/lua.h" 
        "${PROJECT_SOURCE_DIR}/src/luaconf.h" 
        "${PROJECT_SOURCE_DIR}/src/lualib.h" 
        "${PROJECT_SOURCE_DIR}/src/lauxlib.h" 
        "${PROJECT_SOURCE_DIR}/src/lua.hpp" 
        DESTINATION "include")
