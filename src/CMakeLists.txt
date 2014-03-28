# Copyright 2010, 2011, Florent Lamiraux, Thomas Moulard, JRL, CNRS/AIST
#
# This file is part of dynamic-graph-python.
# dynamic-graph-python is free software: you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of
# the License, or (at your option) any later version.
#
# dynamic-graph-python is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Lesser Public License for more details.  You should have
# received a copy of the GNU Lesser General Public License along with
# dynamic-graph-python. If not, see <http://www.gnu.org/licenses/>.

#
# Python
#
set(Python_ADDITIONAL_VERSIONS 2.7 2.6 2.5 2.4 2.3 2.2 2.1 2.0)
INCLUDE(${SOT_CMAKE_DIR}/python.cmake)
FINDPYTHON()

INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})
LINK_DIRECTORIES(${Boost_LIBRARY_DIRS} ${PYTHON_LIBRARY_DIRS})

INCLUDE(${SOT_CMAKE_DIR}/python.cmake)
#
#
#  Python interpreter
#
#
SET(LIBRARY_NAME ${PROJECT_NAME})
ADD_LIBRARY(${LIBRARY_NAME}
  SHARED
  interpreter.cc)

TARGET_LINK_LIBRARIES(${LIBRARY_NAME}
  ${PYTHON_LIBRARY})

TARGET_LINK_LIBRARIES(${LIBRARY_NAME} ${Boost_LIBRARIES} ${catkin_LIBRARIES} dl)

SET_TARGET_PROPERTIES(${LIBRARY_NAME} PROPERTIES SOVERSION ${PROJECT_VERSION})
#PKG_CONFIG_USE_DEPENDENCY(${LIBRARY_NAME} dynamic-graph)

INSTALL(TARGETS ${LIBRARY_NAME}
  DESTINATION ${CMAKE_INSTALL_LIBDIR})

SET(EXECUTABLE_NAME dg-python)
ADD_EXECUTABLE(${EXECUTABLE_NAME} dg-python.cc)
TARGET_LINK_LIBRARIES(${EXECUTABLE_NAME}
  ${LIBRARY_NAME}
  )
INSTALL(TARGETS dg-python DESTINATION ${CMAKE_INSTALL_BINDIR})

#
#
#  Python bindings
#
#

SET(PYTHON_MODULE ${PROJECT_NAME}_wrap)

ADD_LIBRARY(${PYTHON_MODULE}
	MODULE
	exception-python.cc
	convert-dg-to-py.cc
	dynamic-graph-py.cc
	signal-base-py.cc
	entity-py.cc
	factory-py.cc
	pool-py.cc
	signal-caster-py.cc
)

# Remove prefix lib
SET_TARGET_PROPERTIES(${PYTHON_MODULE}
	PROPERTIES PREFIX ""
  LIBRARY_OUTPUT_DIRECTORY ${DYNAMIC_GRAPH_PLUGINDIR}
  OUTPUT_NAME "wrap")

TARGET_LINK_LIBRARIES(${PYTHON_MODULE} ${PYTHON_LIBRARY} ${catkin_LIBRARIES})

CONFIGURE_FILE(link-to-python.hh.in ${CATKIN_DEVEL_PREFIX}/include/link-to-python.hh)
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH}
  ${CMAKE_CURRENT_BINARY_DIR})

#
# Installation
#

SET(PYTHON_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/${PYTHON_SITELIB}/dynamic_graph)

INSTALL(TARGETS ${PYTHON_MODULE}
	       DESTINATION
	       ${PYTHON_INSTALL_DIR})

SET (PYTHON_SOURCES
    __init__.py
    attrpath.py
    entity.py
    signal_base.py
    matlab.py
    script_shortcuts.py
    tools.py
)

FOREACH (SOURCE ${PYTHON_SOURCES})
  PYTHON_INSTALL_ON_SITE(dynamic_graph ${SOURCE} )
ENDFOREACH (SOURCE)

DYNAMIC_GRAPH_PYTHON_MODULE("tracer" "" tracer-wrap)
target_link_libraries(tracer-wrap ${LIBRARY_NAME} ${catkin_LIBRARIES})

# TracerRealTime plugin
DYNAMIC_GRAPH_PYTHON_MODULE("tracer_real_time" "" tracer_real_time-wrap)
target_link_libraries(tracer_real_time-wrap ${LIBRARY_NAME} ${catkin_LIBRARIES})


