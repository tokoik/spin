cmake_minimum_required(VERSION 3.22)

project(spin LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
  add_compile_options("/utf-8")
endif()

set(SOURCES
  main.cpp
)

set(FETCHCONTENT_BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/libs")

if(APPLE)
  add_executable(${PROJECT_NAME} MACOSX_BUNDLE ${SOURCES})

  find_library(GLUT_LIBRARY GLUT)
  find_library(OPENGL_LIBRARY OpenGL)
  target_link_libraries(${PROJECT_NAME} PRIVATE ${GLUT_LIBRARY} ${OPENGL_LIBRARY})

  set_target_properties(${PROJECT_NAME} PROPERTIES
    MACOSX_BUNDLE TRUE
    MACOSX_BUNDLE_GUI_IDENTIFIER "com.tokoik.${PROJECT_NAME}"
    MACOSX_BUNDLE_BUNDLE_NAME "${PROJECT_NAME}"
    MACOSX_BUNDLE_BUNDLE_VERSION "1.0"
    MACOSX_BUNDLE_SHORT_VERSION_STRING "1.0"
    XCODE_GENERATE_SCHEME ON
    XCODE_SCHEME_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
  )
  set_property(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" PROPERTY XCODE_GENERATE_SCHEME ON)

elseif(WIN32)
  add_executable(${PROJECT_NAME} ${SOURCES})

  include(FetchContent)
  FetchContent_Declare(
    freeglut
    GIT_REPOSITORY https://github.com/FreeGLUTProject/freeglut.git
    GIT_TAG master
  )
  set(FREEGLUT_BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
  set(FREEGLUT_BUILD_DEMOS OFF CACHE BOOL "" FORCE)
  FetchContent_MakeAvailable(freeglut)

  find_package(OpenGL REQUIRED)

  target_include_directories(${PROJECT_NAME} PRIVATE
    "${freeglut_SOURCE_DIR}/include"
  )

  target_link_libraries(${PROJECT_NAME} PRIVATE freeglut_static OpenGL::GL OpenGL::GLU)
  target_compile_definitions(${PROJECT_NAME} PRIVATE FREEGLUT_STATIC)

  set_property(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
  set_target_properties(${PROJECT_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "$<TARGET_FILE_DIR:${PROJECT_NAME}>")

else()
  # Linux
  add_executable(${PROJECT_NAME} ${SOURCES})

  set(OpenGL_GL_PREFERENCE "LEGACY")
  find_package(OpenGL REQUIRED)
  find_package(GLUT REQUIRED)

  target_link_libraries(${PROJECT_NAME}
    PRIVATE
    ${OPENGL_LIBRARIES}
    GLUT::GLUT
    m
  )
endif()
