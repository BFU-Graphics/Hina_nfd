cmake_minimum_required(VERSION 3.24)

if(UNIX AND NOT APPLE)
	set(LINUX TRUE)
endif()

if(APPLE)

	project(nfd LANGUAGES C OBJC)

	add_library(nfd STATIC)
	target_sources(nfd
			PRIVATE
			nfd_cocoa.m
			nfd_common.c

			PUBLIC
			FILE_SET HEADERS
			BASE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../
			FILES
			nfd_common.h
			)

elseif(LINUX)

	project(nfd LANGUAGES C)
	
	find_package(PkgConfig REQUIRED)
    pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
    
	include_directories(${GTK3_INCLUDE_DIRS})
    link_directories(${GTK3_LIBRARY_DIRS})
    add_definitions(${GTK3_CFLAGS_OTHER})

	add_library(nfd STATIC)
	target_sources(nfd
			PRIVATE
			nfd_gtk.c
			nfd_common.c

			PUBLIC
			FILE_SET HEADERS
			BASE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../
			FILES
			nfd_common.h
			)

	if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
		set(CMAKE_C_FLAGS "-Wno-format-truncation ${CMAKE_C_FLAGS}")
	endif()

    target_link_directories(nfd PUBLIC ${GTK3_LIBRARY_DIRS})
    target_link_libraries(nfd ${GTK3_LIBRARIES})

else()
	project(nfd LANGUAGES C CXX)

	add_library(nfd STATIC)
	target_sources(nfd
			PRIVATE
			nfd_win.cpp
			nfd_common.c

			PUBLIC
			FILE_SET HEADERS
			BASE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../
			FILES
			nfd_common.h
			)
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /Z7")
endif()

# INSTALL TARGETS
INSTALL(
		TARGETS nfd
		CONFIGURATIONS Release
		FILE_SET HEADERS
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
)