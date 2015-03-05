set(gammaray_clientlib_srcs
  client.cpp
  remotemodel.cpp
  selectionmodelclient.cpp
  clientconnectionmanager.cpp
  propertycontrollerclient.cpp
  probecontrollerclient.cpp
  clientdevice.cpp
  tcpclientdevice.cpp
  localclientdevice.cpp
)

add_library(gammaray_clientlib SHARED ${gammaray_clientlib_srcs})
set_target_properties(gammaray_clientlib PROPERTIES
  ${GAMMARAY_DEFAULT_LIBRARY_PROPERTIES}
  OUTPUT_NAME gammaray_client-${GAMMARAY_PROBE_ABI}
)

target_link_libraries(gammaray_clientlib
  LINK_PUBLIC
  gammaray_ui
  ${QT_QTCORE_LIBRARIES}

  LINK_PRIVATE
  gammaray_ui_internal
  gammaray_common
  ${QT_QTGUI_LIBRARIES}
  ${QT_QTNETWORK_LIBRARIES}
)


set(gammaray_client_srcs main.cpp)
qt4_add_resources(gammaray_client_srcs ${CMAKE_SOURCE_DIR}/resources/gammaray.qrc)

add_executable(gammaray-client WIN32 ${gammaray_client_srcs})

target_link_libraries(gammaray-client
  gammaray_clientlib
  gammaray_common
)

if(QNXNTO)
  target_link_libraries(gammaray-client cpp)
endif()

gammaray_embed_info_plist(gammaray-client ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in)
set_target_properties(gammaray-client PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/${LIBEXEC_INSTALL_DIR}"
)

install(TARGETS gammaray_clientlib EXPORT GammaRayTargets ${INSTALL_TARGETS_DEFAULT_ARGS})
install(TARGETS gammaray-client DESTINATION ${LIBEXEC_INSTALL_DIR})


gammaray_install_headers(
  gammaray_clientlib_export.h
  clientconnectionmanager.h
)

ecm_generate_pri_file(BASE_NAME GammaRayClient
                      LIB_NAME gammaray_client-${GAMMARAY_PROBE_ABI}
                      DEPS "core gui network GammaRayCommon"
                      FILENAME_VAR PRI_FILENAME
)

install(FILES ${PRI_FILENAME} DESTINATION ${ECM_MKSPECS_INSTALL_DIR})
