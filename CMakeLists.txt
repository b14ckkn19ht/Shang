#Modifid from clang cmakelist.txt

set(VBE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(VBE_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})

if(MSVC) # Work around for the fucking msvc bug on rtti.
  set(LLVM_REQUIRES_RTTI 1)
endif (MSVC)
# Add appropriate flags for GCC
# if (CMAKE_COMPILER_IS_GNUCXX)
  # FIXME: Turn off exceptions, RTTI:
  # -fno-exceptions -fno-rtti
  # set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-common -Woverloaded-virtual -Wno-long-long -Wall -W -Wno-unused-parameter -Wwrite-strings")
# endif ()

#Add path for custom modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${VBE_SOURCE_DIR}/cmake")

FIND_PACKAGE(BOOST REQUIRED)
FIND_PACKAGE(Lua REQUIRED)
FIND_PACKAGE(LuaBind REQUIRED)
FIND_PACKAGE(LpSolve REQUIRED)

option(ENABLE_LOGIC_SYNTHESIS
       "Enable the Pre-schedule bitlevel optimization by ABC" OFF)
if (ENABLE_LOGIC_SYNTHESIS)
FIND_PACKAGE(ABC REQUIRED)
endif(ENABLE_LOGIC_SYNTHESIS)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR} #include the inc files generated by tablegen
  )

install(DIRECTORY include
  DESTINATION .
  PATTERN ".svn" EXCLUDE
  )

add_definitions( -D_GNU_SOURCE )

set(LLVM_TARGET_DEFINITIONS VTM.td)
tablegen(LLVM VerilogBackendGenRegisterInfo.inc -gen-register-info)
tablegen(LLVM VerilogBackendGenInstrInfo.inc -gen-instr-info)
tablegen(LLVM VerilogBackendGenDAGISel.inc -gen-dag-isel)
tablegen(LLVM VerilogBackendGenIntrinsics.inc -gen-tgt-intrinsic)
add_public_tablegen_target(VerilogBackendTableGen)

add_llvm_target(VerilogBackend
  DatapathCodehMotion.cpp
  DataPathPromotion.cpp
  DeadMemOpElimination.cpp
  FixMachineCode.cpp
  FixTerminators.cpp
  HyperBlockFormation.cpp
  MemOpsFusing.cpp
  ScriptingPass.cpp
  VTargetMachine.cpp
  VFrameLowering.cpp
  VFInfo.cpp
  VISelDAGToDAG.cpp
  VISelLowering.cpp
  VInstrInfo.cpp
  VIntrinsicsInfo.cpp
  VRegisterInfo.cpp
  VSelectionDAGInfo.cpp
  MachineBasicBlockTopOrder.cpp
)

target_link_libraries (LLVMVerilogBackend
  LLVMVerilogBackendDesc
  VTMBitLevelOpt
  VTMHighLevelOpt
  VTMSchedule
  VTMRTLCodegen 
  VTMScripting)

add_subdirectory(include)
add_subdirectory(lib)
add_subdirectory(MCTargetDesc)
add_subdirectory(tools)
option(ENABLE_TESTSUITE "Run the testsuite" ON)
if (ENABLE_TESTSUITE)
  add_subdirectory(testsuite)
endif (ENABLE_TESTSUITE)

IF (NOT MSVC)
  add_subdirectory(util/TestAllDelayandLEs)
ENDIF (NOT MSVC)

# TODO: docs.
