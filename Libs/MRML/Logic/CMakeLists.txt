project(MRMLLogic)

#-----------------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8.4)
#-----------------------------------------------------------------------------

#-----------------------------------------------------------------------------
# See http://cmake.org/cmake/help/cmake-2-8-docs.html#section_Policies for details
#-----------------------------------------------------------------------------
if(POLICY CMP0017)
  cmake_policy(SET CMP0017 OLD)
endif()

if(NOT DEFINED BUILD_SHARED_LIBS)
  option(BUILD_SHARED_LIBS "Build with shared libraries." ON)
endif()

# --------------------------------------------------------------------------
# Include dirs
# --------------------------------------------------------------------------
set(include_dirs
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  ${MRMLCore_INCLUDE_DIRS}
  ${vtkTeem_INCLUDE_DIRS}
  ${RemoteIO_INCLUDE_DIRS}
  ${LibArchive_INCLUDE_DIR}
  )
include_directories(${include_dirs})

# --------------------------------------------------------------------------
# Configure headers
# --------------------------------------------------------------------------
set(configure_header_file vtkMRMLLogicConfigure.h)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/${configure_header_file}.in
  ${CMAKE_CURRENT_BINARY_DIR}/${configure_header_file}
  )

# --------------------------------------------------------------------------
# Install headers
# --------------------------------------------------------------------------
if(NOT DEFINED ${PROJECT_NAME}_INSTALL_NO_DEVELOPMENT)
  set(${PROJECT_NAME}_INSTALL_NO_DEVELOPMENT ON)
endif()
if(NOT ${PROJECT_NAME}_INSTALL_NO_DEVELOPMENT)
  file(GLOB headers "${CMAKE_CURRENT_SOURCE_DIR}/*.h")
  install(
    FILES ${headers} ${CMAKE_CURRENT_BINARY_DIR}/${configure_header_file}
    DESTINATION include/${PROJECT_NAME} COMPONENT Development)
endif()

# --------------------------------------------------------------------------
# Sources
# --------------------------------------------------------------------------
set(MRMLLogic_SRCS
  vtkMRMLAbstractLogic.cxx
  vtkMRMLApplicationLogic.cxx
  vtkMRMLColorLogic.cxx
  vtkMRMLDisplayableHierarchyLogic.cxx
  vtkMRMLRemoteIOLogic.cxx
  vtkMRMLLayoutLogic.cxx
  vtkMRMLModelHierarchyLogic.cxx
  vtkMRMLSliceLayerLogic.cxx
  vtkMRMLSliceLogic.cxx
  vtkMRMLSliceLinkLogic.cxx

  # slicer's vtk extensions (filters)
  vtkImageLabelOutline.cxx
  vtkImageNeighborhoodFilter.cxx
  vtkImageLinearReslice.cxx
  vtkImageResliceMask.cxx
  vtkArchive.cxx
  )

# vtkArchive is not really a vtk class
set_source_files_properties(vtkArchive.cxx
  WRAP_EXCLUDE
  )

# set hints for tcl and python
set(VTK_WRAP_HINTS ${CMAKE_CURRENT_SOURCE_DIR}/Wrapping/Tcl/hints)

# --------------------------------------------------------------------------
# Build library
# --------------------------------------------------------------------------
set(lib_name ${PROJECT_NAME})

set(srcs ${MRMLLogic_SRCS})
add_library(${lib_name} ${srcs})

set(VTK_LIBRARIES
  #Common # Provided by MRMLCore, vtkTeem, RemoteIO
  #Filtering # Provided by MRMLCore, vtkTeem
  #Graphics # Provided by MRMLCore, vtkTeem
  #Imaging # Provided by MRMLCore, vtkTeem
  #Rendering # Provided by MRMLCore
  "" # Since no direct dependency to VTK libraries are required, let's explicitly set
     # the variable to an empty string to avoid complaints from vtkMacroKitPythonWraps.cmake.
  )
set(libs
  MRMLCore
  vtkTeem
  RemoteIO
  ${VTK_LIBRARIES}
  ${LibArchive_LIBRARY}
  )
target_link_libraries(${lib_name} ${libs})

# Apply user-defined properties to the library target.
if(Slicer_LIBRARY_PROPERTIES)
  set_target_properties(${lib_name} PROPERTIES ${Slicer_LIBRARY_PROPERTIES})
endif()

# --------------------------------------------------------------------------
# Export target
# --------------------------------------------------------------------------
if(NOT DEFINED ${PROJECT_NAME}_EXPORT_FILE)
  set(${PROJECT_NAME}_EXPORT_FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake)
endif()
export(TARGETS ${lib_name} APPEND FILE ${${PROJECT_NAME}_EXPORT_FILE})

# --------------------------------------------------------------------------
# Install library
# --------------------------------------------------------------------------
if(NOT DEFINED ${PROJECT_NAME}_INSTALL_BIN_DIR)
  set(${PROJECT_NAME}_INSTALL_BIN_DIR bin)
endif()
if(NOT DEFINED ${PROJECT_NAME}_INSTALL_LIB_DIR)
  set(${PROJECT_NAME}_INSTALL_LIB_DIR lib/${PROJECT_NAME})
endif()

install(TARGETS ${lib_name}
  RUNTIME DESTINATION ${${PROJECT_NAME}_INSTALL_BIN_DIR} COMPONENT RuntimeLibraries
  LIBRARY DESTINATION ${${PROJECT_NAME}_INSTALL_LIB_DIR} COMPONENT RuntimeLibraries
  ARCHIVE DESTINATION ${${PROJECT_NAME}_INSTALL_LIB_DIR} COMPONENT Development
  )

# --------------------------------------------------------------------------
# Python Wrapping
# --------------------------------------------------------------------------
if(VTK_WRAP_PYTHON)
  vtkMacroKitPythonWrap(
    KIT_NAME ${lib_name}
    KIT_SRCS ${MRMLLogic_SRCS}
    KIT_PYTHON_LIBRARIES MRMLCorePythonD
    KIT_INSTALL_BIN_DIR ${${PROJECT_NAME}_INSTALL_BIN_DIR}
    KIT_INSTALL_LIB_DIR ${${PROJECT_NAME}_INSTALL_LIB_DIR}
    )
  # Export target
  export(TARGETS ${lib_name}Python ${lib_name}PythonD APPEND FILE ${${PROJECT_NAME}_EXPORT_FILE})
endif()

# --------------------------------------------------------------------------
# Testing
# --------------------------------------------------------------------------
if(BUILD_TESTING)
  add_subdirectory(Testing)
endif()

# --------------------------------------------------------------------------
# Set INCLUDE_DIRS variable
# --------------------------------------------------------------------------
set(${PROJECT_NAME}_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}
  CACHE INTERNAL "${PROJECT_NAME} include dirs" FORCE)
