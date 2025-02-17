cmake_minimum_required(VERSION 3.15)

set(libBasename JKQTFastPlotter)
set(libIncludeSubdir jkqtfastplotter)

if(JKQtPlotter_BUILD_SHARED_LIBS)
    set(libsh_name ${libBasename}SharedLib)
    set(libsh_name_decorated ${libsh_name}${JKQtPlotter_LIBNAME_ADDITION})
    set(${libBasename}_SHARED_LIBRARY ${libsh_name_decorated}  CACHE STRING "name of static library for ${libBasename}" FORCE)
    message( STATUS "-- Building ${libsh_name}" )
endif(JKQtPlotter_BUILD_SHARED_LIBS)

if(JKQtPlotter_BUILD_STATIC_LIBS)
    set(lib_name ${libBasename}Lib)
    set(lib_name_decorated ${lib_name}${JKQtPlotter_LIBNAME_ADDITION})
    set(${libBasename}_STATIC_LIBRARY ${lib_name_decorated}  CACHE STRING "name of static library for ${libBasename}" FORCE)
    message( STATUS "-- Building ${lib_name}" )
endif(JKQtPlotter_BUILD_STATIC_LIBS)

set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# Set up source files
set(SOURCES
  ${CMAKE_CURRENT_LIST_DIR}/jkqtfastplotter.cpp
)

set(HEADERS
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/jkqtfastplotter.h>
  $<INSTALL_INTERFACE:jkqtfastplotter.h>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/jkqtfastplotter_imexport.h>
  $<INSTALL_INTERFACE:jkqtfastplotter_imexport.h>
)


include(CMakePackageConfigHelpers)


if(JKQtPlotter_BUILD_SHARED_LIBS)
    add_library(${libsh_name} SHARED ${SOURCES} ${RESOURCES} ${HEADERS})
    set_property(TARGET ${libsh_name} PROPERTY VERSION "${PROJECT_VERSION}")
    set_property(TARGET ${libsh_name} PROPERTY OUTPUT_NAME  "${libsh_name_decorated}")
    target_link_libraries(${libsh_name} PUBLIC Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Gui Qt${QT_VERSION_MAJOR}::Widgets ${QT_PRINT_SUPPORT_STRING} Qt${QT_VERSION_MAJOR}::OpenGL JKQTCommonSharedLib)
    if(${QT_VERSION_MAJOR} VERSION_LESS "6")
        set_property(TARGET ${libsh_name} PROPERTY CXX_STANDARD 11)
        set_property(TARGET ${libsh_name} PROPERTY CXX_STANDARD_REQUIRED TRUE)
        target_compile_features(${libsh_name} PUBLIC cxx_std_11)
    else()
        target_link_libraries(${libsh_name} PUBLIC Qt${QT_VERSION_MAJOR}::OpenGLWidgets)
        set_property(TARGET ${libsh_name} PROPERTY CXX_STANDARD 17)
        set_property(TARGET ${libsh_name} PROPERTY CXX_STANDARD_REQUIRED TRUE)
        target_compile_features(${libsh_name} PUBLIC cxx_std_17)
    endif()
    target_compile_definitions(${libsh_name} PUBLIC JKQTFASTPLOTTER_LIB_IN_DLL)
    if(MINGW)
      # COMPILER-SETTINGS FOR MINGW
      target_compile_options(${libsh_name} PUBLIC -fexceptions)
    elseif(MSVC)
      # COMPILER-SETTINGS FOR MS VISUAL C++
      target_compile_options(${libsh_name} PUBLIC /EHsc)
      target_compile_definitions(${libsh_name} PUBLIC NOMINMAX)
    endif()
    target_compile_definitions(${libsh_name} PRIVATE JKQTFASTPLOTTER_LIB_EXPORT_LIBRARY)
	  target_include_directories(${libsh_name} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  
    )
    set_target_properties(${libsh_name} PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS "ON")
    write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${libsh_name}Version.cmake 
                                     VERSION ${PROJECT_VERSION} 
                                     COMPATIBILITY AnyNewerVersion )    
endif()

