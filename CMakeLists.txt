cmake_minimum_required(VERSION 3.1)
project(module_cosmicweb)

# ┌──────────────────────────────────────────────────────────────────┐
# │  Options                                                         │
# └──────────────────────────────────────────────────────────────────┘ 
option(OSPRAY_MODULE_COSMICWEB "Cosmic Web module" OFF)
option(OSPRAY_MODULE_COSMICWEB_WORKER "Cosmic Web rendering worker" OFF)
option(OSPRAY_MODULE_COSMICWEB_VIEWER "Cosmic Web client viewer" OFF)

if (OSPRAY_MODULE_COSMICWEB)

  # ┌────────────────────────────────────────────────────────────────┐
  # │  Configure CMake Globally                                      │
  # └────────────────────────────────────────────────────────────────┘   
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_MODULE_PATH 
    ${CMAKE_MODULE_PATH} 
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

  # ┌────────────────────────────────────────────────────────────────┐
  # │  Find Packages                                                 │
  # └────────────────────────────────────────────────────────────────┘   
  # Turbo-Jpeg
  find_package(TurboJpeg REQUIRED)
  # MPI
  if (OSPRAY_MODULE_COSMICWEB_WORKER)
    find_package(MPI REQUIRED)
    if (MPI_FOUND)
      add_library(MPI INTERFACE IMPORTED)
      set_target_properties(MPI PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "${MPI_CXX_INCLUDE_PATH}"
        INTERFACE_LINK_LIBRARIES "${MPI_CXX_LIBRARIES}")
      if (MPI_CXX_COMPILE_FLAGS)
        set_target_properties(MPI PROPERTIES 
          INTERFACE_COMPILE_FLAGS "${MPI_CXX_COMPILE_FLAGS}")
      endif (MPI_CXX_COMPILE_FLAGS)
      if (MPI_CXX_LINK_FLAGS)
        set_target_properties(MPI PROPERTIES 
          INTERFACE_LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
      endif (MPI_CXX_LINK_FLAGS)
    endif()
  endif()
  # GLFW
  if (OSPRAY_MODULE_COSMICWEB_VIEWER)
    if (NOT TARGET glfw)
      find_package(glfw3 REQUIRED)
    endif()
    target_include_directories(glfw INTERFACE 
      "$<BUILD_INTERFACE:${GLFW_INCLUDE_DIRS};>")
  endif ()

  # ┌────────────────────────────────────────────────────────────────┐
  # │  Util Library                                                  │
  # └────────────────────────────────────────────────────────────────┘     
  add_library(cosmicweb_app_util
    util.cpp
    image_util.cpp
    client_server.cpp)
  target_link_libraries(cosmicweb_app_util PUBLIC
    ospray
    ospray_common
    TurboJpeg)
  target_include_directories(cosmicweb_app_util PUBLIC
    "$<BUILD_INTERFACE:"
    "${CMAKE_SOURCE_DIR}/apps/common;"
    "${CMAKE_SOURCE_DIR}/apps/exampleViewer;"
    "${CMAKE_SOURCE_DIR}/apps/exampleViewer/common/imgui;"
    ">")

  # ┌────────────────────────────────────────────────────────────────┐
  # │  Render Worker                                                 │
  # └────────────────────────────────────────────────────────────────┘     
  if (OSPRAY_MODULE_COSMICWEB_WORKER)
    ospray_create_application(cosmicweb_render_worker
      cosmicweb_render_worker.cpp
      LINK
      # local
      cosmicweb_app_util
      # ospray
      ospray
      ospray_common
      ospray_mpi_common
      # extra
      TurboJpeg
      MPI)
  endif()

  # ┌────────────────────────────────────────────────────────────────┐
  # │  Client Viewer                                                 │
  # └────────────────────────────────────────────────────────────────┘     
  if (OSPRAY_MODULE_COSMICWEB_VIEWER)
    ospray_create_application(cosmicweb_viewer
      cosmicweb_viewer.cpp
      arcball.cpp
      LINK
      # local
      cosmicweb_app_util
      # ospray
      ospray
      ospray_common
      ospray_sg
      ospray_imgui
      ospray_imgui3d_sg
      # extra
      TurboJpeg
      glfw)
  endif()  

endif()

