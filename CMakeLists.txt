# Main CMake file for the ZoneMinder project.
# Created by mastertheknife (Kfir Itzhak)
# The goal is to ease up the installation of zoneminder.
# Our current installation method (using autotools) is outdated, slow and breaks now and then.
# The CMake installation method will require no parameters at all, default should sufficient and reliable.
# It will be still possible to install ZoneMinder using autotools, they don't conflict with each other. The cmake way is a complete re-write (different syntax) and aims to be identical to the autotools way,
# by having options using the same name and leaving ZM totally unmodified, while providing exactly the same things that ZM expects (config.h, configuration in *.in files, etc).
# Configuration can be done in 4 ways:
# 	1) As a command line parameter, e.g. cmake -DCMAKE_VERBOSE_MAKEFILE=ON .
# 	2) By editing the cache file CMakeCache.txt (after it has been generated)
# 	3) Using cmake-gui
#	4) Providing cmake with an initial cache file with the -C option
#
# Possible configuration options: 
# 	ZM_RUNDIR	Location of transient process files, default: /var/run/zm
#       ZM_TMPDIR	Location of temporary files, default: /tmp/zm
#	ZM_LOGDIR 	Location of generated log files, default: /var/log/zm
#	ZM_WEBDIR	Location of the web files, default: <prefix>/${CMAKE_INSTALL_DATADIR}/zoneminder/www
#	ZM_CGIDIR	Location of the cgi-bin files, default: <prefix>/${CMAKE_INSTALL_LIBEXECDIR}/zoneminder/cgi-bin
#	ZM_CONTENTDIR	Location of dynamic content (events and images), default: /var/lib/zoneminder
#	ZM_DB_HOST	Hostname where ZoneMinder database located, default: localhost
#	ZM_DB_NAME	Name of ZoneMinder database, default: zm
#	ZM_DB_USER	Name of ZoneMinder database user, default: zmuser
#	ZM_DB_PASS	Password of ZoneMinder database user, default: zmpass
#	ZM_EXTRA_LIBS	A list of optional libraries, separated by space, e.g. libpng
#	ZM_NO_MMAP	Set to ON to not use mmap shared memory. Shouldn't be enabled unless you experience problems with the shared memory. default: OFF
#	ZM_WEB_USER	The user apache or the local web server runs on. Leave empty for automatic detection. If that fails, you can use this variable to force")
#	ZM_WEB_GROUP	The group apache or the local web server runs on, Leave empty to be the same as the web user")
# Not really needed:
#	ZM_NO_CRASHTRACE	Set to ON to skip crash trace code. Useful if zm_signal.cpp fails to compile. default: OFF
#	ZM_MYSQL_ENGINE		MySQL engine to use with database, default: InnoDB
#
# For changing the prefix (default /usr/local), please use CMAKE_INSTALL_PREFIX
# (e.g. -DCMAKE_INSTALL_PREFIX=/usr)
# CMAKE_INSTALL_PREFIX behaves like --prefix from autoconf
# Configuration options:
# *   For changing include search path, use CMAKE_INCLUDE_PATH
# *   For changing library search path, use CMAKE_LIBRARY_PATH
# Also see CMAKE_PREFIX_PATH for overriding both and some others.
#
# The DESTDIR environment variable is also supported.
# For more information about DESTDIR, see:
# *   http://www.gnu.org/prep/standards/html_node/DESTDIR.html
#
#
# To append to the CFLAGS and CXXFLAGS, please use the CFLAGS and CXXFLAGS environment variables (before running cmake)
# Or use the CMAKE_C_FLAGS and CMAKE_CXX_FLAGS configuration options.
# To replace the CFLAGS and CXXFLAGS entirely:
# *   For the Release build type: use CMAKE_C_FLAGS_RELEASE for the CFLAGS and CMAKE_CXX_FLAGS_RELEASE for the CXXFLAGS
# *   For the Debug build type: use CMAKE_C_FLAGS_DEBUG for the CFLAGS and CMAKE_CXX_FLAGS_DEBUG for the CXXFLAGS


cmake_minimum_required (VERSION 2.6)
project (ZoneMinder)
# ZoneMinder version
set(ZoneMinder_VERSION "1.26.3")
# Default build type. To change the build type, use the CMAKE_BUILD_TYPE configuration option.
if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type: Release or Debug" FORCE)
endif(NOT CMAKE_BUILD_TYPE)

