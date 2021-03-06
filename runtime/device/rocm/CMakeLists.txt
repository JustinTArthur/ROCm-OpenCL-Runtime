include(bc2h)

set(INC_SUFFIX "amdgcn.inc")

include_directories(${CMAKE_SOURCE_DIR}/compiler/lib/loaders/elf/utils/common)
include_directories(${CMAKE_SOURCE_DIR}/compiler/lib/loaders/elf/utils/libelf)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${ROCM_OCL_INCLUDES})

add_library(oclrocm OBJECT
  rocprintf.cpp
  rocprogram.cpp
  rocmemory.cpp
  rocdevice.cpp
  roccompiler.cpp
  rocblit.cpp
  rockernel.cpp
  rocvirtual.cpp
  rocglinterop.cpp
  rocappprofile.cpp
  rocsettings.cpp
)
set_target_properties(oclrocm PROPERTIES POSITION_INDEPENDENT_CODE ON)

# generating libraries.amdgcn.inc
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc "// Automatically generated file; DO NOT EDIT.\n")

foreach(AMDGCN_LIB_TARGET ${AMDGCN_LIB_TARGETS})
  get_target_property(lib_file_name ${AMDGCN_LIB_TARGET} ARCHIVE_OUTPUT_NAME)
  get_target_property(lib_file_path ${AMDGCN_LIB_TARGET} ARCHIVE_OUTPUT_DIRECTORY)
  set(bclib "${lib_file_path}/${lib_file_name}.lib.bc")
  set(header "${lib_file_name}.${INC_SUFFIX}")
  set(symbol "${lib_file_name}_amdgcn")
  add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${header}
    COMMAND bc2h ${bclib} ${CMAKE_CURRENT_BINARY_DIR}/${header} ${symbol}
    DEPENDS bc2h ${AMDGCN_LIB_TARGET}
    COMMENT "Generating ${header}"
  )
  set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_CURRENT_BINARY_DIR}/${target}.inc)

  add_custom_target(${header} ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${header})
  add_dependencies(oclrocm ${header})

  file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc "#include \"${header}\"\n")
endforeach()

file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc
  "static inline std::pair<const void*, size_t>\nget_oclc_isa_version(uint gfxip)\n{\nswitch (gfxip) {\n")
foreach(AMDGCN_LIB_TARGET ${AMDGCN_LIB_TARGETS})
  if (${AMDGCN_LIB_TARGET} MATCHES "^oclc_isa_version_[0-9]+_lib$")
    string(REGEX REPLACE "^oclc_isa_version_([0-9]+)_lib$" "\\1" gfxip ${AMDGCN_LIB_TARGET})
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc
      "case ${gfxip}: return std::make_pair(oclc_isa_version_${gfxip}_amdgcn, oclc_isa_version_${gfxip}_amdgcn_size); break;\n")
  endif()
endforeach()
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc
  "default: return std::make_pair((const void*)0,(size_t)0);\n}\n}\n")

foreach(AMDGCN_LIB_TARGET ${AMDGCN_LIB_TARGETS})
  if (${AMDGCN_LIB_TARGET} MATCHES "oclc_(.*)_on_lib")
    string(REGEX REPLACE "oclc_(.*)_on_lib" "\\1" function ${AMDGCN_LIB_TARGET})
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libraries.amdgcn.inc
      "static inline std::pair<const char*, size_t> get_oclc_${function}(bool on)\n{ return std::make_pair("
                "(const char*)(on ? oclc_${function}_on_amdgcn : oclc_${function}_off_amdgcn),"
                "on ? oclc_${function}_on_amdgcn_size : oclc_${function}_off_amdgcn_size);}\n")
  endif()
endforeach()

# generating opencl*.inc files
add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.pch
  COMMAND clang -cc1 -x cl-header -triple amdgcn-amd-amdhsa-opencl -Werror -O3 -Dcl_khr_fp64 -Dcl_khr_fp16 -DNDEBUG -cl-std=CL1.2 -emit-pch -o ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.pch < ${CMAKE_SOURCE_DIR}/compiler/llvm/tools/clang/lib/Headers/opencl-c.h 
  DEPENDS clang ${CMAKE_SOURCE_DIR}/compiler/llvm/tools/clang/lib/Headers/opencl-c.h
  COMMENT "Generating opencl1.2-c.amdgcn.pch"
)
add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.inc
  COMMAND bc2h ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.pch ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.inc opencl1_2_c_amdgcn
  DEPENDS bc2h ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.pch
  COMMENT "Generating opencl1.2-c.amdgcn.inc"
)
set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.inc ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.pch)
add_custom_target(opencl1.2-c.amdgcn.inc ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/opencl1.2-c.amdgcn.inc)
add_dependencies(oclrocm opencl1.2-c.amdgcn.inc)

add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.pch
  COMMAND clang -cc1 -x cl-header -triple amdgcn-amd-amdhsa-opencl -Werror -O3 -Dcl_khr_fp64 -Dcl_khr_fp16 -DNDEBUG -cl-std=CL2.0 -emit-pch -o ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.pch < ${CMAKE_SOURCE_DIR}/compiler/llvm/tools/clang/lib/Headers/opencl-c.h 
  DEPENDS clang ${CMAKE_SOURCE_DIR}/compiler/llvm/tools/clang/lib/Headers/opencl-c.h
  COMMENT "Generating opencl2.0-c.amdgcn.pch"
)
add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.inc
  COMMAND bc2h ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.pch ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.inc opencl2_0_c_amdgcn
  DEPENDS bc2h ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.pch
  COMMENT "Generating opencl2.0-c.amdgcn.inc"
)
set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.inc ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.pch)
add_custom_target(opencl2.0-c.amdgcn.inc ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/opencl2.0-c.amdgcn.inc)
add_dependencies(oclrocm opencl2.0-c.amdgcn.inc)
