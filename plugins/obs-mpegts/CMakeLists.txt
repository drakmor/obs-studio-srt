cmake_minimum_required(VERSION 3.16...3.25)

legacy_check()

find_package(FFmpeg REQUIRED COMPONENTS avcodec avutil avformat)

add_library(obs-mpegts MODULE)
add_library(OBS::mpegts ALIAS obs-mpegts)

find_package(Librist REQUIRED)
find_package(Libsrt REQUIRED)

foreach(_output_lib IN ITEMS Librist Libsrt)
  if(NOT TARGET ${_output_lib}::${_output_lib})
    list(APPEND _error_messages "MPEGTS output library ${_output_lib} not found.")
  endif()
endforeach()

if(_error_messages)
  list(JOIN "\n" _error_string _error_string)
  message(
    FATAL_ERROR
      "${_error_string}\n Disable this error by setting ENABLE_NEW_MPEGTS_OUTPUT to OFF or providing the build system with required SRT and Rist libraries."
  )
endif()

target_sources(obs-mpegts PRIVATE obs-mpegts.c obs-mpegts-common.h obs-mpegts-rist.h obs-mpegts-srt.h)

target_link_libraries(obs-mpegts PRIVATE OBS::libobs FFmpeg::avcodec FFmpeg::avformat FFmpeg::avutil Librist::Librist
                                         Libsrt::Libsrt)

if(OS_WINDOWS)
  target_link_libraries(obs-mpegts PRIVATE ws2_32)
  target_link_libraries(obs-mpegts PRIVATE OBS::w32-pthreads)

  configure_file(cmake/windows/obs-module.rc.in obs-mpegts.rc)
  target_sources(obs-mpegts PRIVATE obs-mpegts.rc)
endif()

set_target_properties_obs(obs-mpegts PROPERTIES FOLDER plugins PREFIX "")

