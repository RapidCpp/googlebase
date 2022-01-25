CMAKE_MINIMUM_REQUIRED(VERSION 3.15)
PROJECT(base)

SET(CMAKE_CXX_STANDARD 11)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS OFF)

# only retain 1 config: Release
SET(CMAKE_CONFIGURATION_TYPES "release;debug" CACHE STRING "" FORCE)
SET(CMAKE_BUILD_TYPE release CACHE STRING "Choose the type of build, options are: release debug." FORCE)
SET(BASE_VERSION "1.0.0")
SET(BASE_SOVERSION "1.0.0")

OPTION(USE_GLIBC "Using glibc for message pump" FALSE)
OPTION(TOOLKIT_USE_GTK "Using GTK for message pump" FALSE)

IF(WIN32)
    ADD_DEFINITIONS(-DWIN32 -D_WINDOWS -DUNICODE -D_UNICODE -DNOMINMAX -DWIN32_LEAN_AND_MEAN)
	
	SET(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDLL")
	SET(CMAKE_CXX_FLAGS_RELEASE "/O2 /Oy- /Oi /Ot /Zi")
	SET(CMAKE_SHARED_LINKER_FLAGS_RELEASE "/DEBUG")
	SET(CMAKE_STATIC_LINKER_FLAGS_RELEASE "/DEBUG")

	SET(CMAKE_CXX_FLAGS_DEBUG "/Od /Oy- /Zi")
	SET(CMAKE_SHARED_LINKER_FLAGS_DEBUG "/DEBUG")
	SET(CMAKE_STATIC_LINKER_FLAGS_DEBUG "/DEBUG")
ELSEIF(APPLE)
	# Enable runtime search path support for dynamic libraries on OSX
    SET(CMAKE_MACOSX_RPATH 1)
	SET(XCODE_ATTRIBUTE_SDKROOT macosx)
	SET(CMAKE_OSX_SYSROOT macosx)

	SET(CMAKE_CXX_FLAGS_RELEASE "-O3 -g -fno-omit-frame-pointer")
	SET(CMAKE_SHARED_LINKER_FLAGS_RELEASE "-mmacosx-version-min=10.12")
	SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g -D__tb_debug__ -fno-omit-frame-pointer")
	SET(CMAKE_SHARED_LINKER_FLAGS_DEBUG "-mmacosx-version-min=10.12")
ENDIF()

INCLUDE(GNUInstallDirs)

MESSAGE(STATUS "cxx compiler id: ${CMAKE_CXX_COMPILER_ID}")
IF(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wconversion -Wshadow")
ELSEIF(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wconversion -Wshadow -Wextra")
ELSEIF(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wconversion -Wshadow -Wextra")
ELSEIF(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /wd4819 /wd4251 /wd4275 /wd4100 /wd4273")
	IF(MSVC_VERSION GREATER 1500 OR MSVC_VERSION EQUAL 1500)
		SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
	ENDIF()
ENDIF()


FILE(GLOB_RECURSE headers_public "base/*.h")
#使用 SOURCE_GROUP TREE 命令的限制：
#	1.FILE 列表不能用 RELATIVE 修饰，否则没有 TREE 需要的 root，无法正确生成
#	2.FILE 列表使用 LIST REMOVE_ITEM 移除时，需要使用全路径，增加前缀: ${CMAKE_CURRENT_SOURCE_DIR}/
#	3.SOURCE_GROUP 的 PREFIX 格式："Public\\"
SOURCE_GROUP(TREE "${CMAKE_CURRENT_SOURCE_DIR}/base" PREFIX "Public\\" FILES ${headers_public})

FILE(GLOB sources_cross_platform LIST_DIRECTORIES false
		"base/allocator/allocator_extension.cc"
		"base/allocator/type_profiler_control.cc"
		
		"base/debug/alias.cc"
		"base/debug/crash_logging.cc"
		"base/debug/debugger.cc"
		"base/debug/profiler.cc"
		"base/debug/stack_trace.cc"
		"base/debug/trace_event_impl.cc"
		"base/debug/trace_event_impl_constants.cc"
		
		"base/files/file_path.cc"
		"base/files/file_path_constants.cc"
		"base/files/file_path_watcher.cc"
		"base/files/file_util_proxy.cc"
		"base/files/important_file_writer.cc"
		"base/files/memory_mapped_file.cc"
		"base/files/scoped_temp_dir.cc"
		
		"base/json/json_file_value_serializer.cc"
		"base/json/json_parser.cc"
		"base/json/json_reader.cc"
		"base/json/json_string_value_serializer.cc"
		"base/json/json_writer.cc"
		"base/json/string_escape.cc"
		
		"base/memory/aligned_memory.cc"
		"base/memory/discardable_memory.cc"
		"base/memory/ref_counted.cc"
		"base/memory/ref_counted_memory.cc"
		"base/memory/singleton.cc"
		"base/memory/weak_ptr.cc"
		
		"base/message_loop/message_loop_proxy.cc"
		"base/message_loop/message_loop_proxy_impl.cc"

		"base/metrics/bucket_ranges.cc"
		"base/metrics/field_trial.cc"
		"base/metrics/histogram.cc"
		"base/metrics/histogram_base.cc"
		"base/metrics/histogram_samples.cc"
		"base/metrics/histogram_snapshot_manager.cc"
		"base/metrics/sample_map.cc"
		"base/metrics/sample_vector.cc"
		"base/metrics/sparse_histogram.cc"
		"base/metrics/statistics_recorder.cc"
		"base/metrics/stats_counters.cc"
		"base/metrics/stats_table.cc"
		
		"base/power_monitor/power_monitor.cc"
		
		"base/process/process_iterator.cc"
		
		"base/profiler/alternate_timer.cc"
		"base/profiler/scoped_profile.cc"
		"base/profiler/tracked_time.cc"
		
		"base/strings/latin1_string_conversions.cc"
		"base/strings/string_number_conversions.cc"
		"base/strings/string_piece.cc"
		"base/strings/string_split.cc"
		"base/strings/utf_offset_string_conversions.cc"
		"base/strings/utf_string_conversion_utils.cc"
		"base/strings/utf_string_conversions.cc"
		
		"base/synchronization/cancellation_flag.cc"
		"base/synchronization/lock.cc"

		"base/system_monitor/system_monitor.cc"
		
		"base/threading/non_thread_safe_impl.cc"
		"base/threading/post_task_and_reply_impl.cc"
		"base/threading/sequenced_worker_pool.cc"
		"base/threading/simple_thread.cc"
		"base/threading/thread.cc"
		"base/threading/thread_checker_impl.cc"
		"base/threading/thread_collision_warner.cc"
		"base/threading/thread_id_name_manager.cc"
		"base/threading/thread_restrictions.cc"
		"base/threading/watchdog.cc"
		"base/threading/worker_pool.cc"
		
		"base/time/clock.cc"
		"base/time/default_clock.cc"
		"base/time/default_tick_clock.cc"
		"base/time/tick_clock.cc"
		
		"base/third_party/dmg_fp/dtoa_wrapper.cc"
		"base/third_party/dmg_fp/g_fmt.cc"
		"base/third_party/dynamic_annotations/dynamic_annotations.c"
		"base/third_party/icu/icu_utf.cc"
		"base/third_party/nspr/prtime.cc"

        "base/trace/default_trace_controller.cc"
        "base/trace/double_trace_buffer.cc"
        "base/trace/trace_agent.cc"
        "base/trace/trace_buffer.cc"
        "base/trace/trace_config.cc"
        "base/trace/trace_ipc_name_pipe_win.cc"
        "base/trace/trace_object.cc"
        "base/trace/trace_server_client_name_pipe_win.cc"
        "base/trace/trace_writer.cc"
		
		"third_party/modp_b64/modp_b64.cc"
		
		"base/at_exit.cc"
		"base/base_paths.cc"
		"base/base64.cc"
		"base/bind_helpers.cc"
		"base/build_time.cc"
		"base/callback_internal.cc"
		"base/command_line.cc"
		"base/cpu.cc"
		"base/deferred_sequenced_task_runner.cc"
		"base/environment.cc"
		"base/file_util.cc"
		"base/guid.cc"
		"base/hash.cc"
		"base/lazy_instance.cc"
		"base/location.cc"
		"base/logging.cc"
		"base/md5.cc"
		"base/message_loop.cc"
		"base/message_pump.cc"
		"base/message_pump_default.cc"
		"base/path_service.cc"
		"base/pending_task.cc"
		"base/pickle.cc"
		"base/platform_file.cc"
		"base/process_util.cc"
		"base/rand_util.cc"
		"base/run_loop.cc"
		"base/scoped_native_library.cc"
		"base/sequence_checker_impl.cc"
		"base/sequenced_task_runner.cc"
		"base/sha1_portable.cc"
		"base/string_util.cc"
		"base/string_util_constants.cc"
		"base/stringprintf.cc"
		"base/supports_user_data.cc"
		"base/base_switches.cc"
		"base/sys_info.cc"
		"base/task_runner.cc"
		"base/thread_task_runner_handle.cc"
		"base/time.cc"
		"base/timer.cc"
		"base/tracked_objects.cc"
		"base/tracking_info.cc"
		"base/value_conversions.cc"
		"base/values.cc"
		"base/version.cc"
		"base/vlog.cc"
		)
LIST(APPEND sources_cross_platform
	"${CMAKE_CURRENT_SOURCE_DIR}/base/allocator/allocator_extension_thunks.cc"
	)
# remove all unittest sources
LIST(FILTER sources_cross_platform EXCLUDE REGEX "^.+_unittests?\\.[^.]+")
SOURCE_GROUP(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${sources_cross_platform})


ADD_DEFINITIONS(
	#-DNDEBUG
	-D__STD_C -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D_CRT_RAND_S
	-DCHROMIUM_BUILD -DCOMPONENT_BUILD -DBASE_I18N_IMPLEMENTATION -DBASE_IMPLEMENTATION
	-DNO_TCMALLOC -DENABLE_PEPPER_CDMS -DENABLE_CONFIGURATION_POLICY
	-DENABLE_ONE_CLICK_SIGNIN -DENABLE_REMOTING=1
	-DENABLE_TASK_MANAGER=1 -DENABLE_EXTENSIONS=1-DENABLE_SESSION_SERVICE=1
	-DENABLE_LANGUAGE_DETECTION=1 -DENABLE_CAPTIVE_PORTAL_DETECTION=1
	-DDYNAMIC_ANNOTATIONS_ENABLED=0 -DU_USING_ICU_NAMESPACE=0
	)

IF(NOT USE_GLIBC)
	LIST(REMOVE_ITEM sources_cross_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/atomicops_internals_x86_gcc.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/message_pump_glib.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/message_pump_aurax11.cc"
		)
	LIST(FILTER sources_cross_platform EXCLUDE REGEX "^nix/.+")
ENDIF()
IF(NOT TOOLKIT_USE_GTK)
	LIST(REMOVE_ITEM sources_cross_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/message_pump_gtk.cc"
		)
ENDIF()

IF(WIN32)
	ADD_DEFINITIONS(
		-D_WIN32_WINNT=0x0602 -DWINVER=0x0602 -DNTDDI_VERSION=0x06020000
		-D_SILENCE_STDEXT_HASH_DEPRECATION_WARNINGS -D_CRT_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_DEPRECATE -D_SCL_SECURE_NO_DEPRECATE
		-DPSAPI_VERSION=1 -DCERT_CHAIN_PARA_HAS_EXTRA_FIELDS
		-D_ATL_NO_OPENGL -D_SECURE_ATL
		-DTOOLKIT_VIEWS=1
		-DZLIB_WINAPI
		)
	FILE(GLOB_RECURSE sources_platform LIST_DIRECTORIES false
		"base/win/*.cc"
		"base/win/*.h"
		)
	SET(includes_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/base/win"
		#"${CMAKE_CURRENT_SOURCE_DIR}/third_party/wtl/include"
		)
	LIST(REMOVE_ITEM sources_cross_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/base/event_recorder_stubs.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/file_descriptor_shuffle.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/message_pump_libevent.cc"
		# Not using sha1_win.cc because it may have caused a regression to page cycler moz.
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sha1_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/string16.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_kqueue.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_stub.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/os_compat_nacl.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/rand_util_nacl.cc"
		)
	LIST(FILTER sources_cross_platform EXCLUDE REGEX "^.+_posix\.cc")
	LIST(APPEND sources_platform
		#"${CMAKE_CURRENT_SOURCE_DIR}/base/allocator/win_allocator.cc"
		#"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/debug_on_start_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/debugger_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/stack_trace_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/trace_event_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/memory_mapped_file_win.cc"
		#"${CMAKE_CURRENT_SOURCE_DIR}/base/i18n/icu_util_nacl_win64.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/memory/shared_memory_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/power_monitor/power_monitor_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process/process_iterator_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process/process_metrics_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/strings/sys_string_conversions_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/condition_variable_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock_impl_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/waitable_event_watcher_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/waitable_event_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/platform_thread_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_storage_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/worker_pool_win.cc"

		"${CMAKE_CURRENT_SOURCE_DIR}/base/base_paths_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/event_recorder_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/file_util_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/file_version_info_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/guid_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/hi_res_timer_manager_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/logging_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/message_pump_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/native_library_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/platform_file_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_info_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_util_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/rand_util_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sync_socket_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_info_win.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/time_win.cc"
		
		"${CMAKE_CURRENT_SOURCE_DIR}/base/base.rc"
		)

	SET(libs_platform wininet.lib dnsapi.lib version.lib msimg32.lib ws2_32.lib usp10.lib psapi.lib dbghelp.lib winmm.lib shlwapi.lib delayimp.lib)

ELSEIF(ANDROID)
	FILE(GLOB_RECURSE sources_platform LIST_DIRECTORIES false
		"base/android/*.h"
		"base/android/*.cpp"
		)
	LIST(REMOVE_ITEM sources_cross_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/base/base_paths_posix.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/system_monitor/system_monitor_posix.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_kqueue.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_stub.cc"
		)
	LIST(APPEND sources_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_info_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_string_conversions_posix.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_util_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/worker_pool_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/base_paths_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/stack_trace_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/trace_event_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/memory/discardable_memory_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/power_monitor/power_monitor_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/file_util_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/memory/shared_memory_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/message_pump_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/os_compat_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_info_android.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/platform_thread_android.cc"
		)

	SET(includes_platform "${CMAKE_CURRENT_SOURCE_DIR}/base/android")
	SET(libs_platform "z" "libc++_shared")
ELSEIF(LINUX)
	LIST(APPEND sources_platform
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_kqueue.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_stub.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/file_util_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/linux_util.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process/process_iterator_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process/process_metrics_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/process_util_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_info_linux.cc"
		"${CMAKE_CURRENT_SOURCE_DIR}/base/threading/platform_thread_linux.cc"
		)

ELSEIF(APPLE)
	IF(APPLE_IOS)
		FILE(GLOB_RECURSE sources_platform LIST_DIRECTORIES false
			"base/ios/*.h"
			"base/ios/*.cc"
			"base/ios/*.mm"
			"base/message_pump_io_ios.cc"
			"base/process/process_metrics_ios.cc"
			)
		LIST(REMOVE_ITEM sources_cross_platform
			"${CMAKE_CURRENT_SOURCE_DIR}/base/base_paths_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/native_library_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_string_conversions_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path_watcher_stub.cc"
			# Exclude all process_util except the minimal implementation
			# needed on iOS (mostly for unit tests).
            "${CMAKE_CURRENT_SOURCE_DIR}/base/process_util.cc"
			)
		SET(includes_platform "${CMAKE_CURRENT_SOURCE_DIR}/base/ios")
		SET(libs_platform "")
	ELSE()
		FILE(GLOB_RECURSE sources_platform LIST_DIRECTORIES false
			"base/mac/*.h"
			"base/mac/*.cc"
			"base/posix/*.cc"
			"base/*.mm"
			"base/*_mac.cc"
			"base/*_mac.h"
			"base/*_posix.cc"
			"base/message_pump_libevent.cc"
			)
		LIST(FILTER sources_platform EXCLUDE REGEX "^.+_unittests?\.[^.]+")
		LIST(FILTER sources_platform EXCLUDE REGEX "^.+/ios/.+")
		LIST(FILTER sources_platform EXCLUDE REGEX "^.+/test/.+")
		LIST(REMOVE_ITEM sources_platform
			"${CMAKE_CURRENT_SOURCE_DIR}/base/native_library_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/base_paths_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/posix/unix_domain_socket_linux.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/sys_info_ios.mm"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/critical_closure_ios.mm"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/power_monitor/power_monitor_ios.mm"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/strings/sys_string_conversions_posix.cc"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/process_util_ios.mm"
			"${CMAKE_CURRENT_SOURCE_DIR}/base/debug/stack_trace_ios.mm"
			)
		LIST(REMOVE_ITEM sources_cross_platform
			"${CMAKE_CURRENT_SOURCE_DIR}/base/memory/discardable_memory.cc"
			)
		INCLUDE_DIRECTORIES("base/mac")
	ENDIF()
	
	# iOS and MacOS
ENDIF()

# remove all unittest sources
LIST(FILTER sources_platform EXCLUDE REGEX "^.+_unittests?\\.[^.]+")
SOURCE_GROUP(TREE "${CMAKE_CURRENT_SOURCE_DIR}/base" PREFIX "Platform\\" FILES ${sources_platform})

IF(WIN32)
ELSE()
	FILE(GLOB sources_libevent
    	"third_party/libevent/*.c"
    	"third_party/libevent/*.h"
	  	)
	IF(APPLE)
		LIST(REMOVE_ITEM sources_libevent
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/devpoll.c"
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/epoll.c"
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/epoll_sub.c"
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/evport.c"
			)
		SET(includes_libevent "${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/mac")
    ELSEIF(LINUX)
		LIST(REMOVE_ITEM sources_libevent
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/kqueue.c"
			)
		SET(includes_libevent "${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/linux")
    ELSEIF(ANDROID)
		LIST(REMOVE_ITEM sources_libevent
			"${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/kqueue.c"
			)
		SET(includes_libevent "${CMAKE_CURRENT_SOURCE_DIR}/third_party/libevent/android")
    ENDIF()

    INCLUDE_DIRECTORIES("third_party/libevent/mac" ".")
    ADD_LIBRARY(libevent STATIC ${sources_libevent})
	TARGET_COMPILE_DEFINITIONS(libevent PRIVATE 
		HAVE_SYS_TIME_H
        HAVE_FCNTL_H
        HAVE_UNISTD_H
        HAVE_NETDB_H
        HAVE_SYS_SOCKET_H
        HAVE_GETADDRINFO
        HAVE_STRUCT_IN6_ADDR
        HAVE_STRLCPY
        HAVE_STRSEP
        HAVE_STRTOK_R)
    TARGET_INCLUDE_DIRECTORIES(libevent PUBLIC includes_libevent)
ENDIF()

# Install instructions for this target
ADD_LIBRARY(base SHARED
	${headers_public}
	${sources_cross_platform} ${sources_platform}
	)
SET_TARGET_PROPERTIES(base PROPERTIES
		OUTPUT_NAME base
		DEBUG_OUTPUT_NAME base${DEBUG_LIBNAME_SUFFIX}
		#VERSION ${BASE_VERSION}
		#SOVERSION ${BASE_SOVERSION}
		ARCHIVE_OUTPUT_DIRECTORY ../lib
		LIBRARY_OUTPUT_DIRECTORY ../lib
		PDB_OUTPUT_DIRECTORY     ../lib
		RUNTIME_OUTPUT_DIRECTORY ../lib)

TARGET_INCLUDE_DIRECTORIES(base PRIVATE
		${includes_platform}
		${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/third_party/modp_b64
		)

IF(WIN32)
	IF(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
		SET_TARGET_PROPERTIES(base PROPERTIES LINK_FLAGS "/DELAYLOAD:dbghelp.dll /DELAYLOAD:shell32.dll")
	ENDIF()
	IF(CMAKE_GENERATOR_PLATFORM STREQUAL "x64")
		SET(path_platform "x64")
	ELSE()
		SET(path_platform "x86")
	ENDIF()
ELSEIF(APPLE)
	SET_TARGET_PROPERTIES(base PROPERTIES
		#FRAMEWORK TRUE
  		#FRAMEWORK_VERSION A
  		MACOSX_FRAMEWORK_IDENTIFIER com.iqiyi.base
  		#MACOSX_FRAMEWORK_INFO_PLIST Info.plist
  		#VERSION ${base_VERSION}
  		#SOVERSION ${base_SOVERSION}
  		#PUBLIC_HEADER ${CMAKE_INSTALL_INCLUDEDIR}
		#RESOURCE "${RESOURCE_FILES1}"
  		#XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "iPhone Developer"
		)
	IF(CMAKE_SYSTEM_PROCESSOR MATCHES "i386")
		SET(path_platform "mac-i386")
	ELSE()
		SET(path_platform "mac-x86_64")
	ENDIF()
	SET(CMAKE_SHARED_LINKER_FLAGS "-framework AppKit -framework IOKit -framework ApplicationServices -framework CoreText -framework CoreGraphics -framework CoreServices -framework Security")
	TARGET_COMPILE_DEFINITIONS(base PRIVATE
			__APPLE__ NDEBUG
			DYNAMIC_ANNOTATIONS_ENABLED=0
			)
	TARGET_LINK_LIBRARIES(base PRIVATE libevent)
ENDIF()

TARGET_LINK_LIBRARIES(base PRIVATE ${libs_platform})

# install root
SET(CMAKE_INSTALL_PREFIX "${EXTERNAL_PATH}/base" CACHE PATH "..." FORCE)

# install public headers
INSTALL(DIRECTORY base/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/base
		FILES_MATCHING PATTERN "*.h")
INSTALL(DIRECTORY build/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${path_platform}/build
		FILES_MATCHING PATTERN "*.h")
INSTALL(DIRECTORY build/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/build
		FILES_MATCHING PATTERN "*.h")
INSTALL(DIRECTORY testing/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/testing
		FILES_MATCHING PATTERN "*.h")

# install targets(dll, lib, pdb)
IF(WIN32)
	INSTALL(TARGETS base
			RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${path_platform}/$<CONFIG>
			LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/${path_platform}/$<CONFIG>
			ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/${path_platform}/$<CONFIG>)
	INSTALL(FILES $<TARGET_PDB_FILE:${PROJECT_NAME}>
			DESTINATION ${CMAKE_INSTALL_BINDIR}/${path_platform}/$<CONFIG> OPTIONAL)
ELSEIF(APPLE)
	INSTALL(TARGETS base
			RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${path_platform}/$<CONFIG>
			LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/${path_platform}/$<CONFIG>
			ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/${path_platform}/$<CONFIG>
			PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/base
			FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}/${path_platform}/$<CONFIG>
			)
ENDIF()
