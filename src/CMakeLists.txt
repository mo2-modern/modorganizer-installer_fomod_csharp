cmake_minimum_required(VERSION 3.16)

find_package(mo2-cmake CONFIG REQUIRED)
find_package(mo2-uibase CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)

add_library(installer_fomod_csharp SHARED)
mo2_configure_plugin(installer_fomod_csharp WARNINGS OFF CLI ON WERROR ON)

# spdlog::spdlog, NOT spdlog::spdlog_header_only -- the distinction is the whole fix.
# uibase's public log.h forward-declares spdlog::logger while holding a
# std::unique_ptr<spdlog::logger>, and /clr cannot put an incomplete native type into
# managed metadata (LNK4248). Completing the type needs spdlog's headers, but the
# header-only target additionally pulls the *-inl.h implementation headers, which use
# thread_local -- illegal in managed code (C2482/C2483). The compiled-lib target
# defines SPDLOG_COMPILED_LIB, so logger.h stops at the class definition. Layout is
# identical between the two modes: logger's members vary only on
# SPDLOG_USE_STD_FORMAT (both define it) and SPDLOG_WCHAR_TO_UTF8_SUPPORT (neither).
target_link_libraries(installer_fomod_csharp PRIVATE mo2::uibase spdlog::spdlog)
target_compile_options(installer_fomod_csharp PRIVATE "/FIspdlog/logger.h")
mo2_install_plugin(installer_fomod_csharp)

set_target_properties(installer_fomod_csharp PROPERTIES CXX_STANDARD 20)
