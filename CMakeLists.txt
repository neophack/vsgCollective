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
option(VSGCOLLECTIVE_SHARED_LIBRARY "Set to ON to build libraries for dynamic linking.  Use OFF for static." ON)

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

##############################################################################################################
#
# VulkkanSDK
#
if (VULKAN_SDK)
    set(ENV{VULKAN_SDK} ${VULKAN_SDK})
    find_package(Vulkan)
endif()

if (NOT VULKAN_FOUND)
    set(VulkanSDKVersion "1.2.176.1" CACHE STRING "Vulkan_SDK version to download")

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

        find_package(Vulkan)

    endif()
endif()


##############################################################################################################
#
# glslang
#
OPTION(build_glslang "Build glslang" ON)
if (${build_glslang})
    ExternalProject_Add(glslang
        PREFIX components
        GIT_REPOSITORY "https://github.com/KhronosGroup/glslang.git"
        GIT_TAG "master"
        SOURCE_DIR components/glslang
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
endif()

##############################################################################################################
#
# VulkaSceneGraph
#
OPTION(build_VulkanSceneGraph "Build VulkanSceneGraph" ON)
if (${build_VulkanSceneGraph})
    ExternalProject_Add(VulkanSceneGraph
        PREFIX components
        GIT_REPOSITORY "https://github.com/vsg-dev/VulkanSceneGraph.git"
        GIT_TAG "master"
        SOURCE_DIR components/VulkanSceneGraph
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
    ExternalProject_Add_StepDependencies(VulkanSceneGraph build glslang)
endif()


##############################################################################################################
#
# vsgGIS
#
OPTION(build_vsgGIS "Build vsgGIS" ON)
if (${build_vsgGIS})
    ExternalProject_Add(vsgGIS
        PREFIX components
        GIT_REPOSITORY "https://github.com/vsg-dev/vsgGIS.git"
        GIT_TAG "master"
        SOURCE_DIR components/vsgGIS
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
    ExternalProject_Add_StepDependencies(vsgGIS build VulkanSceneGraph)
endif()



##############################################################################################################
#
# assimp
#
OPTION(build_assimp "Build assimp" ON)
if (${build_assimp})
    ExternalProject_Add(assimp
        PREFIX components
        GIT_REPOSITORY "https://github.com/assimp/assimp.git"
        GIT_TAG "master"
        SOURCE_DIR components/assimp
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
endif()


##############################################################################################################
#
# vsgXchange
#
OPTION(build_vsgXchange "Build vsgXchange" ON)
if (${build_vsgXchange})
    ExternalProject_Add(vsgXchange
        PREFIX components
        GIT_REPOSITORY "https://github.com/vsg-dev/vsgXchange.git"
        GIT_TAG "master"
        SOURCE_DIR components/vsgXchange
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
    ExternalProject_Add_StepDependencies(vsgXchange build assimp vsgGIS VulkanSceneGraph)
endif()


##############################################################################################################
#
# vsgImGui
#
OPTION(build_vsgImGui "Build vsgImGui" ON)
if (${build_vsgImGui})
    ExternalProject_Add(vsgImGui
        PREFIX components
        GIT_REPOSITORY "https://github.com/vsg-dev/vsgImGui.git"
        GIT_TAG "master"
        SOURCE_DIR components/vsgImGui
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
    ExternalProject_Add_StepDependencies(vsgImGui build vsgXchange VulkanSceneGraph)
endif()


##############################################################################################################
#
# vsgExamples
#
OPTION(build_vsgExamples "Build vsgExamples" ON)
if (${build_vsgExamples})
    ExternalProject_Add(vsgExamples
        PREFIX components
        GIT_REPOSITORY "https://github.com/vsg-dev/vsgExamples.git"
        GIT_TAG "master"
        SOURCE_DIR components/vsgExamples
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX=${PROJECT_BINARY_DIR} -DCMAKE_PREFIX_PATH=${PREFIX_PATH} -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS} "${EXTRA_CMAKE_ARGS_VULKAN_SDK}"
    )
    ExternalProject_Add_StepDependencies(vsgExamples build VulkanSceneGraph vsgXchange vsgImGui)
endif()
