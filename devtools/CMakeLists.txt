cmake_minimum_required(VERSION 3.2)
project(make-scumm-fontdata)
project(md5table)
project(convbdf)

add_executable(make-scumm-fontdata make-scumm-fontdata.cpp )
add_executable(md5table md5table.cpp)
add_executable(convbdf convbdf.cpp)