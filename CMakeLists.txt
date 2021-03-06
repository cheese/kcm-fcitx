cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
project(kcm-fcitx)

set(QT_MIN_VERSION "5.3.0")
find_package(ECM 1.0.0 REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings)
include(FeatureSummary)
option(ENABLE_TEST "Enable test" Off)

find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS Core Gui Widgets X11Extras)

find_package(KF5 REQUIRED COMPONENTS
    CoreAddons
    WidgetsAddons
    ConfigWidgets
    ItemViews
    I18n
    KIO
    NewStuff
)

find_package(FcitxQt5DBusAddons 1.1.0 REQUIRED)
find_package(FcitxQt5WidgetsAddons 1.1.0 REQUIRED)

set(MAJOR_VERSION "0")
set(MINOR_VERSION "5")
set(PATCH_VERSION "4")
set(SUFFIX_VERSION "")

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

set(VERSION_STRING "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}")
if(NOT SUFFIX_VERSION)
    set(VERSION_STRING_FULL "${VERSION_STRING}")
else(NOT SUFFIX_VERSION)
    set(VERSION_STRING_FULL "${VERSION_STRING}-${SUFFIX_VERSION}")
endif(NOT SUFFIX_VERSION)

find_package(PkgConfig REQUIRED)
find_package(Gettext REQUIRED)
find_package(Fcitx 4.2.8 REQUIRED)
find_package(XkbFile REQUIRED)
find_package(X11 REQUIRED)

set(CMAKE_C_FLAGS "-Wall -Wextra -Wno-sign-compare -Wno-unused-parameter -fvisibility=hidden ${CMAKE_C_FLAGS}")
set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-sign-compare -Wno-unused-parameter -fvisibility=hidden ${CMAKE_CXX_FLAGS}")
set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined,--as-needed ${CMAKE_SHARED_LINKER_FLAGS}")
set(CMAKE_MODULE_LINKER_FLAGS "-Wl,--no-undefined,--as-needed ${CMAKE_MODULE_LINKER_FLAGS}")

string(REGEX REPLACE "^([0-9]+)\\.[0-9]+\\.[0-9]+.*" "\\1" FCITX4_MAJOR_VERSION "${FCITX4_VERSION}")
string(REGEX REPLACE "^[0-9]+\\.([0-9]+)\\.[0-9]+.*" "\\1" FCITX4_MINOR_VERSION "${FCITX4_VERSION}")
string(REGEX REPLACE "^[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" FCITX4_PATCH_VERSION "${FCITX4_VERSION}")

EXECUTE_PROCESS(COMMAND "${FCITX4_FCITXCONFIG_EXECUTABLE}" --exec-prefix OUTPUT_VARIABLE FCITX4_EXEC_PREFIX ERROR_QUIET OUTPUT_STRIP_TRAILING_WHITESPACE)

set(package ${FCITX4_PACKAGE_NAME})
set(datadir ${CMAKE_INSTALL_PREFIX}/share)
set(localedir ${CMAKE_INSTALL_PREFIX}/share/locale)
set(exec_prefix "${CMAKE_INSTALL_PREFIX}/bin")

find_program(some_x_program NAMES iceauth xrdb xterm)
if (NOT some_x_program)
    set(some_x_program /usr/bin/xrdb)
    message("Warning: Could not determine X binary directory. Assuming /usr/bin.")
endif (NOT some_x_program)
get_filename_component(proto_xbindir "${some_x_program}" PATH)
get_filename_component(XBINDIR "${proto_xbindir}" ABSOLUTE)
get_filename_component(xrootdir "${XBINDIR}" PATH)
set(XLIBDIR "${xrootdir}/lib/X11")

configure_file(config.h.in config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(
    ${FCITX4_FCITX_INCLUDE_DIRS}
    ${FCITX4_FCITX_UTILS_INCLUDE_DIRS}
    ${FCITX4_FCITX_CONFIG_INCLUDE_DIRS}
    )

link_directories(
    ${FCITX4_FCITX_LIBRARY_DIRS}
    ${FCITX4_FCITX_UTILS_LIBRARY_DIRS}
    ${FCITX4_FCITX_CONFIG_LIBRARY_DIRS}
    )

add_subdirectory(po)
add_subdirectory(src)
add_subdirectory(layout)
