# CMakeLists.txt for nsx-nanopb
#
# Vendored nanopb v0.4.9.1 (Zlib license).
# Pure portable C — no Ambiq/SoC dependency.
#
# Pre-generated .pb.c/.pb.h files produced from application .proto files
# must include this module's includes-api/ directory (already provided via
# the PUBLIC include path below) and must be compiled into each application's
# own target, NOT into this library.

cmake_minimum_required(VERSION 3.20)

if(NOT DEFINED NSX_BOARD_FLAGS_TARGET)
    message(FATAL_ERROR "NSX_BOARD_FLAGS_TARGET is not defined – include a board.cmake first.")
endif()

add_library(nsx_nanopb STATIC
    ${CMAKE_CURRENT_LIST_DIR}/src/pb_common.c
    ${CMAKE_CURRENT_LIST_DIR}/src/pb_encode.c
    ${CMAKE_CURRENT_LIST_DIR}/src/pb_decode.c
)
set_target_properties(nsx_nanopb PROPERTIES EXPORT_NAME nanopb)
add_library(nsx::nanopb ALIAS nsx_nanopb)

# Inherit board-specific arch flags (cpu, thumb, float ABI, etc.).
target_link_libraries(nsx_nanopb PUBLIC ${NSX_BOARD_FLAGS_TARGET})

# Disable dynamic memory allocation — the whole point of using nanopb on
# a microcontroller.  Applications can override this on their own target if
# they truly need it, but should not add -DPB_ENABLE_MALLOC to this target.
target_compile_definitions(nsx_nanopb PUBLIC
    PB_NO_ERRMSG=1          # saves ~1 KB of error-string flash
)

target_include_directories(nsx_nanopb PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/includes-api>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/nsx/modules/nsx-nanopb/includes-api>
)

install(DIRECTORY
    ${CMAKE_CURRENT_LIST_DIR}/includes-api
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/nsx/modules/nsx-nanopb
)

install(TARGETS
    nsx_nanopb
    EXPORT nsxTargets
)
