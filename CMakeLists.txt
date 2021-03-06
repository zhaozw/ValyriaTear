CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
CMAKE_POLICY(VERSION 2.6)

PROJECT(VALYRIATEAR)

OPTION(EDITOR_SUPPORT "Compile the Qt editor along with the game" OFF)
OPTION(DEBUG_FEATURES "Compile the game with the debug features" OFF)
OPTION(USE_SYSTEM_LUABIND "Use the luabind headers and lib provided by the system (Linux only)" OFF)
OPTION(DISABLE_TRANSLATIONS "Disable gettext / l10n support" OFF)
OPTION(USE_PCH_COMPILATION "Using precompiled header for compilation for GCC" ON)

IF (NOT VERSION)
    SET(VERSION 0.1.0)
ENDIF()

STRING(REPLACE "." " " _VERSION ${VERSION})
SEPARATE_ARGUMENTS(_VERSION)
LIST(LENGTH _VERSION _LEN)
IF(NOT _LEN EQUAL 4 AND NOT _LEN EQUAL 3)
    MESSAGE(FATAL_ERROR "Version needs to be in the form MAJOR.MINOR.RELEASE[.BUILD]")
ENDIF()

LIST(GET _VERSION 0 VER_MAJOR)
LIST(GET _VERSION 1 VER_MINOR)
LIST(GET _VERSION 2 VER_RELEASE)
IF(_LEN EQUAL 4)
    LIST(GET _VERSION 3 VER_BUILD)
ELSE()
    SET(VER_BUILD 0)
ENDIF()

# where to look for cmake modules
SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMake/Modules)

SET(PKG_NAME "valyriatear")
IF (WIN32)
    SET(PKG_DATADIR ".")
    SET(PKG_BINDIR ".")
    SET(LOCALEDIR ".")
    CONFIGURE_FILE(src/winver.h.in src/winver.h)
ELSE (WIN32)
    IF (NOT OSX AND NOT BEOS)
        OPTION(USE_X11 "Use X11 Clipboard functionality" ON)
    ENDIF ()
    SET(PKG_DATADIR ${CMAKE_INSTALL_PREFIX}/share/valyriatear CACHE PATH "Data dir")
    SET(LOCALEDIR ${CMAKE_INSTALL_PREFIX}/share/locale CACHE PATH "Locale dir")
    SET(PKG_BINDIR ${CMAKE_INSTALL_PREFIX}/bin CACHE PATH "Binary dir")
ENDIF (WIN32)

# The sub-folders to parse
ADD_SUBDIRECTORY(src)

# Add data packages
IF(NOT DISABLE_TRANSLATIONS)
    FIND_PACKAGE(Gettext)
    ADD_SUBDIRECTORY(po)
ENDIF(NOT DISABLE_TRANSLATIONS)

# CPack installation part
If(UNIX)
    # Shortcut desktop file
    INSTALL(FILES "${CMAKE_CURRENT_SOURCE_DIR}/valyriatear.desktop" DESTINATION share/applications)
    # data files
    INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/dat" DESTINATION ${PKG_DATADIR} FILES_MATCHING PATTERN "*.lua" PATTERN "*.png")
    INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/img" DESTINATION ${PKG_DATADIR} FILES_MATCHING PATTERN "*.lua" PATTERN "*.png" PATTERN "*.ttf")
    INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/mus" DESTINATION ${PKG_DATADIR} FILES_MATCHING PATTERN "*.ogg")
    INSTALL(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/snd" DESTINATION ${PKG_DATADIR} FILES_MATCHING PATTERN "*.wav" PATTERN "*.ogg")
    # icon file
    INSTALL(FILES "${CMAKE_CURRENT_SOURCE_DIR}/img/logos/program_icon_48x48.png"
            DESTINATION ${CMAKE_INSTALL_PREFIX}/share/icons/hicolor/48x48/apps RENAME valyriatear.png)
ENDIF()

SET(CPACK_PACKAGE_NAME "valyriatear")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Valyria Tear")
SET(CPACK_PACKAGE_VENDOR "Bertram")
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README")
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING.GPL-2" )
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "ValyriaTear")
SET(CPACK_PACKAGE_VERSION_MAJOR ${VER_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${VER_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH ${VER_RELEASE})
IF(WIN32)
    SET(CPACK_NSIS_DISPLAY_NAME "Valyria Tear")
    SET(CPACK_NSIS_MUI_ICON "${CMAKE_CURRENT_SOURCE_DIR}/img/logos/program_icon.ico")
    SET(CPACK_PACKAGE_ICON "${CMAKE_CURRENT_SOURCE_DIR}/img/logos/program_icon.png") #TODO: use a bmp
    SET(CPACK_NSIS_URL_INFO_ABOUT "http://valyriatear.blogspot.com")
ENDIF()
INCLUDE(CPack)
