cmake_minimum_required(VERSION 3.22)
project(BetterHitreg LANGUAGES CXX)

# Build for Android with the NDK toolchain (see README.md or .github/workflows/build.yml).
if(NOT ANDROID)
  message(FATAL_ERROR "BetterHitreg is an Android native mod. Configure with the NDK's android.toolchain.cmake.")
endif()
if(NOT CMAKE_ANDROID_ARCH_ABI STREQUAL "arm64-v8a" AND NOT CMAKE_ANDROID_ARCH_ABI STREQUAL "armeabi-v7a")
  message(FATAL_ERROR "Unsupported Android ABI: ${CMAKE_ANDROID_ARCH_ABI}")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# The package directory name is the runtime mod id - keep it stable after release.
set(MOD_ID "BetterHitreg")
set(MOD_LIBRARY_NAME "BetterHitreg") # -> libBetterHitreg.so

# manifest.json is the single source of truth for name/version/entry.
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/manifest.json" MANIFEST_JSON)
string(JSON MOD_VERSION GET "${MANIFEST_JSON}" version)
string(JSON MOD_ENTRY GET "${MANIFEST_JSON}" entry)
if(NOT MOD_ENTRY STREQUAL "lib${MOD_LIBRARY_NAME}.so")
  message(FATAL_ERROR "manifest.json entry (${MOD_ENTRY}) must be lib${MOD_LIBRARY_NAME}.so")
endif()

# --- SDK and header-only dependencies (versions pinned) -------------------------------------
include(FetchContent)

FetchContent_Declare(nlohmann_json GIT_REPOSITORY https://github.com/nlohmann/json.git GIT_TAG v3.11.3)
FetchContent_Declare(boost_pfr GIT_REPOSITORY https://github.com/boostorg/pfr.git GIT_TAG boost-1.88.0)
FetchContent_Declare(magic_enum GIT_REPOSITORY https://github.com/Neargye/magic_enum.git GIT_TAG v0.9.7)
FetchContent_Declare(fmt GIT_REPOSITORY https://github.com/fmtlib/fmt.git GIT_TAG 11.2.0)
FetchContent_Declare(preloader_android GIT_REPOSITORY https://github.com/LiteLDev/preloader-android.git GIT_TAG 0.2.2)

set(FMT_DOC OFF CACHE BOOL "" FORCE)
set(FMT_TEST OFF CACHE BOOL "" FORCE)
set(FMT_INSTALL OFF CACHE BOOL "" FORCE)

# Dependencies first, then the SDK, so the SDK reuses the same copies.
FetchContent_MakeAvailable(nlohmann_json boost_pfr magic_enum fmt)
FetchContent_MakeAvailable(preloader_android)

# --- The mod library --------------------------------------------------------------------------
add_library(${MOD_LIBRARY_NAME} SHARED
  src/main.cpp
  src/mod/BetterHitregMod.cpp
  src/hooks/BedrockBridge.cpp
)

target_include_directories(${MOD_LIBRARY_NAME} PRIVATE
  src
  "${preloader_android_SOURCE_DIR}/include"
)

target_compile_options(${MOD_LIBRARY_NAME} PRIVATE
  -fvisibility=hidden
  -fvisibility-inlines-hidden
)
target_link_options(${MOD_LIBRARY_NAME} PRIVATE "-Wl,-z,max-page-size=16384")

find_library(log-lib log REQUIRED)
target_link_libraries(${MOD_LIBRARY_NAME} PRIVATE
  ${log-lib}
  preloader
  nlohmann_json::nlohmann_json
  Boost::pfr
  magic_enum::magic_enum
  fmt::fmt
)

if(NOT CMAKE_STRIP)
  message(FATAL_ERROR "CMAKE_STRIP is required to strip the generated library.")
endif()
add_custom_command(TARGET ${MOD_LIBRARY_NAME} POST_BUILD
  COMMAND "${CMAKE_STRIP}" --strip-unneeded "$<TARGET_FILE:${MOD_LIBRARY_NAME}>"
  COMMENT "Stripping symbols from lib${MOD_LIBRARY_NAME}.so"
  VERBATIM
)

# --- .levipack --------------------------------------------------------------------------------
# A .levipack is a zip with one top-level folder (the mod id):
#   BetterHitreg/{manifest.json, libBetterHitreg.so, config/config.json, config/config.schema.json}
set(LEVI_PACKAGE_ROOT "${CMAKE_CURRENT_BINARY_DIR}/package")
set(LEVI_PACKAGE_DIR "${LEVI_PACKAGE_ROOT}/${MOD_ID}")
set(LEVI_PACKAGE_FILE "${CMAKE_CURRENT_BINARY_DIR}/${MOD_ID}-${MOD_VERSION}-${CMAKE_ANDROID_ARCH_ABI}.levipack")

add_custom_target(levi_package
  COMMAND "${CMAKE_COMMAND}" -E rm -rf "${LEVI_PACKAGE_ROOT}"
  COMMAND "${CMAKE_COMMAND}" -E make_directory "${LEVI_PACKAGE_DIR}/config"
  COMMAND "${CMAKE_COMMAND}" -E copy "$<TARGET_FILE:${MOD_LIBRARY_NAME}>" "${LEVI_PACKAGE_DIR}/lib${MOD_LIBRARY_NAME}.so"
  COMMAND "${CMAKE_COMMAND}" -E copy "${CMAKE_CURRENT_SOURCE_DIR}/manifest.json" "${LEVI_PACKAGE_DIR}/manifest.json"
  COMMAND "${CMAKE_COMMAND}" -E copy "${CMAKE_CURRENT_SOURCE_DIR}/config/config.json" "${LEVI_PACKAGE_DIR}/config/config.json"
  COMMAND "${CMAKE_COMMAND}" -E copy "${CMAKE_CURRENT_SOURCE_DIR}/config/config.schema.json" "${LEVI_PACKAGE_DIR}/config/config.schema.json"
  COMMAND "${CMAKE_COMMAND}" -E chdir "${LEVI_PACKAGE_ROOT}" "${CMAKE_COMMAND}" -E tar cf "${LEVI_PACKAGE_FILE}" --format=zip "${MOD_ID}"
  DEPENDS ${MOD_LIBRARY_NAME}
  COMMENT "Packaging ${MOD_ID} for Levi Launcher"
  VERBATIM
)
