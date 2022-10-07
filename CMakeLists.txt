cmake_minimum_required(VERSION 3.5)

project(nlohmann_json_schema_validator_vendor)

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_libraries REQUIRED)

macro(build_nlohmann_json_schema_validator)

  set(cmake_commands)
  set(cmake_configure_args -Wno-dev)

  if(WIN32)
    if(DEFINED CMAKE_GENERATOR)
      list(APPEND cmake_configure_args -G ${CMAKE_GENERATOR})
    endif()
    if("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "^(x86_|x86-|AMD|amd|x)64$")
      list(APPEND cmake_configure_args -A x64)
    endif()
  endif()

  if(DEFINED CMAKE_BUILD_TYPE)
    if(WIN32)
      build_command(_build_command CONFIGURATION ${CMAKE_BUILD_TYPE})
      list(APPEND cmake_commands "BUILD_COMMAND ${_build_command}")
    else()
      list(APPEND cmake_configure_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
    endif()
  endif()

  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND cmake_configure_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
    if(ANDROID)
      if(DEFINED ANDROID_ABI)
        list(APPEND cmake_configure_args "-DANDROID_ABI=${ANDROID_ABI}")
      endif()
      if(DEFINED ANDROID_CPP_FEATURES)
        list(APPEND cmake_configure_args "-DANDROID_CPP_FEATURES=${ANDROID_CPP_FEATURES}")
      endif()
      if(DEFINED ANDROID_FUNCTION_LEVEL_LINKING)
        list(APPEND cmake_configure_args "-DANDROID_FUNCTION_LEVEL_LINKING=${ANDROID_FUNCTION_LEVEL_LINKING}")
      endif()
      if(DEFINED ANDROID_NATIVE_API_LEVEL)
        list(APPEND cmake_configure_args "-DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL}")
      endif()
      if(DEFINED ANDROID_NDK)
        list(APPEND cmake_configure_args "-DANDROID_NDK=${ANDROID_NDK}")
      endif()
      if(DEFINED ANDROID_STL)
        list(APPEND cmake_configure_args "-DANDROID_STL=${ANDROID_STL}")
      endif()
      if(DEFINED ANDROID_TOOLCHAIN_NAME)
        list(APPEND cmake_configure_args "-DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME}")
      endif()
    endif()
  else()
    list(APPEND cmake_configure_args "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
  endif()
  set(json_external_project_dir ${CMAKE_CURRENT_BINARY_DIR}/json_external_project)
  include(ExternalProject)
  set(nlohmann_json_schema_validator_version "main")
  externalproject_add(nlohmann_json_schema_validator-${nlohmann_json_schema_validator_version}
    PREFIX ${json_external_project_dir}
    GIT_REPOSITORY https://github.com/pboettch/json-schema-validator.git
    GIT_TAG ${nlohmann_json_schema_validator_version}
    GIT_CONFIG advice.detachedHead=false
    # Suppress git update due to https://gitlab.kitware.com/cmake/cmake/-/issues/16419
    UPDATE_COMMAND ""
    PATCH_COMMAND ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> git apply -p1 --ignore-space-change --whitespace=nowarn
        ${CMAKE_CURRENT_SOURCE_DIR}/patchcmake.patch
    TIMEOUT 6000
    ${cmake_commands}
    BUILD_ALWAYS ON
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${json_external_project_dir}/install/
	  -DBUILD_TESTS:BOOL=OFF
	  -DBUILD_EXAMPLES:BOOL=OFF
      -DBUILD_SHARED_LIBS:BOOL=true
      ${cmake_configure_args}
    INSTALL_DIR ${json_external_project_dir}/install/
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    DIRECTORY
      ${json_external_project_dir}/install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
    USE_SOURCE_PERMISSIONS
  )

endmacro()

build_nlohmann_json_schema_validator()

ament_export_include_directories(include)
ament_export_libraries(nlohmann_json_schema_validator)
ament_export_dependencies(nlohmann_json_schema_validator)

# this ensures that the package has an environment hook setting the PATH
ament_package()
