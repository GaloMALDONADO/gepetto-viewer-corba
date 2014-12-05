# Copyright (c) 2014 LAAS-CNRS
# Author: Mathieu Geisert
#
# This file is part of SceneViewer-corba.
# SceneViewer-corba is free software: you can redistribute it
# and/or modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation, either version
# 3 of the License, or (at your option) any later version.
#
# SceneViewer-corba is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty
# of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Lesser Public License for more details.  You should have
# received a copy of the GNU Lesser General Public License along with
# hpp-corbaserver.  If not, see
# <http://www.gnu.org/licenses/>.

SET(LIBRARY_NAME ${PROJECT_NAME})
SET(IDL_SOURCES
  graphicalinterface
  )

OMNIIDL_INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/idl)
INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR}/src)
FOREACH(IDL ${IDL_SOURCES})
  GENERATE_IDL_FILE (${IDL}
    ${CMAKE_SOURCE_DIR}/idl/Graphics/corbaserver)
ENDFOREACH()

ADD_LIBRARY(${LIBRARY_NAME}
  SHARED
  graphicalinterface.hh
  graphicalinterfaceSK.cc
  GraphicalInterface.impl.cpp
  GraphicalInterface.impl.hh
  server.hh
  server.cc
  server-private.cc
  server-private.hh
  client-cpp.cc
  client-cpp.hh
  se3.hpp
  ${CMAKE_CURRENT_BINARY_DIR}/Graphics/corbaserver/graphicalinterface_idl.py
)

PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} omniORB4)
PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} openscenegraph)
PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} openthreads)
PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} SceneViewer)
PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} urdfdom)
#PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} eigen3)
#PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} boost)

INSTALL(TARGETS ${LIBRARY_NAME} DESTINATION lib)

# search for python
FINDPYTHON()

FOREACH (IDL ${IDL_SOURCES})
  ADD_CUSTOM_COMMAND(
    OUTPUT
    ${CMAKE_CURRENT_BINARY_DIR}/Graphics/corbaserver/${IDL}_idl.py
    COMMAND ${OMNIIDL}
    ARGS -bpython -Wbpackage="" -I${CMAKE_SOURCE_DIR}/idl
    ${CMAKE_SOURCE_DIR}/idl/Graphics/corbaserver/${IDL}.idl
    MAIN_DEPENDENCY
    ${CMAKE_SOURCE_DIR}/idl/Graphics/corbaserver/${IDL}.idl
    )
  SET_SOURCE_FILES_PROPERTIES(
    ${CMAKE_CURRENT_BINARY_DIR}/${IDL}_idl.py
    GENERATED=ON
    )
  INSTALL(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${IDL}_idl.py
    DESTINATION ${PYTHON_SITELIB}/Graphics/corbaserver
    )
ENDFOREACH()

INSTALL(
  FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/Graphics/__init__.py
  ${CMAKE_CURRENT_SOURCE_DIR}/Graphics/quaternion.py
  DESTINATION ${PYTHON_SITELIB}/Graphics
  )
INSTALL(
  FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/Graphics/corbaserver/__init__.py
  DESTINATION ${PYTHON_SITELIB}/Graphics/corbaserver
  )
INSTALL(
  FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/Graphics/corbaserver/client.py
  DESTINATION ${PYTHON_SITELIB}/Graphics/corbaserver
  )
INSTALL(
  FILES
  ${CMAKE_SOURCE_DIR}/idl/Graphics/corbaserver/graphicalinterface.idl
  DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/idl/Graphics/corbaserver
)

# Standalone corba server
ADD_EXECUTABLE (SceneViewercorbaserver SceneViewer-corbaserver.cc)
TARGET_LINK_LIBRARIES(SceneViewercorbaserver ${LIBRARY_NAME})
TARGET_LINK_LIBRARIES(SceneViewercorbaserver SceneViewer)
TARGET_LINK_LIBRARIES(SceneViewercorbaserver urdfdom_model)
TARGET_LINK_LIBRARIES(SceneViewercorbaserver ${Boost_LIBRARIES})


ADD_EXECUTABLE (clientTest ../test/test-client-cpp.cc)
TARGET_LINK_LIBRARIES(clientTest ${LIBRARY_NAME})
TARGET_LINK_LIBRARIES(clientTest SceneViewer)
TARGET_LINK_LIBRARIES(clientTest urdfdom_model)
TARGET_LINK_LIBRARIES(clientTest ${Boost_LIBRARIES})
#TARGET_LINK_LIBRARIES(clientTest eigen3)

INSTALL (TARGETS SceneViewercorbaserver DESTINATION bin)
INSTALL (TARGETS clientTest DESTINATION bin)