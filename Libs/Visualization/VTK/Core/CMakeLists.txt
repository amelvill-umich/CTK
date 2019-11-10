project(CTKVisualizationVTKCore)

#
# 3rd party dependencies
#
set(VTK_LIBRARIES "")
find_package(VTK COMPONENTS
    ChartsCore
    CommonCore
    CommonDataModel
    FiltersCore
    FiltersGeneral
    FiltersModeling
    FiltersSources
    FiltersStatistics
    IOLegacy
    IOXML
    InfovisCore
    InfovisLayout
    InteractionStyle
    RenderingAnnotation
    RenderingCore
    RenderingFreeType
    RenderingOpenGL2
    ViewsCore
    ViewsInfovis
    WrappingPythonCore
    ViewsContext2D
    OPTIONAL_COMPONENTS
    TestingRendering
    )
if (VTK_VERSION VERSION_LESS "8.90")
  include(${VTK_USE_FILE})
endif ()
if(NOT DEFINED VTK_RENDERING_BACKEND)
  set(VTK_RENDERING_BACKEND "OpenGL")
endif()

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  find_package(PythonQt REQUIRED)
endif()

#
# See CTK/CMake/ctkMacroBuildLib.cmake for details
#

set(KIT_export_directive "CTK_VISUALIZATION_VTK_CORE_EXPORT")


# Source files
set(KIT_SRCS
  ctkVTKConnection.cpp
  ctkVTKConnection.h
  ctkVTKErrorLogMessageHandler.cpp
  ctkVTKErrorLogMessageHandler.h
  ctkVTKObject.h
  ctkVTKObjectEventsObserver.cpp
  ctkVTKObjectEventsObserver.h
  ctkVTKScalarsToColorsUtils.cpp
  ctkVTKScalarsToColorsUtils.h
  vtkLightBoxRendererManager.cpp
  vtkLightBoxRendererManager.h
  )

# Headers that should run through moc
set(KIT_MOC_SRCS
  ctkVTKConnection.h
  ctkVTKObjectEventsObserver.h
  )

# UI files
set(KIT_UI_FORMS
  )

# Resources
set(KIT_resources
  )

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  list(APPEND KIT_SRCS
    ctkVTKPythonQtWrapperFactory.cpp
    )
endif()

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  include_directories(
    ${PYTHON_INCLUDE_DIRS}
    ${PYTHONQT_INCLUDE_DIR}
    )
endif()

# Set VTK_LIBRARIES variable
if(${VTK_VERSION_MAJOR} GREATER 5)
  set(VTK_LIBRARIESx
    # a few of these have to be specified explicitly to workaround
    # underlinked libraries from the debian VTK package
    # see Debian bug #747436
    CommonCore
    CommonDataModel
    CommonSystem
    FiltersStatistics
    IOImage
    InteractionStyle
    RenderingAnnotation
    RenderingCore
    )
  if(TARGET vtkRendering${VTK_RENDERING_BACKEND})
    list(APPEND VTK_LIBRARIESx Rendering${VTK_RENDERING_BACKEND})
  endif()
  if(TARGET vtkIOMPIImage)
    list(APPEND VTK_LIBRARIESx IOMPIImage )
  endif()
  if(TARGET vtkRenderingMatplotlib)
    list(APPEND VTK_LIBRARIESx RenderingMatplotlib )
  endif()
  if(TARGET vtkTestingRendering)
    list(APPEND VTK_LIBRARIESx TestingRendering )
  endif()
  # With VTK > 6.2.0 (commit 4f7460a5), RenderingFreeTypeOpenGL has been removed.
  if(TARGET RenderingFreeTypeOpenGL)
    list(APPEND VTK_LIBRARIESx RenderingFreeTypeOpenGL)
  endif()
  if(TARGET RenderingFreeType )
    list(APPEND VTK_LIBRARIESx RenderingFreeType)
  endif()
  if (TARGET RenderingFreeTypeFontConfig AND UNIX AND NOT APPLE)
      list(APPEND VTK_LIBRARIESx
        RenderingFreeTypeFontConfig
        )
  endif()
else()
  set(VTK_LIBRARIESx
    vtkCommon
    vtkFiltering
    vtkRendering
    vtkHybrid
    )