if(JKQtPlotter_BUILD_STATIC_LIBS)
    add_library(${lib_name} STATIC ${SOURCES} ${RESOURCES} ${HEADERS})
    set_property(TARGET ${lib_name} PROPERTY VERSION "${PROJECT_VERSION}")
    set_property(TARGET ${lib_name} PROPERTY OUTPUT_NAME  "${lib_name_decorated}")
    if(MINGW)
      # COMPILER-SETTINGS FOR MINGW
      target_compile_options(${lib_name} PUBLIC -fexceptions)
    elseif(MSVC)
      # COMPILER-SETTINGS FOR MS VISUAL C++
      target_compile_options(${lib_name} PUBLIC /EHsc)
      target_compile_definitions(${lib_name} PUBLIC NOMINMAX)
    endif()
    target_link_libraries(${lib_name} PUBLIC Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Gui Qt${QT_VERSION_MAJOR}::Widgets ${QT_PRINT_SUPPORT_STRING} Qt${QT_VERSION_MAJOR}::OpenGL JKQTCommonLib)
    if(${QT_VERSION_MAJOR} VERSION_LESS "6")
        set_property(TARGET ${lib_name} PROPERTY CXX_STANDARD 11)
        set_property(TARGET ${lib_name} PROPERTY CXX_STANDARD_REQUIRED TRUE)
        target_compile_features(${lib_name} PUBLIC cxx_std_11)
    else()
        target_link_libraries(${lib_name} PUBLIC Qt${QT_VERSION_MAJOR}::OpenGLWidgets)
        set_property(TARGET ${lib_name} PROPERTY CXX_STANDARD 17)
        set_property(TARGET ${lib_name} PROPERTY CXX_STANDARD_REQUIRED TRUE)
        target_compile_features(${lib_name} PUBLIC cxx_std_17)
    endif()
    target_include_directories(${lib_name} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>  
    )
    write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${lib_name}Version.cmake
                                     VERSION ${PROJECT_VERSION} 
                                     COMPATIBILITY AnyNewerVersion )    
endif()



# Installation
if(JKQtPlotter_BUILD_SHARED_LIBS)
    install(TARGETS ${libsh_name} EXPORT ${libsh_name}_TARGETS
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${libIncludeSubdir}
    )
    set(JKQTP_CURRENT_TARGET_FILENAME "${libsh_name}Targets.cmake")
    set(JKQTP_CURRENT_TARGET_SHAREDPART "Shared")
    configure_file(LibTarget.cmake.in "${CMAKE_CURRENT_BINARY_DIR}/${libsh_name}Config.cmake" @ONLY)
    install(EXPORT ${libsh_name}_TARGETS
        FILE "${JKQTP_CURRENT_TARGET_FILENAME}"
        DESTINATION lib/cmake
    )

    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${libsh_name}Version.cmake"
                    DESTINATION lib/cmake  )
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${libsh_name}Config.cmake"
                                DESTINATION lib/cmake  )
endif(JKQtPlotter_BUILD_SHARED_LIBS)

if(JKQtPlotter_BUILD_STATIC_LIBS)
    install(TARGETS ${lib_name} EXPORT ${lib_name}_TARGETS
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${libIncludeSubdir}
    )
    set(JKQTP_CURRENT_TARGET_FILENAME "${lib_name}Targets.cmake")
    set(JKQTP_CURRENT_TARGET_SHAREDPART "")
    configure_file(LibTarget.cmake.in "${CMAKE_CURRENT_BINARY_DIR}/${lib_name}Config.cmake" @ONLY)
    install(EXPORT ${lib_name}_TARGETS
        FILE "${JKQTP_CURRENT_TARGET_FILENAME}"
        DESTINATION lib/cmake
    )

    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${lib_name}Version.cmake"
                    DESTINATION lib/cmake  )
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${lib_name}Config.cmake"
                                DESTINATION lib/cmake  )
endif(JKQtPlotter_BUILD_STATIC_LIBS)

install(FILES ${HEADERS}
		DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${libIncludeSubdir}
		COMPONENT Headers)

configure_file(${PROJECT_SOURCE_DIR}/readme.txt.in ${CMAKE_CURRENT_BINARY_DIR}/${lib_name}_Readme.txt @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${lib_name}_Readme.txt" DESTINATION doc/JKQtPlotter )
install(FILES "${PROJECT_SOURCE_DIR}/LICENSE" DESTINATION doc/JKQtPlotter RENAME "${lib_name}_LICENSE.txt" )

