# -*-cmake-*-
find_package(Qt4 REQUIRED)

INCLUDE( ${QT_USE_FILE} )

#set(CMAKE_VERBOSE_MAKEFILE true)

SET(BundleName "qt-gui")

SET(guiFILES "application-states.cpp"
  "controllers.cpp"
  "configuration.cpp"
  "gui/avatar-selection.cpp"
  "gui/main-window.cpp"
  "gui/avatar.cpp"
  "gui/mesh-drawer.cpp"
  "gui/graphics-scrollbar.cpp"
  "gui/windowed-gui-controller.cpp"
  "gui/item-controllers.cpp"
  "gui/worker-thread.cpp"
  "gui/item-positions-calculator.cpp"
)

SET(MOC_HEADERS
  "controllers.hpp"
  "gui/avatar-selection.hpp"
  "gui/main-window.hpp"
  "gui/avatar.hpp"
#  "gui/mesh-drawer.hpp"
  "gui/graphics-scrollbar.hpp"
  "gui/windowed-gui-controller.hpp"
  "gui/item-controllers.hpp"
  "gui/worker-thread.hpp")
#  "gui/item-positions-calculator.hpp")

QT4_WRAP_CPP(MOC_SRCS ${MOC_HEADERS})

#
#SET(APFLAGS "-objc++ -I `gnustep-config --variable=GNUSTEP_SYSTEM_HEADERS` -L -lgnustep-base -lpthread -lobjc  `gnustep-config --variable=GNUSTEP_SYSTEM_LIBRARIES` -fconstant-string-class=NSConstantString")

#SET(APFLAGS "-objc++ `gnustep-config --objc-flags -lgnustep-base` -L -lobjc -lgnustep-base  -lpthread")

#SET(APFLAGS "-objc++ `gnustep-config --objc-flags -lgnustep-base` `gnustep-config --gui-libs`")

SET(APFLAGS "-objc++ -DGNUSTEP -DGNUSTEP_BASE_LIBRARY=1 -DGNU_RUNTIME=1 -DGNUSTEP_BASE_LIBRARY=1 -fno-strict-aliasing -fexceptions -fobjc-exceptions -D_NATIVE_OBJC_EXCEPTIONS -pthread -fPIC -Wall -DGSWARN -DGSDIAGNOSE -Wno-import -g  -fgnu-runtime -fconstant-string-class=NSConstantString -fexec-charset=UTF-8 -I. -I/home/snisioi/GNUstep/Library/Headers -I/usr/local/include/GNUstep -I/usr/include/GNUstep `gnustep-config --gui-libs`")

INCLUDE_DIRECTORIES(".")
set_source_files_properties(osx-configuration.mm PROPERTIES COMPILE_FLAGS ${APFLAGS})


add_executable(demo-application
  main.cpp
  command-line-configuration.cpp
  ${guiFILES}
  ${MOC_SRCS}
  ${EXTRA_SRC})
target_link_libraries(demo-application ${LIBS} ${QT_LIBRARIES} clmTracker avatarAnim utilities ${EXTRA_LIBS})

# Apple Specific Targets

IF(APPLE)
  SET(MACOSX_BUNDLE_BUNDLE_NAME ${BundleName})
  set_source_files_properties(${CMAKE_CURRENT_SOURCE_DIR}/Configuration.plist PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  
  SET(EXTRA_SRC "Configuration.plist")

ENDIF()


ADD_EXECUTABLE(${BundleName} MACOSX_BUNDLE 
  main.cpp 
  osx-configuration.mm
  ${guiFILES} 
  ${MOC_SRCS}
  ${EXTRA_SRC}
  )
TARGET_LINK_LIBRARIES(${BundleName} ${LIBS} ${QT_LIBRARIES} clmTracker avatarAnim ${EXTRA_LIBS})


# IF(APPLE)

#  # set_target_properties(${BundleName} PROPERTIES MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Configuration.plist)
 

# ELSEIF(UNIX)

# ENDIF()


IF(APPLE)

  SET(plugin_dest_dir "${BundleName}.app/Contents/PlugIns")
  SET(qtconf_dest_dir "${BundleName}.app/Contents/Resources")
  SET(APP_FOLDER "\${CMAKE_INSTALL_PREFIX}/${BundleName}.app")
  
  INSTALL(TARGETS ${BundleName}
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION bin COMPONENT Runtime
    )
  
  FILE(GLOB trackerFiles "${CMAKE_CURRENT_SOURCE_DIR}/../tracker/resources/*")
  FILE(GLOB avatarFiles "${CMAKE_CURRENT_SOURCE_DIR}/../avatar/resources/*")

  INSTALL(FILES ${trackerFiles} DESTINATION ${qtconf_dest_dir} COMPONENT Runtime)
  INSTALL(FILES ${avatarFiles} DESTINATION ${qtconf_dest_dir} COMPONENT Runtime)

  SET(DIRS ${QT_LIBRARY_DIR} "/usr/local/" "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
  
  INSTALL(CODE "
    file(GLOB_RECURSE QTPLUGINS
      \"\${CMAKE_INSTALL_PREFIX}/${plugin_dest_dir}/*${CMAKE_SHARED_LIBRARY_SUFFIX}\")
    include(BundleUtilities)
    fixup_bundle(\"${APP_FOLDER}\" \"\${QTPLUGINS}\" \"${DIRS}\")
    " COMPONENT Runtime)

  include(CPack)

ENDIF()
