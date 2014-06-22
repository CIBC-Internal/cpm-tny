
#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME iauns_cpm_tny)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/CIBC-Internal/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# TNY
#------------------------------------------------------------------------------

# We use the origin/master tag because there are no tags to use in the Tny repo.
set(REPO_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/tny")
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${REPO_SOURCE_DIR}
  GIT_REPOSITORY "https://github.com/BobMarlon/Tny.git"
  #GIT_TAG "origin/master"
  GIT_TAG "00d0f3499170e09bf2e8a136a713e9e5113ec657"
  USE_CACHING TRUE
  )
CPM_ExportAdditionalIncludeDir("${REPO_SOURCE_DIR}/src")

# Create a tny.hpp file that simply wraps the tny.h file in extern C.
set(TNY_CPP_HEADER_FILE "${REPO_SOURCE_DIR}/src/tny/tny.hpp")
if (NOT EXISTS "${TNY_CPP_HEADER_FILE}")
  file(WRITE ${TNY_CPP_HEADER_FILE} "#if defined __cplusplus\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "extern \"C\" {\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "#endif\n\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "#include \"tny.h\"\n\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "#if defined __cplusplus\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "};\n")
  file(APPEND ${TNY_CPP_HEADER_FILE} "#endif\n")
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_CFLAGS} -std=c99")

file (GLOB Sources
  "${REPO_SOURCE_DIR}/src/tny/*.c"
  "${REPO_SOURCE_DIR}/src/tny/*.h"
  )

# Create static library to use.
add_library(${CPM_LIB_TARGET_NAME} ${Sources})
if (NOT EMSCRIPTEN AND CPM_LIBRARIES)
  target_link_libraries(${CPM_LIB_TARGET_NAME} ${CPM_LIBRARIES})
endif()

