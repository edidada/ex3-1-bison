cmake_minimum_required(VERSION 3.14)
project(ch1-05 LANGUAGES C)

set(CMAKE_C_STANDARD 11)

# 查找工具
find_program(FLEX_EXECUTABLE flex REQUIRED)
find_program(BISON_EXECUTABLE bison REQUIRED)

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
)

# 包含生成的头文件
target_include_directories(parser PRIVATE
        ${CMAKE_CURRENT_BINARY_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}
)

# 链接数学库（如果 %left '+' 等需要）
target_link_libraries(parser PRIVATE l y m)

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