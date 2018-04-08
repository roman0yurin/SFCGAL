file( GLOB SFCGAL_BENCHS_SOURCES *.cpp )
add_executable( bench-sfcgal ${SFCGAL_BENCHS_SOURCES} )
target_link_libraries( bench-sfcgal
       sfcgal
       ${Boost_LIBRARIES}
       ${CGAL_LIBRARIES}
)
if( SFCGAL_WITH_MPFR )
	target_link_libraries( bench-sfcgal ${MPFR_LIBRARIES} ${GMP_LIBRARIES} )
endif()
set_target_properties( bench-sfcgal PROPERTIES DEBUG_POSTFIX "d" )
install( TARGETS bench-sfcgal DESTINATION bin )

