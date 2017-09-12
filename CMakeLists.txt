cmake_minimum_required( VERSION 3.1.0 )

project( libuv C )


SET( CMAKE_INSTALL_PREFIX "" CACHE PATH "Don't want an install target" FORCE)

# We do things my submodules.
set( libuv_TRUE_SOURCE_DIR "${libuv_SOURCE_DIR}/libuv" )

set( libuv_INCLUDE_DIR "${libuv_TRUE_SOURCE_DIR}/include" )
set( libuv_SOURCES_DIR "${libuv_TRUE_SOURCE_DIR}/src" )

set( libuv_SOURCES "" )
set( libuv_COMPILE_FLAGS "" )

set( libuv_LINK "" )
set( libuv_DEFINES "" )

set( libuv_INCLUDE_DIR_PRIVATE "" )
set( libuv_LINK_PRIVATE "" )
set( libuv_DEFINES_PRIVATE "" )
set( libuv_COMPILE_FLAGS_PRIVATE "" )

# The common sources
list( APPEND libuv_SOURCES
	${libuv_SOURCES_DIR}/fs-poll.c
	${libuv_SOURCES_DIR}/inet.c
	${libuv_SOURCES_DIR}/threadpool.c
	${libuv_SOURCES_DIR}/uv-common.c
	${libuv_SOURCES_DIR}/version.c
)

# source dir needs to be included in the list of private includes
list( APPEND libuv_INCLUDE_DIR_PRIVATE "${libuv_SOURCES_DIR}" )

# Now the platform stuff

if( ${CMAKE_SYSTEM_NAME} MATCHES "Windows" )
	
	# Win32 stuff
	list( APPEND libuv_DEFINES_PRIVATE 
		"-DWIN32_LEAN_AND_MEAN" 
		"-D_WIN32_WINNT=0x0600"
		"-D_CRT_SECURE_NO_WARNINGS"
		"-D_CRT_NONSTDC_NO_DEPRECATE"
	)
	
	# remove warnings
	list( APPEND libuv_COMPILE_FLAGS_PRIVATE 
		"/wd4244"
		"/wd4267"
	)
	
	# need to link with these
	list( APPEND libuv_LINK 
		ws2_32
	  	psapi
	  	iphlpapi
	  	shell32
	  	userenv
	)
	
	set( libuv_SOURCES_WIN_DIR "${libuv_SOURCES_DIR}/win" )

	list( APPEND libuv_INCLUDE_DIR_PRIVATE "${libuv_SOURCES_WIN_DIR}" )

	# Win32 sources
	list( APPEND libuv_SOURCES
		${libuv_SOURCES_WIN_DIR}/async.c
		${libuv_SOURCES_WIN_DIR}/core.c
		${libuv_SOURCES_WIN_DIR}/detect-wakeup.c
		${libuv_SOURCES_WIN_DIR}/dl.c
		${libuv_SOURCES_WIN_DIR}/error.c
		${libuv_SOURCES_WIN_DIR}/fs.c
		${libuv_SOURCES_WIN_DIR}/fs-event.c
		${libuv_SOURCES_WIN_DIR}/getaddrinfo.c
		${libuv_SOURCES_WIN_DIR}/getnameinfo.c
		${libuv_SOURCES_WIN_DIR}/handle.c
		${libuv_SOURCES_WIN_DIR}/loop-watcher.c
		${libuv_SOURCES_WIN_DIR}/pipe.c
		${libuv_SOURCES_WIN_DIR}/poll.c
		${libuv_SOURCES_WIN_DIR}/process-stdio.c
		${libuv_SOURCES_WIN_DIR}/process.c
		${libuv_SOURCES_WIN_DIR}/req.c
		${libuv_SOURCES_WIN_DIR}/signal.c
		${libuv_SOURCES_WIN_DIR}/snprintf.c
		${libuv_SOURCES_WIN_DIR}/stream.c
		${libuv_SOURCES_WIN_DIR}/tcp.c
		${libuv_SOURCES_WIN_DIR}/thread.c
		${libuv_SOURCES_WIN_DIR}/timer.c
		${libuv_SOURCES_WIN_DIR}/tty.c
		${libuv_SOURCES_WIN_DIR}/udp.c
		${libuv_SOURCES_WIN_DIR}/util.c
		${libuv_SOURCES_WIN_DIR}/winapi.c
		${libuv_SOURCES_WIN_DIR}/winsock.c
	)

