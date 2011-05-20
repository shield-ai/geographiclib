# $Id$

if (MAINTAINER)
  set (MANDIR ${PROJECT_BINARY_DIR}/man)
else (MAINTAINER)
  set (MANDIR ${PROJECT_SOURCE_DIR}/man)
endif (MAINTAINER)

set (HTMLMAN)

foreach (TOOL ${TOOLS})
  set (HTMLMAN ${HTMLMAN} ${MANDIR}/${TOOL}.1.html)
endforeach (TOOL)

if (MAINTAINER)
  file (GLOB SOURCES ../src/*.cpp ../include/GeographicLib/*.hpp ../tools/*.cpp)
  file (GLOB EXTRA_FILES ../maxima/*.mac tmseries30.html geodseries30.html)
  add_custom_target (doc ALL
    DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/doxygen.log)
  add_dependencies (doc htmlman)
  add_custom_command (OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/doxygen.log
    DEPENDS Doxyfile Geographic.doc ${SOURCES} ${EXTRA_FILES} ${HTMLMAN}
    COMMAND rm -rf html && mkdir html
    COMMAND
      cp -p ${EXTRA_FILES} ${HTMLMAN} html/
    COMMAND
      sed \"s%\\.\\./%${PROJECT_SOURCE_DIR}/%g\"
        ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile | doxygen - > doxygen.log
    COMMENT "Generating html documentation tree")
  install (DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/html
    DESTINATION share/doc/GeographicLib)

  add_custom_target (distrib-doc)
  add_dependencies (distrib-doc doc distrib-man)
  add_custom_command (TARGET distrib-doc
    COMMAND rsync -a --delete html/ ${CMAKE_CURRENT_SOURCE_DIR}/html/
    COMMAND touch ${CMAKE_CURRENT_SOURCE_DIR}/html/index.html
    COMMENT "Installing html documentation tree in source tree")

else (MAINTAINER)
  install (DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/html
    DESTINATION share/doc/GeographicLib)
endif (MAINTAINER)