cmake_minimum_required(VERSION 3.15)

project(Lua LANGUAGES CXX VERSION 5.4.7)

option(LUA_USES_POSIX "Use posix as standard" ON)

add_subdirectory(src)
