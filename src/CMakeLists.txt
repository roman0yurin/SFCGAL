file( GLOB_RECURSE SFCGAL_SOURCES "*.cpp" )
file( GLOB_RECURSE SFCGAL_HEADERS "*.h" )

file( GLOB_RECURSE SFCGAL_HEADERS_COPIED RELATIVE ${CMAKE_SFCGAL_SOURCE_DIR}/src "*.h" )
add_custom_target(copy)
foreach (header ${SFCGAL_HEADERS_COPIED})
    add_custom_command(TARGET copy
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_SFCGAL_SOURCE_DIR}/src/${header} ${CMAKE_SFCGAL_BINARY_DIR}/include/SFCGAL/${header} DEPENDS ${CMAKE_SFCGAL_SOURCE_DIR}/src/${header})
endforeach()

set( SFCGAL_OSG_HEADERS
  ${CMAKE_SFCGAL_SOURCE_DIR}/src/io/osg.h
  ${CMAKE_SFCGAL_SOURCE_DIR}/src/detail/io/OsgFactory.h )
set( SFCGAL_OSG_SOURCES
  ${CMAKE_SFCGAL_SOURCE_DIR}/src/io/osg.cpp
  ${CMAKE_SFCGAL_SOURCE_DIR}/src/detail/io/OsgFactory.cpp )
message( STATUS "removing OSG dependencies from the library")
list(REMOVE_ITEM SFCGAL_HEADERS ${SFCGAL_OSG_HEADERS})
list(REMOVE_ITEM SFCGAL_SOURCES ${SFCGAL_OSG_SOURCES})

# Используем статическую линковку
if( SFCGAL_USE_STATIC_LIBS )
	add_definitions( "-fPIC" )
	add_library(
		sfcgal STATIC
		${SFCGAL_HEADERS}
		${SFCGAL_SOURCES}
	)
else()
  add_definitions( "-DSFCGAL_BUILD_SHARED" )
  add_library(
    sfcgal SHARED
    ${SFCGAL_HEADERS}
    ${SFCGAL_SOURCES}
    )
endif()

add_dependencies( sfcgal copy CGAL)

# set VERSION and SOVERSION
set_target_properties( sfcgal PROPERTIES VERSION ${SFCGAL_VERSION}
                                         SOVERSION ${SFCGAL_VERSION_MAJOR} )

target_link_libraries( sfcgal ${CGAL_LIBRARIES} CGAL_ImageIO)
if( ${SFCGAL_WITH_MPFR} )
	target_link_libraries( sfcgal ${MPFR_LIBRARIES} )
endif( ${SFCGAL_WITH_MPFR} )

if( ${SFCGAL_WITH_GMP} )
	target_link_libraries( sfcgal ${GMP_LIBRARIES} )
endif( ${SFCGAL_WITH_GMP} )

target_link_libraries( sfcgal ${Boost_LIBRARIES} )

if ( ${Use_precompiled_headers} )
  if(PCHSupport_FOUND)
    # Add "-fPIC" for shared library build
    if( ${SFCGAL_USE_STATIC_LIBS} )
      set( pch_option "" )
    else()
      set( pch_option "-fPIC" )
    endif()
    add_precompiled_header(sfcgal
      include/CGAL/all.h
      ${pch_option}
      )
  endif()
endif()

# install library
install(
	TARGETS
	sfcgal
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	BUNDLE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# sfcgal-osg
if ( SFCGAL_WITH_OSG )
  if( SFCGAL_USE_STATIC_LIBS )
    add_definitions( "-fPIC" )
    add_library(
      sfcgal-osg
      ${SFCGAL_OSG_HEADERS}
      ${SFCGAL_OSG_SOURCES}
      )

  else()
    add_definitions( "-DSFCGAL_BUILD_SHARED" )
    add_library(
      sfcgal-osg SHARED
      ${SFCGAL_OSG_HEADERS}
      ${SFCGAL_OSG_SOURCES}
      )
  endif()
  target_link_libraries( sfcgal-osg
    sfcgal
    ${OPENSCENEGRAPH_LIBRARIES}
  )
  set_target_properties( sfcgal-osg PROPERTIES VERSION ${SFCGAL_VERSION}
                                         SOVERSION ${SFCGAL_VERSION_MAJOR} )
  install(
	TARGETS
	sfcgal-osg
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	BUNDLE DESTINATION ${CMAKE_INSTALL_LIBDIR}
   )
endif()