# Can make finding problems easier when troubleshooting.
#set(CMAKE_VERBOSE_MAKEFILE ON)
#set(CMAKE_INSTALL_ALWAYS ON)

# To append to the CFLAGS and CXXFLAGS, please use the CFLAGS and CXXFLAGS environment variables or the CMAKE_C_FLAGS and CMAKE_CXX_FLAGS configuration options.
# These set for all build types. To set for specific build types:
# For the Release build type: use CMAKE_C_FLAGS_RELEASE for the CFLAGS and CMAKE_CXX_FLAGS_RELEASE for the CXXFLAGS
# For the Debug build type: use CMAKE_C_FLAGS_DEBUG for the CFLAGS and CMAKE_CXX_FLAGS_DEBUG for the CXXFLAGS
# Default CLFAGS and CXXFLAGS:
set(CMAKE_C_FLAGS_RELEASE "-Wall -D__STDC_CONSTANT_MACROS -O2")
set(CMAKE_CXX_FLAGS_RELEASE "-Wall -D__STDC_CONSTANT_MACROS -O2")
set(CMAKE_C_FLAGS_DEBUG "-Wall -D__STDC_CONSTANT_MACROS -g")
set(CMAKE_CXX_FLAGS_DEBUG "-Wall -D__STDC_CONSTANT_MACROS -g")
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
# Modules that we need:
include (GNUInstallDirs)
include (CheckIncludeFile)
include (CheckIncludeFiles)
include (CheckFunctionExists)
include (CheckPrototypeDefinition)
include (CheckTypeSize)
include (CheckStructHasMember)

