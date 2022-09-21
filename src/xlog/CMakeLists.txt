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