else()

	# need pThreads
	FIND_PACKAGE(Threads REQUIRED)
	
	# Unix stuff
	list( APPEND libuv_DEFINES_PRIVATE 
		"-D_LARGEFILE_SOURCE"
		"-D_FILE_OFFSET_BITS=64"
		"-std=gnu89"  
		"-fvisibility=hidden"
	)

	list( APPEND libuv_LINK 
		Threads::Threads
		dl
	)
	
	set( libuv_SOURCES_UNIX_DIR "${libuv_SOURCES_DIR}/unix" )

	list( APPEND libuv_INCLUDE_DIR_PRIVATE "${libuv_SOURCES_UNIX_DIR}" )
	
	list( APPEND libuv_SOURCES
		${libuv_SOURCES_UNIX_DIR}/async.c
		${libuv_SOURCES_UNIX_DIR}/core.c
		${libuv_SOURCES_UNIX_DIR}/dl.c
		${libuv_SOURCES_UNIX_DIR}/fs.c
		${libuv_SOURCES_UNIX_DIR}/getaddrinfo.c
		${libuv_SOURCES_UNIX_DIR}/getnameinfo.c
		${libuv_SOURCES_UNIX_DIR}/loop-watcher.c
		${libuv_SOURCES_UNIX_DIR}/loop.c
		${libuv_SOURCES_UNIX_DIR}/pipe.c
		${libuv_SOURCES_UNIX_DIR}/poll.c
		${libuv_SOURCES_UNIX_DIR}/process.c
		${libuv_SOURCES_UNIX_DIR}/signal.c
		${libuv_SOURCES_UNIX_DIR}/stream.c
		${libuv_SOURCES_UNIX_DIR}/tcp.c
		${libuv_SOURCES_UNIX_DIR}/thread.c
		${libuv_SOURCES_UNIX_DIR}/timer.c
		${libuv_SOURCES_UNIX_DIR}/tty.c
		${libuv_SOURCES_UNIX_DIR}/udp.c
	)
	
	if( ${CMAKE_SYSTEM_NAME} MATCHES "Darwin" )
		
		list( APPEND libuv_DEFINES_PRIVATE 
			"-D_DARWIN_USE_64_BIT_INODE=1"
			"-D_DARWIN_UNLIMITED_SELECT=1"
		)
	
		list( APPEND libuv_SOURCES
		  	${libuv_SOURCES_UNIX_DIR}/darwin.c
		  	${libuv_SOURCES_UNIX_DIR}/darwin-proctitle.c
		  	${libuv_SOURCES_UNIX_DIR}/fsevents.c
		  	${libuv_SOURCES_UNIX_DIR}/kqueue.c
		  	${libuv_SOURCES_UNIX_DIR}/proctitle.c	  	
		)
		
	elseif( ${CMAKE_SYSTEM_NAME} MATCHES "Linux" )
	
		list( APPEND libuv_DEFINES_PRIVATE 
			"-D_GNU_SOURCE"
		)
	
		list( APPEND libuv_SOURCES
		  	${libuv_SOURCES_UNIX_DIR}/linux-core.c
		  	${libuv_SOURCES_UNIX_DIR}/linux-inotify.c
		  	${libuv_SOURCES_UNIX_DIR}/linux-syscalls.c
		  	${libuv_SOURCES_UNIX_DIR}/proctitle.c  	
		)		
		
		if( ANDROID_NDK )
			list( APPEND libuv_SOURCES
				${libuv_SOURCES_UNIX_DIR}/android-ifaddrs.c
				${libuv_SOURCES_UNIX_DIR}/pthread-fixes.c			
			)	
		endif()
		
	elseif( ${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD" )
	
		list( APPEND libuv_SOURCES 
		  	${libuv_SOURCES_UNIX_DIR}/kqueue.c
	  		${libuv_SOURCES_UNIX_DIR}/freebsd.c	
		)	
	
	endif()
	
endif()

add_library( libuv STATIC ${libuv_SOURCES} )

target_include_directories( libuv PRIVATE ${libuv_INCLUDE_DIR} ${libuv_INCLUDE_DIR_PRIVATE} )
target_compile_definitions( libuv PRIVATE ${libuv_DEFINES} ${libuv_DEFINES_PRIVATE} )
target_link_libraries( libuv PRIVATE ${libuv_LINK} ${libuv_LINK_PRIVATE} )
target_compile_options( libuv PRIVATE ${libuv_COMPILE_FLAGS} ${libuv_COMPILE_FLAGS_PRIVATE} )

target_include_directories( libuv INTERFACE $<BUILD_INTERFACE:${libuv_INCLUDE_DIR}> )
target_link_libraries( libuv INTERFACE $<BUILD_INTERFACE:${libuv_LINK}> )
target_compile_definitions( libuv INTERFACE ${libuv_DEFINES} )
target_compile_options( libuv INTERFACE ${libuv_COMPILE_FLAGS} )

