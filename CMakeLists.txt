cmake_minimum_required(VERSION 3.20)
project(VulkanTriangle)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


# 设置 vcpkg 工具链
#if(DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
#    set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
#endif()

# 查找依赖包
find_package(Vulkan REQUIRED)
find_package(glfw3 REQUIRED)
find_package(glm REQUIRED)

# 检查 Vulkan SDK
if(NOT Vulkan_FOUND)
    message(FATAL_ERROR "Vulkan SDK not found. Please install Vulkan SDK.")
endif()


# 添加可执行文件
add_executable(VulkanTriangle src/main.cpp)

# 链接库
target_link_libraries(VulkanTriangle 
    Vulkan::Vulkan
    glfw
    glm::glm
)

# 设置包含目录
target_include_directories(VulkanTriangle PRIVATE src)

# 设置编译定义
target_compile_definitions(VulkanTriangle PRIVATE
    VULKAN_HPP_DISPATCH_LOADER_DYNAMIC=1
    VULKAN_HPP_NO_STRUCT_CONSTRUCTORS=1
)

# 设置构建类型
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()

# 调试标志配置
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    # 在 Apple 平台上使用 -O1 而不是 -Od 以避免内联导致调试问题
    if(APPLE)
        set(CMAKE_CXX_FLAGS_DEBUG "-O1 -g")
        set(CMAKE_C_FLAGS_DEBUG "-O1 -g")
    else()
        set(CMAKE_CXX_FLAGS_DEBUG "-Od -g")
        set(CMAKE_C_FLAGS_DEBUG "-Od -g")
    endif()
    
    # 添加调试定义
    target_compile_definitions(VulkanTriangle PRIVATE
        DEBUG_BUILD=1
        _DEBUG
    )
else()
    # Release 模式优化
    if(APPLE)
        set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
        set(CMAKE_C_FLAGS_RELEASE "-O3 -DNDEBUG")
    else()
        set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG")
        set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
    endif()
endif()

# 平台特定的编译定义
if(WIN32)
    target_compile_definitions(VulkanTriangle PRIVATE
        VK_USE_PLATFORM_WIN32_KHR
    )
elseif(APPLE)
    target_compile_definitions(VulkanTriangle PRIVATE
        VK_USE_PLATFORM_MACOS_MVK
        __INTELLISENSE__
    )
elseif(UNIX)
    target_compile_definitions(VulkanTriangle PRIVATE
        VK_USE_PLATFORM_XLIB_KHR
    )
endif()

# 查找 Slang 编译器
find_program(SLANGC_EXECUTABLE slangc
    HINTS
        "C:/Program Files/Slang/bin"
        "C:/Slang/bin"
        "/usr/local/bin"
        "/opt/homebrew/bin"
        "$ENV{SLANG_ROOT}/bin"
        "$ENV{HOME}/.local/bin"
    DOC "Slang compiler executable"
)

if(NOT SLANGC_EXECUTABLE)
    message(WARNING "Slang compiler not found. Shader compilation will be skipped.")
    message(STATUS "Please install Slang and set SLANGC_EXECUTABLE or add to PATH")
else()
    message(STATUS "Found Slang compiler: ${SLANGC_EXECUTABLE}")
endif()

function (add_slang_shader_target TARGET)
    cmake_parse_arguments ("SHADER" "" "" "SOURCES" ${ARGN})
    
    if(NOT SLANGC_EXECUTABLE)
        message(WARNING "Slang compiler not found, skipping shader target: ${TARGET}")
        return()
    endif()
    
    set (SHADERS_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/shaders)
    set (SHADERS_BUILD_DIR ${CMAKE_CURRENT_BINARY_DIR}/shaders)
    set (SHADER_SOURCES ${SHADERS_SOURCE_DIR}/shader.slang)
    set (OUTPUT_SPV ${SHADERS_BUILD_DIR}/shader.spv)
    
    # 确保着色器源文件存在
    if(NOT EXISTS ${SHADER_SOURCES})
        message(FATAL_ERROR "Shader source file not found: ${SHADER_SOURCES}")
    endif()
    
    # 创建构建目录
    file(MAKE_DIRECTORY ${SHADERS_BUILD_DIR})
    
    # 编译着色器
    add_custom_command (
        OUTPUT ${OUTPUT_SPV}
        COMMAND ${SLANGC_EXECUTABLE} 
            ${SHADER_SOURCES} 
            -target spirv 
            -profile spirv_1_4 
            -emit-spirv-directly 
            -fvk-use-entrypoint-name 
            -entry vertMain 
            -entry fragMain 
            -o ${OUTPUT_SPV}
        WORKING_DIRECTORY ${SHADERS_BUILD_DIR}
        DEPENDS ${SHADER_SOURCES}
        COMMENT "Compiling Slang shader: ${SHADER_SOURCES} -> ${OUTPUT_SPV}"
        VERBATIM
    )
    
    add_custom_target (${TARGET} DEPENDS ${OUTPUT_SPV})
endfunction()

# 添加着色器编译目标
add_slang_shader_target(compile_shaders)
add_dependencies(VulkanTriangle compile_shaders)

# 拷贝 textures 目录到构建目录
set(TEXTURES_SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/textures)
set(TEXTURES_BUILD_DIR ${CMAKE_CURRENT_BINARY_DIR}/textures)

# 创建构建目录
file(MAKE_DIRECTORY ${TEXTURES_BUILD_DIR})

# 添加自定义命令来拷贝 textures
add_custom_command(
    OUTPUT ${TEXTURES_BUILD_DIR}/.copied
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${TEXTURES_SOURCE_DIR} ${TEXTURES_BUILD_DIR}
    COMMAND ${CMAKE_COMMAND} -E touch ${TEXTURES_BUILD_DIR}/.copied
    DEPENDS ${TEXTURES_SOURCE_DIR}
    COMMENT "Copying textures directory to build directory"
    VERBATIM
)

# 添加自定义目标
add_custom_target(copy_textures DEPENDS ${TEXTURES_BUILD_DIR}/.copied)

# 让 VulkanTriangle 依赖于 copy_textures
add_dependencies(VulkanTriangle copy_textures)
