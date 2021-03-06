option ( PYTHON_BINDINGS "Build python bindings via swig" ON )
option ( LINK_PYTHON_LIBRARY "Link against python libraries" ON )

if ( PYTHON_BINDINGS )
	find_package ( SWIG )
	find_package ( PythonLibs )
    find_package ( PythonInterp )
endif ()

find_program(PYTHON "python")

if ( PYTHON AND SWIG_FOUND AND PYTHONLIBS_FOUND )
	include ( UseSWIG )
  	include_directories ( BEFORE ${CMAKE_SOURCE_DIR}/include/lsh )
  	include_directories ( ${PYTHON_INCLUDE_DIRS} )
  	link_directories ( ${CMAKE_CURRENT_BINARY_DIR}/src/lsh )

    swig_add_module(lsh_api python lsh_api.i)
    swig_link_libraries(lsh_api ${PYTHON_LIBRARIES} lsh)
    
    file(GLOB DEPS "*.py")
	file(COPY ${DEPS} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})


	execute_process ( COMMAND ${PYTHON_EXECUTABLE} -c "from distutils import sysconfig; print( sysconfig.get_python_lib( plat_specific=True, prefix='${CMAKE_INSTALL_PREFIX}' ) )"
             OUTPUT_VARIABLE _ABS_PYTHON_MODULE_PATH
             OUTPUT_STRIP_TRAILING_WHITESPACE )
  
    get_filename_component ( _ABS_PYTHON_MODULE_PATH ${_ABS_PYTHON_MODULE_PATH} ABSOLUTE )
    file ( RELATIVE_PATH _REL_PYTHON_MODULE_PATH ${CMAKE_INSTALL_PREFIX} ${_ABS_PYTHON_MODULE_PATH} )
   
    set ( PYTHON_MODULE_PATH
         ${_REL_PYTHON_MODULE_PATH}
    )

	INSTALL ( FILES ${CMAKE_BINARY_DIR}/python/lsh/_lsh_api.so ${CMAKE_BINARY_DIR}/python/lsh/lsh_api.py ${CMAKE_BINARY_DIR}/python/lsh/__init__.py ${CMAKE_BINARY_DIR}/python/lsh/lsh.py DESTINATION ${PYTHON_MODULE_PATH}/lsh )
	
else ()
  message(STATUS "Not building python bindings")
endif ()
