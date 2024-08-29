cmake_minimum_required(VERSION 3.12)
project(arm_neon_clangd_fix VERSION 0.1)

# define a header-only library
add_library(arm_neon_clangd_fix INTERFACE)
add_library(arm_neon_clangd_fix::arm_neon_clangd_fix ALIAS arm_neon_clangd_fix)

target_include_directories(arm_neon_clangd_fix INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:include>)

install(
    TARGETS arm_neon_clangd_fix
    EXPORT arm_neon_clangd_fix-targets
)

install(
  FILES ${CMAKE_CURRENT_SOURCE_DIR}/arm_neon_clangd_fix.h
  DESTINATION include)
