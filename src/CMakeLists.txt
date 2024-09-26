cmake_minimum_required(VERSION 3.15)

set(LUA_CORE_SRCS lapi.c
    lcode.c
    lctype.c
    ldebug.c
    ldo.c
    ldump.c
    lfunc.c
    lgc.c
    llex.c
    lmem.c
    lobject.c
    lopcodes.c
    lparser.c
    lstate.c
    lstring.c
    ltable.c
    ltm.c
    lundump.c
    lvm.c
    lzio.c)
set(LUA_LIB_SRCS lauxlib.c
    lbaselib.c
    lcorolib.c
    ldblib.c
    liolib.c
    lmathlib.c
    loadlib.c
    loslib.c
    lstrlib.c
    ltablib.c
    lutf8lib.c
    linit.c)

set(MATH_LINKING)

if(UNIX AND NOT APPLE)
    set(MATH_LINKING m)
endif()

set(LUA_INCLUDE_SUFFIX "lua${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")

add_library(lua_shared SHARED ${LUA_CORE_SRCS} ${LUA_LIB_SRCS})
target_include_directories(lua_shared PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> $<INSTALL_INTERFACE:./include/${LUA_INCLUDE_SUFFIX}>)
if(WIN32)
    target_compile_definitions(lua_shared PUBLIC LUA_BUILD_AS_DLL)
endif()
target_link_libraries(lua_shared PUBLIC ${MATH_LINKING})

if(UNIX AND NOT APPLE AND LUA_USES_POSIX)
    target_compile_definitions(lua_shared PUBLIC LUA_USE_POSIX)
endif()

add_library(lua_static STATIC ${LUA_CORE_SRCS} ${LUA_LIB_SRCS})
target_include_directories(lua_static PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> $<INSTALL_INTERFACE:./include>)
target_link_libraries(lua_static PUBLIC ${MATH_LINKING})

if(UNIX AND NOT APPLE AND LUA_USES_POSIX)
    target_compile_definitions(lua_static PUBLIC LUA_USE_POSIX)
endif()

add_executable(lua lua.c)
target_include_directories(lua PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)
target_link_libraries(lua PUBLIC lua_static)

add_executable(luac luac.c)
target_include_directories(luac PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)
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
        DESTINATION "include/${LUA_INCLUDE_SUFFIX}")
