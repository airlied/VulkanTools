file(GLOB TEXTURES
   "${PROJECT_SOURCE_DIR}/via/images/*"
)
file(COPY ${TEXTURES} DESTINATION ${CMAKE_BINARY_DIR}/via/images)


if(WIN32)
    set (LIBRARIES "vulkan-${MAJOR}")

    # For Windows, since 32-bit and 64-bit items can co-exist, we build each in its own build directory.
    # 32-bit target data goes in build32, and 64-bit target data goes into build.  So, include/link the
    # appropriate data at build time.
    if (CMAKE_CL_64)
        set (BUILDTGT_DIR build)
    else ()
        set (BUILDTGT_DIR build32)
    endif()

    # Use static MSVCRT libraries
    foreach(configuration in CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELEASE CMAKE_C_FLAGS_RELWITHDEBINFO
                             CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(${configuration} MATCHES "/MD")
            string(REGEX REPLACE "/MD" "/MT" ${configuration} "${${configuration}}")
        endif()
    endforeach()

    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_CRT_SECURE_NO_WARNINGS -D_USE_MATH_DEFINES")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_CRT_SECURE_NO_WARNINGS -D_USE_MATH_DEFINES")

else()

    if(UNIX)
        set (LIBRARIES "vulkan")
    endif()

    if (BUILD_WSI_XCB_SUPPORT)
        find_package(XCB REQUIRED)

        include_directories(${XCB_INCLUDE_DIRS})
        link_libraries(${XCB_LIBRARIES})
    endif()
    if (BUILD_WSI_XLIB_SUPPORT)
        find_package(X11 REQUIRED)

        include_directories(${X11_INCLUDE_DIRS})
        link_libraries(${X11_LIBRARIES})
    endif()
    if (BUILD_WSI_WAYLAND_SUPPORT)
        find_package(Wayland REQUIRED)

        include_directories(${WAYLAND_CLIENT_INCLUDE_DIR})
        link_libraries(${WAYLAND_CLIENT_LIBRARIES})
    endif()

    link_libraries(vulkan m)

endif()

add_executable(via via.cpp ${JSONCPP_SOURCE_DIR}/jsoncpp.cpp)
target_include_directories(via PUBLIC ${JSONCPP_INCLUDE_DIR})
target_link_libraries(via ${LIBRARIES})
if(WIN32)
    target_link_libraries(via version)
endif()
