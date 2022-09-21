#xcpp/src/xlog/CMakeLists.txt

#xlog  库编译

cmake_minimum_required(VERSION 3.22)

project(xlog)

aux_source_directory(${CMAKE_CURRENT_LIST_DIR} SRC)


#加入.h文件到编译依赖
FILE(GLOB H_FILE ${CMAKE_CURRENT_LIST_DIR}/*.h) 
FILE(GLOB H_FILE_I ${CMAKE_CURRENT_LIST_DIR}/include/*.h) 


add_library(${PROJECT_NAME} ${SRC} ${H_FILE} ${H_FILE_I})


#设置头文件查找路径
target_include_directories(${PROJECT_NAME} PUBLIC
   ${CMAKE_CURRENT_LIST_DIR}/include/
   ${CMAKE_CURRENT_LIST_DIR}
)


#配置C++参数
target_compile_features(${PROJECT_NAME} PRIVATE 
   cxx_std_14
)


#配置vs属性 bigobj 
if(msvc)
  set_target_properties(${PROJECT_NAME} PROPERTIES
      COMPILE_FLAGS "-bigobj"
  )
endif()

#静态库 动态库 不同宏变量
# XLOG_STATIC   XLOG_EXPORTS
target_compile_definitions( ${PROJECT_NAME} PUBLIC XLOG_STATIC) 


#默认为Debug
if(cmake_build_type STREQUAL "") 
   set(CMAKE_BUILD_TYPE Debug)
endif()

#配置输出路径 Debug  Release版本 输出路径一致
set(CONF_TYPES Debug Release  RelWithDebInfo MinSizeRel)
list(APPEND CONF_TYPES "")
foreach(type IN LISTS CONF_TYPES)
  set(conf "")  #设置尾值为空
  if(type)
      string(TOUPPER _${type} conf)
      message("conf = ${conf}")
   endif()

   set_target_properties(${PROJECT_NAME} PROPERTIES
      RUNTIME_OUTPUT_DIRECTORY${conf} ${CMAKE_CURRENT_LIST_DIR}/../../bin # dll exe 执行程序
      LIBRARY_OUTPUT_DIRECTORY${conf} ${CMAKE_CURRENT_LIST_DIR}/../../lib #  .so .dylib
      ARCHIVE_OUTPUT_DIRECTORY${conf} ${CMAKE_CURRENT_LIST_DIR}/../../lib # .lib .a
      PDB_OUTPUT_DIRECTORY${conf} ${CMAKE_CURRENT_LIST_DIR}/../../bin # pdb 
   )
endforeach()

#设置debug 添加d后缀
set_target_properties(${PROJECT_NAME} PROPERTIES
    DEBUG_POSTFIX "d"
)


