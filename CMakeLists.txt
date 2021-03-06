#
# Angharad
#
# CMake file used to build program
#
# Copyright 2018 Nicolas Mora <mail@babelouest.org>
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the MIT License
#
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
#

cmake_minimum_required(VERSION 3.5)

project(angharad)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Werror")

# library info

set(PROGRAM_VERSION_MAJOR "1")
set(PROGRAM_VERSION_MINOR "2")
set(PROGRAM_VERSION_PATCH "3")
set(PROJECT_DESCRIPTION "Online password and secret locker")
set(PROJECT_BUGREPORT_PATH "https://github.com/babelouest/angharad/issues")
set(ORCANIA_VERSION_DOWNLOAD "1.2.9")
set(YDER_VERSION_DOWNLOAD "1.4.4")
set(ULFIUS_VERSION_DOWNLOAD "2.4.4")
set(HOEL_VERSION_DOWNLOAD "1.4.6")

set(BENOIC_MODULES_SRC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/src/benoic/device-modules/")
set(CARLEON_MODULES_SRC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/src/carleon/service-modules/")
set(BENOIC_DEVICES "")
set(CARLEON_SERVICES "")

include(GNUInstallDirs)
include(CheckSymbolExists)

# cmake modules

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake-modules)

# check if _GNU_SOURCE is available

if (NOT _GNU_SOURCE)
    check_symbol_exists(__GNU_LIBRARY__ "features.h" _GNU_SOURCE)

    if (NOT _GNU_SOURCE)
        unset(_GNU_SOURCE CACHE)
        check_symbol_exists(_GNU_SOURCE "features.h" _GNU_SOURCE)
    endif ()
endif ()

if (_GNU_SOURCE)
    add_definitions(-D_GNU_SOURCE)
endif ()

include(FindJansson)
set(JANSSON_MIN_VERSION 2.4)
find_package(Jansson ${JANSSON_MIN_VERSION} REQUIRED)
if (JANSSON_FOUND)
    set(LIBS ${JANSSON_LIBRARIES})
endif ()

include(FindLibJWT)
find_package(LibJWT REQUIRED)
if (LIBJWT_FOUND)
    set(LIBS ${LIBS} ${LIBJWT_LIBRARIES})
endif ()

include(FindLibconfig)
find_package(Libconfig REQUIRED)
if (LIBCONFIG_FOUND)
    set(LIBS ${LIBS} ${LIBCONFIG_LIBRARIES})
    include_directories(${LIBCONFIG_INCLUDE_DIRS})
endif ()

# build

add_executable(angharad ${CMAKE_CURRENT_SOURCE_DIR}/src/angharad.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/glewlwyd_resource.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/static_file_callback.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/glewlwyd_resource.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/static_file_callback.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/condition.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/profile.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/scheduler.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/script.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/trigger.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/webservice.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/benoic/benoic.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/benoic/benoic.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/benoic/device.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/benoic/device-element.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/carleon/carleon.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/carleon/carleon.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/carleon/service.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/gareth/gareth.h
                ${CMAKE_CURRENT_SOURCE_DIR}/src/gareth/gareth.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/gareth/alert.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/gareth/filter.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/gareth/message.c
                ${CMAKE_CURRENT_SOURCE_DIR}/src/angharad.c )
target_link_libraries(angharad ${LIBS})

set(Orcania_FIND_QUIETLY ON) # force to find Orcania quietly
include(FindOrcania)
find_package(Orcania 1.1 QUIET) # try to find orcania
if (NOT ORCANIA_FOUND)
    include(DownloadProject)
    option(INSTALL_HEADER "specific param" off)
    download_project(PROJ orcania # ... otherwise, download archive
            URL "https://github.com/babelouest/orcania/archive/v${ORCANIA_VERSION_DOWNLOAD}.tar.gz"
            QUIET)
    add_subdirectory(${orcania_SOURCE_DIR} ${orcania_BINARY_DIR})
    include_directories(${orcania_SOURCE_DIR}/include)
    add_dependencies(angharad orcania)
		set(ORCANIA_LIBRARIES orcania)
endif ()
set(LIBS ${LIBS} ${ORCANIA_LIBRARIES})

set(Yder_FIND_QUIETLY ON) # force to find Yder quietly
include(FindYder)
find_package(Yder 1.1 QUIET) # try to find Yder
if (NOT YDER_FOUND)
    include(DownloadProject)
    option(CHECK_ORCANIA "specific param" off)
    option(INSTALL_HEADER "specific param" off)
    download_project(PROJ yder # ... otherwise, download archive
            URL "https://github.com/babelouest/yder/archive/v${YDER_VERSION_DOWNLOAD}.tar.gz"
            QUIET)
    add_subdirectory(${yder_SOURCE_DIR} ${yder_BINARY_DIR})
    include_directories(${yder_SOURCE_DIR}/include)
    include_directories(${orcania_SOURCE_DIR}/include)
    add_dependencies(angharad yder)
		set(YDER_LIBRARIES yder)
endif ()
set(LIBS ${LIBS} ${YDER_LIBRARIES})

set(Ulfius_FIND_QUIETLY ON)
include(FindUlfius)
find_package(Ulfius 2.2 QUIET)
if (NOT ULFIUS_FOUND)
    include(DownloadProject)
    option(WITH_WEBSOCKET "specific param" off)
    option(INSTALL_HEADER "specific param" off)
    option(CHECK_YDER "specific param" off)
    download_project(PROJ ulfius
            URL "https://github.com/babelouest/ulfius/archive/v${ULFIUS_VERSION_DOWNLOAD}.tar.gz"
            QUIET)
    add_subdirectory(${ulfius_SOURCE_DIR} ${ulfius_BINARY_DIR})
    include_directories(${yder_SOURCE_DIR}/include)
    include_directories(${orcania_SOURCE_DIR}/include)
    include_directories(${ulfius_SOURCE_DIR}/include)
    add_dependencies(angharad ulfius)
		set(ULFIUS_LIBRARIES ulfius)
endif ()
set(LIBS ${LIBS} ${ULFIUS_LIBRARIES})

set(Hoel_FIND_QUIETLY ON)
include(FindHoel)
find_package(Hoel 1.3 QUIET)
if (NOT HOEL_FOUND)
    include(DownloadProject)
    option(WITH_PGSQL "specific param" off)
    option(INSTALL_HEADER "specific param" off)
    option(CHECK_YDER "specific param" off)
    download_project(PROJ hoel
            URL "https://github.com/babelouest/hoel/archive/v${HOEL_VERSION_DOWNLOAD}.tar.gz"
            QUIET)
    add_subdirectory(${hoel_SOURCE_DIR} ${hoel_BINARY_DIR})
    include_directories(${hoel_SOURCE_DIR}/include)
    add_dependencies(angharad hoel)
		set(HOEL_LIBRARIES hoel)
endif ()
set(LIBS ${LIBS} ${HOEL_LIBRARIES})

target_link_libraries(angharad ${LIBS} pthread m dl)

# benoic device modules

option(WITH_MOCKDEV "Build Mock device" off)
if (WITH_MOCKDEV)
		set(MOCK_LIB_SRC ${BENOIC_MODULES_SRC_PATH}/device-mock.c)

		set(MOCK_LIBS "-ldl")

		add_library(devmock MODULE ${MOCK_LIB_SRC})
		set_target_properties(devmock PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(devmock ${MOCK_LIBS})
		set(BENOIC_DEVICES ${BENOIC_DEVICES} devmock)
endif ()

option(WITH_ZWAVE "Build ZWave device" on)
if (WITH_ZWAVE)
		set(ZWAVE_LIB_SRC ${BENOIC_MODULES_SRC_PATH}/device-zwave.cpp)

		include(FindOpenZWAVE)
		find_package(OpenZWAVE REQUIRED)
		if (OPENZWAVE_FOUND)
				set(ZWAVE_LIBS ${OPENZWAVE_LIBRARIES} "-ldl -Wl,-R '-Wl,-R$$ORIGIN'")
				include_directories(${OPENZWAVE_INCLUDE_DIRS})
		endif ()

		add_library(devzwave MODULE ${ZWAVE_LIB_SRC})
		set_target_properties(devzwave PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(devzwave ${ZWAVE_LIBS})
		set(BENOIC_DEVICES ${BENOIC_DEVICES} devzwave)
endif ()

option(WITH_TAULAS "Build Taulas device" on)
if (WITH_TAULAS)
		set(TAULAS_LIB_SRC ${BENOIC_MODULES_SRC_PATH}/device-taulas.c)

		set(TAULAS_LIBS "-ldl")

		add_library(devtaulas MODULE ${TAULAS_LIB_SRC})
		set_target_properties(devtaulas PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(devtaulas ${TAULAS_LIBS})
		set(BENOIC_DEVICES ${BENOIC_DEVICES} devtaulas)
endif ()

# carleon services modules

option(WITH_MOCKSERVICE "Build Mock service" off)
if (WITH_MOCKSERVICE)
		set(MOCK_LIB_SRC ${CARLEON_MODULES_SRC_PATH}/service-mock.c)

		set(MOCK_LIBS "-ldl")

		add_library(servicemock MODULE ${MOCK_LIB_SRC})
		set_target_properties(servicemock PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(servicemock ${MOCK_LIBS})
		set(CARLEON_SERVICES ${CARLEON_SERVICES} servicemock)
endif ()

option(WITH_MOTION "Build Motion service" on)
if (WITH_MOTION)
		set(MOTION_LIB_SRC ${CARLEON_MODULES_SRC_PATH}/service-motion.c)

		set(MOTION_LIBS "-ldl")

		add_library(servicemotion MODULE ${MOTION_LIB_SRC})
		set_target_properties(servicemotion PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(servicemotion ${MOTION_LIBS})
		set(CARLEON_SERVICES ${CARLEON_SERVICES} servicemotion)
endif ()

option(WITH_LIQUIDSOAP "Build Liquidsoap service" on)
if (WITH_LIQUIDSOAP)
		set(LIQUIDSOAP_LIB_SRC ${CARLEON_MODULES_SRC_PATH}/service-liquidsoap.c)

		set(LIQUIDSOAP_LIBS "-ldl")

		add_library(serviceliquidsoap MODULE ${LIQUIDSOAP_LIB_SRC})
		set_target_properties(serviceliquidsoap PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(serviceliquidsoap ${LIQUIDSOAP_LIBS})
		set(CARLEON_SERVICES ${CARLEON_SERVICES} serviceliquidsoap)
endif ()

option(WITH_MPD "Build MPD service" on)
if (WITH_MPD)
		set(MPD_LIB_SRC ${CARLEON_MODULES_SRC_PATH}/service-mpd.c)

		include(FindLibMPDClient)
		find_package(LibMPDClient REQUIRED)
		if (LIBMPDCLIENT_FOUND)
				set(LIBMPDCLIENT_LIBRARIES ${LIBMPDCLIENT_LIBRARIES} "-ldl")
				include_directories(${LIBMPDCLIENT_INCLUDE_DIRS})
		endif ()

		add_library(servicempd MODULE ${MPD_LIB_SRC})
		set_target_properties(servicempd PROPERTIES
						COMPILE_OPTIONS -Wextra)
		target_link_libraries(servicempd ${LIBMPDCLIENT_LIBRARIES})
		set(CARLEON_SERVICES ${CARLEON_SERVICES} servicempd)
endif ()

# tests

option(BUILD_TESTING "Build the testing tree." OFF)

if (BUILD_TESTING)
    include(FindCheck)
    find_package(Check REQUIRED)
    if (CHECK_FOUND)
        include(FindSubunit)
        find_package(Subunit REQUIRED)

        enable_testing()

        set(CMAKE_CTEST_COMMAND ctest -V)

        set(TST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/test)
        set(LIBS ${LIBS} ${CHECK_LIBRARIES} ulfius yder orcania)
        find_package(Threads REQUIRED)
        set(LIBS ${LIBS} ${SUBUNIT_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} m rt)

				set(TESTS angharad_submodule angharad_script angharad_scheduler angharad_trigger angharad_scheduler_trigger_exec angharad_profile benoic_device benoic_device_element benoic_monitor carleon_service carleon_liquidsoap carleon_motion carleon_mpd gareth_alert_smtp gareth_alert_http gareth_filter gareth_message gareth_trigger)
            

        configure_file(
                "${CMAKE_MODULE_PATH}/CTestCustom.cmake.in"
                "${CMAKE_CURRENT_BINARY_DIR}/CTestCustom.cmake"
                @ONLY)

        foreach (t ${TESTS})
            add_executable(${t} EXCLUDE_FROM_ALL ${TST_DIR}/${t}.c ${TST_DIR}/unit-tests.c ${TST_DIR}/unit-tests.h)
            target_include_directories(${t} PUBLIC ${TST_DIR})
            target_link_libraries(${t} PUBLIC ${LIBS})
            add_test(NAME ${t}
                    WORKING_DIRECTORY ${TST_DIR}
                    COMMAND ${t})
        endforeach ()
    endif ()
endif ()

# install target

install(TARGETS angharad
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES docs/angharad.conf.sample 
        DESTINATION ${CMAKE_INSTALL_SYSCONFDIR}/angharad RENAME angharad.conf COMPONENT config)
install(TARGETS ${BENOIC_DEVICES}
				DESTINATION ${CMAKE_INSTALL_LIBDIR}/angharad/benoic)
install(TARGETS ${CARLEON_SERVICES}
				DESTINATION ${CMAKE_INSTALL_LIBDIR}/angharad/carleon)
install(DIRECTORY webapp/
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/angharad/webapp/ COMPONENT runtime)
install(FILES webapp/config.json.sample 
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/angharad/webapp/ RENAME config.json COMPONENT config)
install(FILES README.md
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)
install(FILES docs/API.md
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)
install(FILES docs/angharad-init
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)
install(FILES docs/angharad.service
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)
install(FILES docs/angharad.mariadb.sql
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)
install(FILES docs/angharad.sqlite3.sql
        DESTINATION ${CMAKE_INSTALL_DOCDIR}/ COMPONENT runtime)

set(TARGETS angharad)

install(TARGETS ${TARGETS}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        CONFIGURATIONS Release)

# uninstall target

if (NOT TARGET uninstall)
    configure_file(
            "${CMAKE_MODULE_PATH}/CMakeUninstall.cmake.in"
            "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
            IMMEDIATE @ONLY)
    add_custom_target(uninstall
            COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif ()

# packaging

set(CPACK_PACKAGE_VERSION_MAJOR ${PROGRAM_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROGRAM_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROGRAM_VERSION_PATCH})

set(PACKAGE_FILE_NAME
        "${CMAKE_PROJECT_NAME}_${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")

set(PACKAGE_IGNORED_FILES
        "${CMAKE_CURRENT_BINARY_DIR}/;/.git/;.gitignore;~$;${CPACK_SOURCE_IGNORE_FILES}")

set(CPACK_GENERATOR "TGZ;DEB")

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "mail@babelouest.org")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION ${PROJECT_DESCRIPTION})
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/babelouest/angharad")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= 2.3.4), libconfig9, libjansson4 (>= 2.7), liborcania (>= 1.2), libyder (>= 1.3), libulfius (>= 2.3), libhoel (>= 1.4), lsb-base (>= 3.0-6)")
set(CPACK_PACKAGE_FILE_NAME ${PACKAGE_FILE_NAME})

set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME ${PACKAGE_FILE_NAME})
set(CPACK_SOURCE_IGNORE_FILES ${PACKAGE_IGNORED_FILES})

include(CPack)

add_custom_target(dist
        COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