endif()

if(CTK_LIB_Visualization/VTK/Widgets_USE_TRANSFER_FUNCTION_CHARTS)
  set(CTK_USE_CHARTS 1)
endif()

if(${CTK_USE_CHARTS})
  set(KIT_SRCS
    vtkDiscretizableColorTransferChart.cpp
    vtkDiscretizableColorTransferChart.h
    vtkDiscretizableColorTransferControlPointsItem.cpp
    vtkDiscretizableColorTransferControlPointsItem.h
    vtkScalarsToColorsContextItem.cpp
    vtkScalarsToColorsContextItem.h
    vtkScalarsToColorsHistogramChart.cpp
    vtkScalarsToColorsHistogramChart.h
    vtkScalarsToColorsPreviewChart.cpp
    vtkScalarsToColorsPreviewChart.h
    ${KIT_SRCS})
  if(${VTK_VERSION_MAJOR} GREATER 5)
    set(VTK_LIBRARIESx
      #ChartsCore
      ${VTK_LIBRARIESx})
  else()
    set(VTK_LIBRARIESx
      vtkCharts
      ${VTK_LIBRARIESx})
  endif()
  add_definitions(-DCTK_USE_CHARTS)
  set(VTK_LIBRARIESx
    #ViewsContext2D
	${VTK_LIBRARIESx})
  if(TARGET vtkRenderingContext${VTK_RENDERING_BACKEND})
    list(APPEND VTK_LIBRARIESx RenderingContext${VTK_RENDERING_BACKEND})
  endif()
endif()

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  ctkFunctionExtractOptimizedLibrary(PYTHON_LIBRARIES PYTHON_LIBRARY)
  list(APPEND VTK_LIBRARIESx ${PYTHON_LIBRARY} ${PYTHONQT_LIBRARIES})
  if(${VTK_VERSION_MAJOR} GREATER 5)
    list(APPEND VTK_LIBRARIESx vtkWrappingPythonCore)
  elseif(${VTK_VERSION_MAJOR}.${VTK_VERSION_MINOR} VERSION_GREATER 5.6)
    list(APPEND VTK_LIBRARIESx vtkPythonCore)
  else()
    list(APPEND VTK_LIBRARIESx vtkCommonPythonD)
  endif()
endif()

if(Q_WS_X11)
  # If the faster 'gold' linker is used, to avoid complaints about undefined symbol
  # 'XGetWindowAttributes', 'XCreateWindow', ..., let's link against X11 libraries.
  find_package(X11)
  list(APPEND VTK_LIBRARIESx ${X11_LIBRARIES})
endif()

# Target libraries - See CMake/ctkFunctionGetTargetLibraries.cmake
# The following macro will read the target libraries from the file 'target_libraries.cmake'
ctkFunctionGetTargetLibraries(KIT_target_libraries)

ctkMacroBuildLib(
  NAME ${PROJECT_NAME}
  EXPORT_DIRECTIVE ${KIT_export_directive}
  SRCS ${KIT_SRCS}
  MOC_SRCS ${KIT_MOC_SRCS}
  UI_FORMS ${KIT_UI_FORMS}
  TARGET_LIBRARIES ${KIT_target_libraries} ${VTK_LIBRARIES} ${PYTHONQT_LIBRARIES}
  RESOURCES ${KIT_resources}
  LIBRARY_TYPE ${CTK_LIBRARY_MODE}
  )

if(CTK_WRAP_PYTHONQT_LIGHT)
  ctkMacroBuildLibWrapper(
    TARGET ${PROJECT_NAME}
    SRCS ${KIT_SRCS}
    WRAPPER_LIBRARY_TYPE ${CTK_LIBRARY_MODE}
    )
endif()

  if (NOT VTK_VERSION VERSION_LESS "8.90.0")
    vtk_module_autoinit(
      TARGETS ${PROJECT_NAME}
      MODULES ${VTK_LIBRARIES}
      )
  endif()

# Plugins
#if(CTK_BUILD_QTDESIGNER_PLUGINS)
#  add_subdirectory(Plugins)
#endif()

# Testing
if(BUILD_TESTING)
  add_subdirectory(Testing)
endif()
