cmake_minimum_required(VERSION 3.5)

project(vsgCollective)

# find the various depdendencies
set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_SOURCE_DIR}/CMakeModules/")
include(ExternalProject)

set(OUTPUT_BINDIR ${PROJECT_BINARY_DIR}/bin)
set(OUTPUT_LIBDIR ${PROJECT_BINARY_DIR}/lib)
set(OUTPUT_INCDIR ${PROJECT_BINARY_DIR}/include)
set(COMPONENTS_DIR ${PROJECT_BINARY_DIR}/components)
set(DOWNLOADS_DIR ${PROJECT_BINARY_DIR}/downloads)
set(PREFIX_PATH   ${CMAKE_PREFIX_PATH} ${PROJECT_BINARY_DIR})

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OUTPUT_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_BINDIR})
if(WIN32)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_BINDIR})
else(WIN32)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_LIBDIR})
endif(WIN32)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_BINDIR})

# set compilation to use C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Change the default build type to Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

# Dynamic vs Static Linking
option(VSGFCOLLECTIVE_SHARED_LIBRARY "Set to ON to build libraries for dynamic linking.  Use OFF for static." ON)

add_custom_target(clobber
    COMMAND git clean -d -f -x
)

find_package(Threads)

find_library(DL_LIBRARY dl)
if(NOT DL_LIBRARY)
    set(DL_LIBRARY "") # change from NOTFOUND to empty when passed to linker
endif()

if (BUILD_SHARED_LIBS)
    set(DEFAULT_LIB_SUFFIX ${CMAKE_SHARED_LIBRARY_SUFFIX})
else()
    set(DEFAULT_LIB_SUFFIX ${CMAKE_STATIC_LIBRARY_SUFFIX})
endif()

if (VULKAN_SDK)
    set(ENV{VULKAN_SDK} ${VULKAN_SDK})
endif()

find_package(Vulkan)

if (NOT VULKAN_FOUND)

    set(VulkanSDKVersion "1.2.131.2" CACHE STRING "Vulkan_SDK version to download")

    if (NOT EXISTS ${DOWNLOADS_DIR}/${VulkanSDKVersion})

        message("Will attempt to download : " ${VulkanSDKVersion})

        file(MAKE_DIRECTORY downloads)

        execute_process(COMMAND wget --no-cookies -O ${DOWNLOADS_DIR}/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz https://sdk.lunarg.com/sdk/download/${VulkanSDKVersion}/linux/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz)
        execute_process(COMMAND tar -zxf ${DOWNLOADS_DIR}/vulkansdk-linux-x86_64-${VulkanSDKVersion}.tar.gz -C ${DOWNLOADS_DIR})

    endif()

    if (EXISTS ${DOWNLOADS_DIR}/${VulkanSDKVersion})

        set(VULKAN_SDK ${DOWNLOADS_DIR}/${VulkanSDKVersion}/x86_64)
        set(ENV{VULKAN_SDK} ${VULKAN_SDK})
        set(EXTRA_CMAKE_ARGS_VULKAN_SDK -DVULKAN_SDK=${VULKAN_SDK})

        message("VULKAN_SDK = " $ENV{VULKAN_SDK})

        find_package(Vulkan)

    endif()

endif()


macro(addExternal TARGET_NAME REPOSITORY TAG EXTRA_CMAKE_ARGS1 EXTRA_CMAKE_ARGS2 EXTRA_CMAKE_ARGS3 EXTRA_CMAKE_ARGS4)

    ExternalProject_Add(
        ${TARGET_NAME}

        PREFIX components

        GIT_REPOSITORY ${REPOSITORY}
        GIT_TAG ${TAG}

        UPDATE_COMMAND ""
        PATCH_COMMAND ""

        SOURCE_DIR "${COMPONENTS_DIR}/${TARGET_NAME}"
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} ${EXTRA_CMAKE_ARGS1} ${EXTRA_CMAKE_ARGS2} ${EXTRA_CMAKE_ARGS3} ${EXTRA_CMAKE_ARGS4}
    )
endmacro()


addExternal(VulkanSceneGraph "https://github.com/vsg-dev/VulkanSceneGraph.git" "master" "-DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}" "${EXTRA_CMAKE_ARGS_VULKAN_SDK}" "" "")