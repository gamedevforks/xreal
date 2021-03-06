project(${XREAL_CLIENT_NAME})

include_directories(${ZLIB_INCLUDE_DIR} ${CURL_INCLUDE_DIR} ${JPEG_INCLUDE_DIR} ${PNG_INCLUDE_DIR})

file(GLOB CLIENT_HEADERS
    "*.h"
)

file(GLOB CLIENT_COMMON_SOURCES
    "../qcommon/*.c"
    "../server/*.c"
    "*.c"
    "../game/q_*.c"
    "../renderer/*.c"
)

if (${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    link_directories("$ENV{DXSDK_DIR}Lib/x86")   # For dxguid.lib

    file(GLOB CLIENT_PLATFORM_SOURCES
        "../sys/sdl_gamma.c"
        "../sys/sdl_glimp.c"
        "../sys/sdl_input.c"
        "../sys/sdl_snd.c"
        "../sys/sys_main.c"
        

        "../sys/sys_win32.c"
        "../sys/con_win32.c"
        "../sys/win/win_resource.rc"
    )

    set(CLIENT_PLATFORM_LIBRARIES "ws2_32.lib" "winmm.lib" "dxguid.lib")
    set(CLIENT_BINARY_TYPE WIN32)

    set(BUNDLE_RESOURCES "")
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(GAME_DATA_DIR xreal)

    file(GLOB CLIENT_PLATFORM_SOURCES
        "../sys/sdl_gamma.c"
        "../sys/sdl_glimp.c"
        "../sys/sdl_input.c"
        "../sys/sdl_snd.c"
        "../sys/sys_main.c"
        "../sys/con_log.c"

        "../sys/sys_unix.c"
        "../sys/con_passive.c"
        "../sys/mac/SDLMain.m"
        "../sys/mac/XreaL-Info.plist"
    )

    set(CLIENT_PLATFORM_LIBRARIES ${SDL_LIBRARY} "-framework IOKit -framework OpenGL")
    set(CLIENT_BINARY_TYPE MACOSX_BUNDLE)

    set(BUNDLE_RESOURCES
        ../sys/mac/Frameworks
        ../../../${GAME_DATA_DIR}/base
        ../../../${GAME_DATA_DIR}/docs
        ../sys/mac/xreal_icon.icns
    )
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    file(GLOB CLIENT_PLATFORM_SOURCES
        "../sys/sdl_gamma.c"
        "../sys/sdl_glimp.c"
        "../sys/sdl_input.c"
        "../sys/sdl_snd.c"
        "../sys/sys_main.c"
        "../sys/con_log.c"

        "../sys/sys_unix.c"
        "../sys/con_tty.c"
    )
    
    set(CLIENT_PLATFORM_LIBRARIES "pthread" "dl" "m" "X11" "Xext" "Xi" "Xrandr" "rt")
    set(CLIENT_BINARY_TYPE "")

    set(BUNDLE_RESOURCES "")
endif()

qf_add_executable(${XREAL_CLIENT_NAME} ${CLIENT_BINARY_TYPE} ${CLIENT_HEADERS} ${CLIENT_COMMON_SOURCES} ${CLIENT_PLATFORM_SOURCES} ${BUNDLE_RESOURCES})

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    add_dependencies(${XREAL_CLIENT_NAME} cgame game ui)

    set_target_properties(${XREAL_CLIENT_NAME} PROPERTIES MACOSX_BUNDLE_INFO_PLIST ../sys/mac/XreaL-Info.plist)
    set_target_properties(${XREAL_CLIENT_NAME} PROPERTIES XCODE_ATTRIBUTE_LD_RUNPATH_SEARCH_PATHS "@executable_path/../Frameworks @executable_path/../Resources/base @executable_path/../Resources/libs")
    set_source_files_properties(../sys/mac/xreal_icon.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

    set_source_files_properties(../sys/mac/Frameworks             PROPERTIES MACOSX_PACKAGE_LOCATION .)
    set_source_files_properties(../../../${GAME_DATA_DIR}/base 	  PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    set_source_files_properties(../../../${GAME_DATA_DIR}/docs    PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

    set(BUNDLE_PATH ${CMAKE_HOME_DIRECTORY}/build/$(CONFIGURATION)/${XREAL_CLIENT_NAME}.app/Contents)
    set(COPY_DYLIBS
        cp -r ${CMAKE_HOME_DIRECTORY}/build/$(CONFIGURATION)/base/ ${BUNDLE_PATH}/Resources/base/
    )

    add_custom_command(TARGET ${XREAL_CLIENT_NAME} POST_BUILD COMMAND ${COPY_DYLIBS})
endif()

target_link_libraries(${XREAL_CLIENT_NAME} ${CURL_LIBRARY} ${ZLIB_LIBRARY} ${JPEG_LIBRARIES} ${PNG_LIBRARIES} ${CLIENT_PLATFORM_LIBRARIES})
qf_set_output_dir(${XREAL_CLIENT_NAME} "")
