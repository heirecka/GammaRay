add_subdirectory(injector)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
)

# shared launcher code
set(gammaray_launcher_shared_srcs
  injector/abstractinjector.cpp
  injector/processinjector.cpp
  injector/injectorfactory.cpp
  injector/preloadinjector.cpp
  injector/styleinjector.cpp
  injector/windllinjector.cpp
  injector/interactiveprocess.cpp
  injector/preloadcheck.cpp

  probefinder.cpp
  launchoptions.cpp
  clientlauncher.cpp
  launcherfinder.cpp
)
if(NOT WIN32)
  list(APPEND gammaray_launcher_shared_srcs
    injector/debuggerinjector.cpp
    injector/gdbinjector.cpp
    injector/lldbinjector.cpp
  )
endif()
add_library(gammaray_launcher_shared STATIC ${gammaray_launcher_shared_srcs})
target_link_libraries(gammaray_launcher_shared ${QT_QTCORE_LIBRARIES} gammaray_common_internal)
if(HAVE_QT_WIDGETS)
  target_link_libraries(gammaray_launcher_shared ${QT_QTGUI_LIBRARIES})
endif()

# command line launcher
set(gammaray_runner_srcs
  launcher.cpp
  main.cpp
)

#if(APPLE)
#  set(MACOSX_BUNDLE_INFO_STRING "{GAMMARAY_LONG_NAME} ${GAMMARAY_VERSION_STRING}")
#  set(MACOSX_BUNDLE_ICON_FILE "{GAMMARAY_SHORT_NAME}.icns")
#  set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.kdab.{GAMMARAY_SHORT_NAME}.injector")
#  set(MACOSX_BUNDLE_BUNDLE_NAME "${GAMMARAY_LONG_NAME}")
#  set(MACOSX_BUNDLE_LONG_VERSION_STRING "${GAMMARAY_VERSION_STRING}")
#  set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${GAMMARAY_VERSION_STRING}")
#  set(MACOSX_BUNDLE_BUNDLE_VERSION "${GAMMARAY_VERSION}")
#  set(MACOSX_BUNDLE_COPYRIGHT "KDAB")
#  add_executable("${GAMMARAY_LONG_NAME}" MACOSX_BUNDLE ${gammaray_runner_srcs})
#else()
  add_executable(gammaray ${gammaray_runner_srcs})
#endif()

target_link_libraries(gammaray gammaray_launcher_shared)

if(QNXNTO)
  target_link_libraries(gammaray cpp)
endif()

if(UNIX AND NOT APPLE AND NOT QNXNTO)
  target_link_libraries(gammaray dl) # for preload check
endif()

if(APPLE)
  gammaray_embed_info_plist(gammaray ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/Info.plist DESTINATION "${GAMMARAY_LONG_NAME}.app/Contents")
  install(FILES ../resources/${GAMMARAY_LONG_NAME}.icns DESTINATION "${RESOURCES_INSTALL_DIR}")
endif()

install(TARGETS gammaray ${INSTALL_TARGETS_DEFAULT_ARGS})

# UI launcher
if(HAVE_QT_CONCURRENT AND HAVE_QT_WIDGETS AND GAMMARAY_BUILD_UI)
  add_subdirectory(ui)
endif()