# Default variables for some configuration options
mark_as_advanced(FORCE ZM_EXTRA_LIBS ZM_NO_CRASHTRACE)
set(ZM_RUNDIR "/var/run/zm" CACHE PATH "Location of transient process files, default: /var/run/zm")
set(ZM_TMPDIR "/tmp/zm" CACHE PATH "Location of temporary files, default: /tmp/zm")
set(ZM_LOGDIR "/var/log/zm" CACHE PATH "Location of generated log files, default: /var/log/zm")
set(ZM_WEBDIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/zoneminder/www" CACHE PATH "Location of the web files, default: <prefix>/${CMAKE_INSTALL_DATADIR}/zoneminder/www")
set(ZM_CGIDIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBEXECDIR}/zoneminder/cgi-bin" CACHE PATH "Location of the cgi-bin files, default: <prefix>/${CMAKE_INSTALL_LIBEXECDIR}/zoneminder/cgi-bin")
set(ZM_CONTENTDIR "/var/lib/zoneminder" CACHE PATH "Location of dynamic content (events and images), default: /var/lib/zoneminder")
# This one is not needed anymore. we do automatic detection of whats available, let ZM know and have ZM decide
#set(ZM_SSL_LIB "openssl" CACHE STRING "Library to use for ssl functions, default: openssl")
set(ZM_DB_HOST "localhost" CACHE STRING "Hostname where ZoneMinder database located, default: localhost")
set(ZM_DB_NAME "zm" CACHE STRING "Name of ZoneMinder database, default: zm")
set(ZM_DB_USER "zmuser" CACHE STRING "Name of ZoneMinder database user, default: zmuser")
set(ZM_DB_PASS "zmpass" CACHE STRING "Password of ZoneMinder database user, default: zmpass")
set(ZM_EXTRA_LIBS "" CACHE STRING "A list of optional libraries, separated by space, e.g. libpng")
set(ZM_NO_MMAP "OFF" CACHE BOOL "Set to ON to not use mmap shared memory. Shouldn't be enabled unless you experience problems with the shared memory. default: OFF")
set(ZM_WEB_USER "" CACHE STRING "The user apache or the local web server runs on. Leave empty for automatic detection. If that fails, you can use this variable to force")
set(ZM_WEB_GROUP "" CACHE STRING "The group apache or the local web server runs on, Leave empty to be the same as the web user")
# Not really needed:
set(ZM_NO_CRASHTRACE "OFF" CACHE BOOL "Set to ON to skip crash trace code. Useful if zm_signal.cpp fails to compile. default: OFF")
set(ZM_MYSQL_ENGINE "InnoDB" CACHE STRING "MySQL engine to use with database, default: InnoDB")

# Required for certain checks to work
set(CMAKE_EXTRA_INCLUDE_FILES ${CMAKE_EXTRA_INCLUDE_FILES} stdio.h stdlib.h math.h signal.h)
# Required for including headers from the this folder
include_directories("${CMAKE_CURRENT_BINARY_DIR}")
# This is required to enable searching in lib64 (if exists), do not change
set_property(GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS ON)

# Check for misc required stuff
check_include_file("linux/videodev.h" HAVE_LINUX_VIDEODEV_H)
check_include_file("linux/videodev2.h" HAVE_LINUX_VIDEODEV2_H)
check_include_file("execinfo.h" HAVE_EXECINFO_H)
check_include_file("sys/sendfile.h" HAVE_SYS_SENDFILE_H)
check_function_exists("sendfile" HAVE_SENDFILE)
check_function_exists("backtrace" HAVE_DECL_BACKTRACE)
check_function_exists("posix_memalign" HAVE_POSIX_MEMALIGN)
check_function_exists("strsignal" HAVE_STRSIGNAL)
check_prototype_definition("round" "double round (double x)" "0.0" "math.h" HAVE_DECL_ROUND)
check_type_size("siginfo_t" SIGINFO_T)
check_type_size("ucontext_t" UCONTEXT_T)
check_type_size("struct sigcontext" STRUCT_SIGCONTEXT)
check_struct_has_member("struct sigcontext" eip signal.h HAVE_STRUCT_SIGCONTEXT_EIP)

# *** LIBRARY CHECKS ***

# zlib
find_package(ZLIB)
if(ZLIB_FOUND)
	set(HAVE_LIBZLIB 1)
	list(APPEND ZM_BIN_LIBS ${ZLIB_LIBRARIES})
	include_directories(${ZLIB_INCLUDE_DIR})
	check_include_file("zlib.h" HAVE_ZLIB_H)
else(ZLIB_FOUND)
	message(FATAL_ERROR "zm requires zlib but it was not found on your system")
endif(ZLIB_FOUND)

# jpeg
find_package(JPEG)
if(JPEG_FOUND)
	set(HAVE_LIBJPEG 1)
	list(APPEND ZM_BIN_LIBS ${JPEG_LIBRARIES})
	include_directories(${JPEG_INCLUDE_DIR})
	check_include_files("stdio.h;jpeglib.h" HAVE_JPEGLIB_H)
	#link_directories(${JPEG_LIBRARY})
else(JPEG_FOUND)
	message(FATAL_ERROR "zm requires jpeg but it was not found on your system")
endif(JPEG_FOUND)

# OpenSSL
find_package(OpenSSL)
if(OPENSSL_FOUND)
	set(HAVE_LIBOPENSSL 1)
	list(APPEND ZM_BIN_LIBS ${OPENSSL_LIBRARIES})
	include_directories(${OPENSSL_INCLUDE_DIR})
	check_include_file("openssl/md5.h" HAVE_OPENSSL_MD5_H)
endif(OPENSSL_FOUND)
# crypto using find_library. 
# This library should be a part of the OpenSSL package but just need to be sure
# In any case, we still need to define HAVE_LIBCRYPTO for zm if its available
find_library(CRYPTO_LIBRARIES crypto)
if(CRYPTO_LIBRARIES)
	set(HAVE_LIBCRYPTO 1)
	list(APPEND ZM_BIN_LIBS ${CRYPTO_LIBRARIES})
endif(CRYPTO_LIBRARIES)

# pthread using find_library
find_library(PTHREAD_LIBRARIES pthread)
if(PTHREAD_LIBRARIES)
	set(HAVE_LIBPTHREAD 1)
	list(APPEND ZM_BIN_LIBS ${PTHREAD_LIBRARIES})
	check_include_file("pthread.h" HAVE_PTHREAD_H)
else(PTHREAD_LIBRARIES)
	message(FATAL_ERROR "zm requires pthread but it was not found on your system")
endif(PTHREAD_LIBRARIES)

# pcre using find_library
find_library(PCRE_LIBRARIES pcre)
if(PCRE_LIBRARIES)
	set(HAVE_LIBPCRE 1)
	list(APPEND ZM_BIN_LIBS ${PCRE_LIBRARIES})
	check_include_file("pcre/pcre.h" HAVE_PCRE_PCRE_H)
	check_include_file("pcre.h" HAVE_PCRE_H)
endif(PCRE_LIBRARIES)

# gcrypt using find_library
find_library(GCRYPT_LIBRARIES gcrypt)
if(GCRYPT_LIBRARIES)
	set(HAVE_LIBGCRYPT 1)
	list(APPEND ZM_BIN_LIBS ${GCRYPT_LIBRARIES})
	check_include_file("gcrypt.h" HAVE_GCRYPT_H)
else(GCRYPT_LIBRARIES)
	message(FATAL_ERROR "zm requires gcrypt but it was not found on your system")
endif(GCRYPT_LIBRARIES)

# gnutls using find_library
find_library(GNUTLS_LIBRARIES gnutls)
if(GNUTLS_LIBRARIES)
	set(HAVE_LIBGNUTLS 1)
	list(APPEND ZM_BIN_LIBS ${GNUTLS_LIBRARIES})
	check_include_file("gnutls/openssl.h" HAVE_GNUTLS_OPENSSL_H)
	check_include_file("gnutls/gnutls.h" HAVE_GNUTLS_GNUTLS_H)
else(GNUTLS_LIBRARIES)
	message(FATAL_ERROR "zm requires gnutls but it was not found on your system")
endif(GNUTLS_LIBRARIES)

# mysqlclient using find_library
find_library(MYSQLCLIENT_LIBRARIES mysqlclient PATH_SUFFIXES mysql)
if(MYSQLCLIENT_LIBRARIES)
	set(HAVE_LIBMYSQLCLIENT 1)
	list(APPEND ZM_BIN_LIBS ${MYSQLCLIENT_LIBRARIES})	
	check_include_file("mysql/mysql.h" HAVE_MYSQL_H)
else(MYSQLCLIENT_LIBRARIES)
	message(FATAL_ERROR "zm requires mysqlclient but it was not found on your system")
endif(MYSQLCLIENT_LIBRARIES)

# avformat using find_library
find_library(AVFORMAT_LIBRARIES avformat)
if(AVFORMAT_LIBRARIES)
	set(HAVE_LIBAVFORMAT 1)
	list(APPEND ZM_BIN_LIBS ${AVFORMAT_LIBRARIES})	
	check_include_file("libavformat/avformat.h" HAVE_LIBAVFORMAT_AVFORMAT_H)
endif(AVFORMAT_LIBRARIES)

# avcodec using find_library
find_library(AVCODEC_LIBRARIES avcodec)
if(AVCODEC_LIBRARIES)
	set(HAVE_LIBAVCODEC 1)
	list(APPEND ZM_BIN_LIBS ${AVCODEC_LIBRARIES})	
	check_include_file("libavcodec/avcodec.h" HAVE_LIBAVCODEC_AVCODEC_H)
endif(AVCODEC_LIBRARIES)

# avdevice using find_library
find_library(AVDEVICE_LIBRARIES avdevice)
if(AVDEVICE_LIBRARIES)
	set(HAVE_LIBAVDEVICE 1)
	list(APPEND ZM_BIN_LIBS ${AVDEVICE_LIBRARIES})	
	check_include_file("libavdevice/avdevice.h" HAVE_LIBAVDEVICE_AVDEVICE_H)
endif(AVDEVICE_LIBRARIES)

# avutil using find_library
find_library(AVUTIL_LIBRARIES avutil)
if(AVUTIL_LIBRARIES)
	set(HAVE_LIBAVUTIL 1)
	list(APPEND ZM_BIN_LIBS ${AVUTIL_LIBRARIES})	
	check_include_file("libavutil/avutil.h" HAVE_LIBAVUTIL_AVUTIL_H)
	check_include_file("libavutil/mathematics.h" HAVE_LIBAVUTIL_MATHEMATICS_H)
endif(AVUTIL_LIBRARIES)

# swscale using find_library
find_library(SWSCALE_LIBRARIES swscale)
if(SWSCALE_LIBRARIES)
	set(HAVE_LIBSWSCALE 1)
	list(APPEND ZM_BIN_LIBS ${SWSCALE_LIBRARIES})	
	check_include_file("libswscale/swscale.h" HAVE_LIBSWSCALE_SWSCALE_H)
endif(SWSCALE_LIBRARIES)

# *** END OF LIBRARY CHECKS ***


# Check for absolutely required headers:
if(NOT HAVE_MYSQL_H)
	message(FATAL_ERROR "zm requires MySQL headers - check that MySQL development packages are installed")
endif(NOT HAVE_MYSQL_H)
if(NOT HAVE_JPEGLIB_H)
	message(WARNING " zm requires libjpeg headers - check that libjpeg development packages are installed")
endif(NOT HAVE_JPEGLIB_H)
if(NOT HAVE_PTHREAD_H)
	message(WARNING " zm requires pthread headers - check that pthread development packages are installed")
endif(NOT HAVE_PTHREAD_H)

# Check for V4L header files and enable ZM_HAS_V4L, ZM_HAS_V4L1, ZM_HAS_V4L2 accordingly
# Setting to zeros first is required because ZM uses #define for these
set(ZM_HAS_V4L 0)
set(ZM_HAS_V4L1 0)
set(ZM_HAS_V4L2 0)
if(HAVE_LINUX_VIDEODEV_H)
	set(ZM_HAS_V4L 1)
	set(ZM_HAS_V4L1 1)
endif(HAVE_LINUX_VIDEODEV_H)
if(HAVE_LINUX_VIDEODEV2_H)
	set(ZM_HAS_V4L 1)
	set(ZM_HAS_V4L2 1)
endif(HAVE_LINUX_VIDEODEV2_H)
if((NOT HAVE_LINUX_VIDEODEV_H) AND (NOT HAVE_LINUX_VIDEODEV2_H))
	message(AUTHOR_WARNING " Video 4 Linux headers weren't found - Analog and USB camera support will not be available")
endif((NOT HAVE_LINUX_VIDEODEV_H) AND (NOT HAVE_LINUX_VIDEODEV2_H))
# Check for PCRE and enable ZM_PCRE accordingly
set(ZM_PCRE 0)
if(HAVE_LIBPCRE AND (HAVE_PCRE_PCRE_H OR HAVE_PCRE_H))
	set(ZM_PCRE 1)
endif(HAVE_LIBPCRE AND (HAVE_PCRE_PCRE_H OR HAVE_PCRE_H))


# Check for authenication functions
if(HAVE_OPENSSL_MD5_H)
	check_prototype_definition(MD5 "unsigned char *MD5(const unsigned char *d, size_t n, unsigned char *md)" "NULL" "openssl/md5.h" HAVE_DECL_MD5)
endif(HAVE_OPENSSL_MD5_H)
if(HAVE_GNUTLS_OPENSSL_H)
	check_prototype_definition(MD5 "unsigned char *MD5 (const unsigned char *buf, unsigned long len, unsigned char *md)" "NULL" "gnutls/openssl.h" HAVE_DECL_MD5)
endif(HAVE_GNUTLS_OPENSSL_H)
if(HAVE_GNUTLS_GNUTLS_H)
	check_prototype_definition(gnutls_fingerprint "int gnutls_fingerprint (gnutls_digest_algorithm_t algo, const gnutls_datum_t * data, void *result, size_t * result_size)" "0" "stdlib.h;gnutls/gnutls.h" HAVE_DECL_GNUTLS_FINGERPRINT)
endif(HAVE_GNUTLS_GNUTLS_H)
if(NOT HAVE_DECL_MD5)
	message(AUTHOR_WARNING " ZM requires a working MD5 function for hashed authenication but none were found - hashed authenication will not be available")
endif(NOT HAVE_DECL_MD5)
# Dirty fix for zm_user not including correct files for MD5.
# This needs to be fixed in zm_user.[h,cpp] but such fix will also require changes to configure.ac
if(HAVE_OPENSSL_MD5_H AND HAVE_DECL_MD5)
	set(HAVE_GCRYPT_H 0)
	set(HAVE_GNUTLS_OPENSSL_H 0)
endif(HAVE_OPENSSL_MD5_H AND HAVE_DECL_MD5)

# Disable backtrace if not available
if((NOT ZM_NO_CRASHTRACE) AND ((NOT HAVE_DECL_BACKTRACE) OR (NOT HAVE_EXECINFO_H)))
	message(AUTHOR_WARNING " Backtrace is not available. disabling")
	set(ZM_NO_CRASHTRACE ON)
	set(HAVE_EXECINFO_H 0)
	set(HAVE_DECL_BACKTRACE 0)
endif((NOT ZM_NO_CRASHTRACE) AND ((NOT HAVE_DECL_BACKTRACE) OR (NOT HAVE_EXECINFO_H)))


if(NOT ZM_NO_MMAP)
	set(ZM_MMAP_PERLPACKAGE "Sys::Mmap")
endif(NOT ZM_NO_MMAP)
# Check for Perl. Version checking is removed because its ignored before CMake 2.8.8 and it seems that Perl is being detected as 5.16. instead of 5.6.0 ??
find_package(Perl)
if(NOT PERL_FOUND)
	message(FATAL_ERROR "zm requires Perl 5.6.0 or newer but it was not found on your system")
endif(NOT PERL_FOUND)
# Checking for perl modules requires FindPerlModules.cmake
# Check all required modules at once
# TODO: Add checking for the optional modules
find_package(PerlModules COMPONENTS Sys::Syslog DBI DBD::mysql Getopt::Long Time::HiRes Date::Manip LWP::UserAgent ExtUtils::MakeMaker ${ZM_MMAP_PERLPACKAGE})
if(NOT PERLMODULES_FOUND)
	message(FATAL_ERROR "Not all required perl modules were found on your system")
endif(NOT PERLMODULES_FOUND)

# Attempt to check which user apache (or other web server) runs on by searching for known users in /etc/passwd
# At this point we're only checking for apache, www-data and www
if(ZM_WEB_USER STREQUAL "")
	# Check if apache is running under user apache
	file(STRINGS "/${CMAKE_INSTALL_SYSCONFDIR}/passwd" userline REGEX "^apache")
	if(NOT (userline STREQUAL ""))
		set(ZM_WEB_USER apache)
	endif(NOT (userline STREQUAL ""))
	# Check if apache is running under user www-data
	file(STRINGS "/${CMAKE_INSTALL_SYSCONFDIR}/passwd" userline REGEX "^www-data")
	if(NOT ("${userline}" STREQUAL ""))
		set(ZM_WEB_USER www-data)
	endif(NOT ("${userline}" STREQUAL ""))
	# Check if apache is running under user www
	file(STRINGS "/${CMAKE_INSTALL_SYSCONFDIR}/passwd" userline REGEX "^www")
	if(NOT ("${userline}" STREQUAL ""))
		set(ZM_WEB_USER www)
	endif(NOT ("${userline}" STREQUAL ""))
	message(STATUS "Detected web server user: ${ZM_WEB_USER}")
endif(ZM_WEB_USER STREQUAL "")
# Check if webgroup contains anything. If not, use the web user as the web group
if(NOT ZM_WEB_GROUP)
	set(ZM_WEB_GROUP ${ZM_WEB_USER})
endif(NOT ZM_WEB_GROUP)
message(STATUS "Using webuser: ${ZM_WEB_USER}")
message(STATUS "Using webgroup: ${ZM_WEB_GROUP}")

# Some variables that zm expects
set(ZM_PID "${ZM_RUNDIR}/zm.pid")
set(ZM_CONFIG "/${CMAKE_INSTALL_SYSCONFDIR}/zm.conf")
set(VERSION "${ZoneMinder_VERSION}")
set(PATH_BUILD "${PROJECT_SOURCE_DIR}")
set(TIME_BUILD "1000000") # Don't have a solution for this one yet
set(BINDIR "${CMAKE_INSTALL_FULL_BINDIR}")
set(LIBDIR "${CMAKE_INSTALL_FULL_LIBDIR}")
set(SYSCONFDIR "/${CMAKE_INSTALL_SYSCONFDIR}")
set(WEB_PREFIX "${ZM_WEBDIR}")
set(CGI_PREFIX "${ZM_CGIDIR}")
set(WEB_USER "${ZM_WEB_USER}")
set(WEB_GROUP "${ZM_WEB_GROUP}")

# Generate files from the .in files
configure_file(zoneminder-config.cmake config.h @ONLY)
configure_file(zm.conf.in zm.conf @ONLY)
configure_file(zmconfgen.pl.in zmconfgen.pl @ONLY)
configure_file(zmlinkcontent.sh.in zmlinkcontent.sh @ONLY)

# Run ZM configuration generator
message(STATUS "Running ZoneMinder configuration generator")
execute_process(COMMAND perl ./zmconfgen.pl RESULT_VARIABLE zmconfgen_result)
if(zmconfgen_result EQUAL 0)
	message(STATUS "ZoneMinder configuration generator completed successfully")
else(zmconfgen_result EQUAL 0)
	message(FATAL_ERROR "ZoneMinder configuration generator failed. Exit code: ${zmconfgen_result}")
endif(zmconfgen_result EQUAL 0)

# Process subdirectories
add_subdirectory(src)
add_subdirectory(scripts)
add_subdirectory(db)
add_subdirectory(misc)
add_subdirectory(web)

# Install zm.conf
install(FILES zm.conf DESTINATION "/${CMAKE_INSTALL_SYSCONFDIR}")

