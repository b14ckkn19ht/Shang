add_llvm_library(VTMHighLevelOpt
  FunctionFilter.cpp
  HLSInliner.cpp
  TrivialLoopUnrollPass.cpp
  )

llvm_config(VTMHighLevelOpt transformutils ipo)

add_dependencies(VTMHighLevelOpt VerilogBackendTableGen)
