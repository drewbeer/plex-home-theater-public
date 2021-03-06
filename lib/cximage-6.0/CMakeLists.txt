if(NOT TARGET_WIN32)
  add_definitions(-D_DLL -DXBMC)
  include_directories(${dependdir}/include)
else(NOT TARGET_WIN32)
  set(COMPILE_DEFINITIONS "")
  add_definitions(-D_DLL -DWIN32 -D_WINDOWS -D_USRDLL -D_AFXDLL -DJAS_WIN_MSVC_BUILD -D_CRT_SECURE_NO_DEPRECATE -D_CONSOLE)
  set(CMAKE_C_FLAGS_DEBUG "/MP /Od /Oy- /D_DEBUG /Gm- /MDd /GS /arch:SSE /fp:precise /Zc:wchar_t /Zc:forScope /wd\"4996\"")
  set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
  set(CMAKE_C_FLAGS_RELEASE " /WX- /O2 /Ob1 /Oy- /DNDEBUG /GF /Gm- /EHa /MD /GS- /Gy /arch:SSE /fp:precise /Zc:wchar_t /Zc:forScope")
  set(CMAKE_CXX_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})
endif(NOT TARGET_WIN32)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(DISABLE_WARNS "-Wno-unused-value -Wno-format -Wno-null-arithmetic -Wno-switch -Wno-string-plus-int -Wno-dangling-else")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${DISABLE_WARNS} -fPIC -fno-common")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${DISABLE_WARNS} -fPIC -fno-common")
endif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")

set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} ${PLEX_LINK_WRAPPED}")

if(TARGET_WIN32)
  # Use the zlib that comes with cximage
  include_directories(zlib zlib/win32)
endif(TARGET_WIN32)

# 0 not support, 1 = supported internally, 2 = supported externally 
set(CX_SUPPORT_JBIG 0)
set(CX_SUPPORT_JSP 0)
set(CX_SUPPORT_RAW 1)
set(CX_SUPPORT_ZLIB 1)

if(NOT WIN32)
  set(CX_SUPPORT_MNG 0)
  set(CX_SUPPORT_JPG 2)
  set(CX_SUPPORT_TIFF 2)
  set(CX_SUPPORT_PNG 2)
else(NOT WIN32)
  set(CX_SUPPORT_JPG 1)
  set(CX_SUPPORT_MNG 1)
  set(CX_SUPPORT_TIFF 1)
  set(CX_SUPPORT_PNG 1)
endif(NOT WIN32)

add_subdirectory(CxImage)
set(OBJECTS cximage)

if(CX_SUPPORT_JBIG EQUAL 1)
  add_subdirectory(jbig)
  list(APPEND OBJECTS cx_jbig)
endif(CX_SUPPORT_JBIG EQUAL 1)

if(CX_SUPPORT_JSP EQUAL 1)
  add_subdirectory(jasper)
  list(APPEND OBJECTS cx_jasper)
endif(CX_SUPPORT_JSP EQUAL 1)

if(CX_SUPPORT_RAW EQUAL 1)
  add_subdirectory(raw)
  list(APPEND OBJECTS cx_raw)
endif(CX_SUPPORT_RAW EQUAL 1)

if(CX_SUPPORT_ZLIB EQUAL 1)
  add_subdirectory(zlib)
  list(APPEND OBJECTS cx_zlib)
endif(CX_SUPPORT_ZLIB EQUAL 1)

if(CX_SUPPORT_JPG EQUAL 1)
  add_subdirectory(jpeg)
  list(APPEND OBJECTS cx_jpeg)
elseif(CX_SUPPORT_JPG EQUAL 2)
  list(APPEND LIBS ${CONFIG_LIBRARY_JPEG})
endif(CX_SUPPORT_JPG EQUAL 1)

if(CX_SUPPORT_MNG EQUAL 1)
  add_subdirectory(mng)
  list(APPEND OBJECTS cx_mng)
elseif(CX_SUPPORT_MNG EQUAL 2)
  list(APPEND LIBS ${CONFIG_LIBRARY_MNG})
endif(CX_SUPPORT_MNG EQUAL 1)

if(CX_SUPPORT_TIFF EQUAL 1)
  add_subdirectory(tiff)
  list(APPEND OBJECTS cx_tiff)
elseif(CX_SUPPORT_TIFF EQUAL 2)
  list(APPEND LIBS ${CONFIG_LIBRARY_TIFF})
endif(CX_SUPPORT_TIFF EQUAL 1)

if(CX_SUPPORT_PNG EQUAL 1)
  add_subdirectory(png)
  list(APPEND OBJECTS cx_png)
elseif(CX_SUPPORT_PNG EQUAL 2)
  list(APPEND LIBS ${CONFIG_LIBRARY_PNG})
endif(CX_SUPPORT_PNG EQUAL 1)

set(SRC CxImage/DllInterface.cpp)

if(NOT TARGET_WIN32)
  list(APPEND OBJECTS wrapper)
  add_library(ImageLib-${ARCH} MODULE ${SRC})
  target_link_libraries(ImageLib-${ARCH} ${PLEX_LINK_WHOLEARCHIVE} ${OBJECTS} ${PLEX_LINK_NOWHOLEARCHIVE} ${LIBS})
else(NOT TARGET_WIN32)
  add_library(ImageLib SHARED ${SRC})
  target_link_libraries(ImageLib ${OBJECTS} ${LIBS})

  set(LIBS kernel32 user32 gdi32 winspool comdlg32 advapi32 shell32 ole32 oleaut32 uuid odbc32 odbccp32)
  foreach(l ${LIBS})
    find_library(t ${l})
    target_link_libraries(ImageLib ${t})
  endforeach(l ${LIBS})
endif(NOT TARGET_WIN32)

if(TARGET_COMMON_DARWIN)
  install(TARGETS ImageLib-${ARCH} DESTINATION ${RESOURCEPATH}/system COMPONENT RUNTIME)
elseif(TARGET_COMMON_LINUX)
  install(TARGETS ImageLib-${ARCH} DESTINATION ${BINPATH}/system COMPONENT RUNTIME)
elseif(TARGET_WIN32)
  install(TARGETS ImageLib RUNTIME DESTINATION ${RESOURCEPATH}/system LIBRARY DESTINATION ${RESOURCEPATH}/system COMPONENT RUNTIME)
endif(TARGET_COMMON_DARWIN)
