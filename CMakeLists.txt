cmake_minimum_required (VERSION 3.19.0)
project("libusb")

add_library(libusb SHARED
    libusb/core.c
    libusb/descriptor.c
    libusb/hotplug.c
    libusb/io.c
    libusb/strerror.c
    libusb/sync.c)

if(WIN32)
    set_target_properties(libusb PROPERTIES
        WINDOWS_EXPORT_ALL_SYMBOLS ON
        OUTPUT_NAME libusb
        PDB_NAME libusb
        COMPILE_PDB_NAME libusb
        COMPILE_PDB_OUTPUT_DIR ${PROJECT_BINARY_DIR})
else()
    set_target_properties(libusb PROPERTIES
        OUTPUT_NAME usb)
endif()

target_include_directories(libusb
    PUBLIC libusb)

if(WIN32)
    target_include_directories(libusb
        PRIVATE msvc)
    target_sources(libusb
        PRIVATE
        libusb/os/events_windows.c
        libusb/os/threads_windows.c
        libusb/os/windows_common.c
        libusb/os/windows_usbdk.c
        libusb/os/windows_winusb.c)
elseif(APPLE)
    target_include_directories(libusb
        PRIVATE Xcode)
    target_sources(libusb
        PRIVATE
        libusb/os/darwin_usb.c
        libusb/os/events_posix.c
        libusb/os/threads_posix.c)
    find_library(COREFOUNDATION_LIBRARY CoreFoundation)
    find_library(SECURITY_LIBRARY Security)
    find_library(IOKIT_LIBRARY IOKit)
    target_link_libraries(libusb
        ${COREFOUNDATION_LIBRARY}
        ${SECURITY_LIBRARY}
        ${IOKIT_LIBRARY})
endif()

install(FILES
    libusb/libusb.h
    DESTINATION include/libusb)

install(TARGETS libusb
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib)

if(MSVC)
    install(FILES
        ${PROJECT_BINARY_DIR}/libusb.pdb
        DESTINATION bin
        CONFIGURATIONS Debug RelWithDebInfo)
endif()
