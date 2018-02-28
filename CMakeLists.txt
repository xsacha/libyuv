cmake_minimum_required(VERSION 3.0)

project(libyuv VERSION 1703.0)

SET ( ly_base_dir	${CMAKE_CURRENT_LIST_DIR} )
SET ( ly_src_dir	${ly_base_dir}/source )
SET ( ly_inc_dir	${ly_base_dir}/include )
SET ( ly_lib_name	yuv )

FILE ( GLOB_RECURSE	ly_source_files ${ly_src_dir}/*.cc )
LIST ( SORT			ly_source_files )

include_directories( BEFORE ${ly_inc_dir} )

# this creates the library
ADD_LIBRARY				( ${ly_lib_name} ${ly_source_files} )

option(BUILD_EXAMPLES "Build console apps" OFF)
option(LIBYUV_USE_JPEG "Use libjpeg for example yuv conversions" OFF)
if(BUILD_EXAMPLES)
  # this creates the conversion tool
  ADD_EXECUTABLE        ( yuvconvert ${ly_base_dir}/util/yuvconvert.cc )
  TARGET_LINK_LIBRARIES	( yuvconvert ${ly_lib_static} )
  
  if(LIBYUV_USE_JPEG)
    include(FindJPEG)
    if (JPEG_FOUND)
      include_directories(${JPEG_INCLUDE_DIR})
      target_link_libraries(yuvconvert ${JPEG_LIBRARY})
      add_definitions(-DHAVE_JPEG)
    endif()
  endif()
endif()

################## install #######################
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${version_config}" COMPATIBILITY SameMajorVersion)

# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS "${ly_lib_name}"
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(FILES ${ly_header_files} DESTINATION "${include_install_dir}/${PROJECT_NAME}")
install(FILES ${ly_inc_dir}/libyuv.h DESTINATION include/)
install(FILES "${project_config}" "${version_config}" DESTINATION "${config_install_dir}")
install(EXPORT "${targets_export_name}" NAMESPACE "${namespace}" DESTINATION "${config_install_dir}")
