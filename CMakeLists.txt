cmake_minimum_required(VERSION 3.14)
project(ex3-1-bison LANGUAGES C)

set(CMAKE_C_STANDARD 11)

# 查找工具
find_program(FLEX_EXECUTABLE flex REQUIRED)
find_program(BISON_EXECUTABLE bison REQUIRED)

if(APPLE)
    # 设置 flex 路径
    set(FLEX_HINT_PATHS /usr/local/opt/flex /opt/homebrew/opt/flex)

    # 查找 flex 的 include 目录
    find_path(FLEX_INCLUDE_DIR NAMES location.hh PATHS ${FLEX_HINT_PATHS} PATH_SUFFIXES include)
    find_library(FLEX_LIB NAMES fl PATHS ${FLEX_HINT_PATHS} PATH_SUFFIXES lib)

    if(FLEX_INCLUDE_DIR)
        include_directories(${FLEX_INCLUDE_DIR})
        message(STATUS "找到 Flex 头文件目录: ${FLEX_INCLUDE_DIR}")
    endif()

    if(FLEX_LIB)
        message(STATUS "找到 Flex 库: ${FLEX_LIB}")
    else()
        message(WARNING "未找到 Flex 库，尝试使用系统默认路径")
    endif()
    # 设置 bison 路径
    set(BISON_HINT_PATHS /usr/local/opt/bison /opt/homebrew/opt/bison)

    # 查找 bison 的 include 目录
    find_path(BISON_INCLUDE_DIR NAMES location.hh PATHS ${BISON_HINT_PATHS} PATH_SUFFIXES include)
    find_library(BISON_LIB NAMES y PATHS ${BISON_HINT_PATHS} PATH_SUFFIXES lib)

    if(BISON_INCLUDE_DIR)
        include_directories(${BISON_INCLUDE_DIR})
        message(STATUS "找到 Bison 头文件目录: ${BISON_INCLUDE_DIR}")
    endif()

    if(BISON_LIB)
        message(STATUS "找到 Bison 库: ${BISON_LIB}")
    else()
        message(WARNING "未找到 Bison 库，尝试使用系统默认路径")
    endif()

    # 设置编译器标志
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L/usr/local/opt/bison/lib")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L/usr/local/opt/flex/lib")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L/usr/local/opt/bison/lib -L/usr/local/opt/flex/lib")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L/opt/homebrew/opt/flex/lib -L/opt/homebrew/opt/bison/lib")

elseif(UNIX AND NOT APPLE)
    message(STATUS "配置 Linux 环境")
    # Linux 下的配置
    # 通常 bison 会安装在标准路径，不需要特殊配置
    # 包含生成的头文件
endif()
# 源文件
set(LEXER_SOURCE src/lexer.l)
set(PARSER_SOURCE src/parser.y)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)
# 生成文件
set(GENERATED_LEX_C ${CMAKE_CURRENT_BINARY_DIR}/lexer.c)
set(GENERATED_PARSER_C ${CMAKE_CURRENT_BINARY_DIR}/parser.c)
set(GENERATED_PARSER_H ${CMAKE_CURRENT_BINARY_DIR}/parser.h)

# ==================== Flex 目标 ====================
add_custom_command(
        OUTPUT ${GENERATED_LEX_C}
        COMMAND ${FLEX_EXECUTABLE}
        --outfile=${GENERATED_LEX_C}
        ${CMAKE_CURRENT_SOURCE_DIR}/${LEXER_SOURCE}
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${LEXER_SOURCE}
        COMMENT "Generating lexer with flex"
)

# ==================== Bison 目标 ====================
add_custom_command(
        OUTPUT ${GENERATED_PARSER_C} ${GENERATED_PARSER_H}
        COMMAND ${BISON_EXECUTABLE}
        --output=${GENERATED_PARSER_C}
        --defines=${GENERATED_PARSER_H}
        ${CMAKE_CURRENT_SOURCE_DIR}/${PARSER_SOURCE}
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${PARSER_SOURCE}
        COMMENT "Generating parser with bison"
)

# ==================== 可执行文件 ====================
add_executable(parser
        ${GENERATED_LEX_C}
        ${GENERATED_PARSER_C}
        ${CMAKE_CURRENT_SOURCE_DIR}/src/ast.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/display.c
)

# 包含生成的头文件
target_include_directories(parser PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# 链接数学库（如果 %left '+' 等需要）
target_link_libraries(parser PRIVATE fl y)

# Windows MSVC 兼容
if(MSVC)
    target_compile_definitions(parser PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

# ==================== 运行测试（可选）================
add_custom_target(run_test_cases_case1
        COMMAND ${CMAKE_CURRENT_BINARY_DIR}/parser ${CMAKE_CURRENT_SOURCE_DIR}/test_cases/case1.sy
        DEPENDS parser
        COMMENT "Running parser on test_cases/case1.sy"
)