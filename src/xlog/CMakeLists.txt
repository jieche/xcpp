#xcpp/src/xlog/CMakeLists.txt

#xlog  库编译

cmake_minimum_required(VERSION 3.22)

project(xlog)

#生成c++代码  传递命名空间
# cmakedefine X_XCPP_NAMESPACE_ @_XCPP_NAMESPACE_@

set(_XCPP_NAMESPACE_ "namespace xlog {")
configure_file(xconfig.h.in ${CMAKE_CURRENT_LIST_DIR}/include/xconfig.h)


#用户指定是动态库 静态库
option(XLOG_SHARED "OFF is static" ON)
set(TYPE_STATIC)
if(XLOG_SHARED)
   set(TYPE SHARED)
endif()

# #静态库 动态库 不同宏变量 ---对象没生成 不能放在这
# # XLOG_STATIC   XLOG_EXPORTS
# if(XLOG_SHARED)
#    target_compile_definitions( ${PROJECT_NAME} PUBLIC XLOG_EXPORTS) 
# else()
#    target_compile_definitions( ${PROJECT_NAME} PUBLIC XLOG_STATIC) 
# endif()


#CMAKE_CURRENT_LIST_DIR 当前CMakelists.txt 文件所在路径
aux_source_directory(${CMAKE_CURRENT_LIST_DIR} SRC)


#加入.h文件到编译依赖
FILE(GLOB H_FILE ${CMAKE_CURRENT_LIST_DIR}/*.h) 
FILE(GLOB H_FILE_I ${CMAKE_CURRENT_LIST_DIR}/include/*.h) 


add_library(${PROJECT_NAME} ${TYPE}  ${SRC} ${H_FILE} ${H_FILE_I})

#静态库 动态库 不同宏变量
# XLOG_STATIC   XLOG_EXPORTS
if(XLOG_SHARED)
   target_compile_definitions( ${PROJECT_NAME} PUBLIC XLOG_EXPORTS) 
else()
   target_compile_definitions( ${PROJECT_NAME} PUBLIC XLOG_STATIC) 
endif()


#设置头文件查找路径
target_include_directories(${PROJECT_NAME} PRIVATE
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

#设置安装头文件
set_target_properties(${PROJECT_NAME} PROPERTIES
PUBLIC_HEADER "${H_FILE_I}"
)

#安装库
set(version 1.0)
#cmake --install build --prefix=out
install(TARGETS ${PROJECT_NAME}
      EXPORT ${PROJECT_NAME}
      RUNTIME DESTINATION bin
      LIBRARY DESTINATION lib
      PUBLIC_HEADER DESTINATION include
)

#支持 find_package


#配置文件 config
instaLl(EXPORT ${PROJECT_NAME} FILE ${PROJECT_NAME}Config.cmake
    DESTINATION lib/config/${PROJECT_NAME}-${version}
)

#版本文件
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
${CMAKE_CURRENT_LIST_DIR}/../../lib/conf/${PROJECT_NAME}-${version}/${PROJECT_NAME}ConfigVersion.cmake
VERSION ${version}
COMPATIBILITY SameMajorVersion  # 版本兼容问题
)

install(FILES ${CMAKE_CURRENT_LIST_DIR}/../../lib/conf/${PROJECT_NAME}-${version}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION lib/config/${PROJECT_NAME}-${version}
)

