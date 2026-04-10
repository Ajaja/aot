cmake_minimum_required(VERSION 3.10)
project(SimpleGrammarLib)

# 1. Locate Bison and Flex executables
find_program(BISON_EXE NAMES win_bison bison REQUIRED)
find_program(FLEX_EXE NAMES win_flex flex REQUIRED)

# 2. Automatically find the Flex header directory (FlexLexer.h)
# On Windows/Choco, FlexLexer.h is usually in the same folder as win_flex.exe
get_filename_component(FLEX_EXE_DIR "${FLEX_EXE}" DIRECTORY)
find_path(FLEX_INCLUDE_DIR 
    NAMES FlexLexer.h 
    HINTS "${FLEX_EXE_DIR}"
    DOC "Path to FlexLexer.h"
)

message(STATUS "Found Bison: ${BISON_EXE}")
message(STATUS "Found Flex:  ${FLEX_EXE}")
message(STATUS "Flex Include Dir: ${FLEX_INCLUDE_DIR}")

# 3. Define absolute paths for generated files
set(GEN_YACC_CPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.cpp")
set(GEN_YACC_HPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.hpp")
set(GEN_LEX_CPP  "${CMAKE_CURRENT_SOURCE_DIR}/chunker_lex.cpp")

# 4. Custom command for Bison
add_custom_command(
    OUTPUT "${GEN_YACC_CPP}" "${GEN_YACC_HPP}"
    COMMAND "${BISON_EXE}" -d -o "${GEN_YACC_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating Bison parser"
    VERBATIM
)

# 5. Custom command for Flex
add_custom_command(
    OUTPUT "${GEN_LEX_CPP}"
    COMMAND "${FLEX_EXE}" -o "${GEN_LEX_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating Flex lexer"
    VERBATIM
)

# 6. Define the library target
add_library(SimpleGrammarLib
    AhoKorasick.cpp
    ChunkNode.cpp
    ChunkParser.cpp
    FirstAndFollow.cpp
    GLR.cpp
    GLRTable.cpp
    GetMaxCoverage.cpp
    GrammarItem.cpp
    LR-items.cpp
    Loader.cpp
    Parser.cpp
    SimpleGrammarLib.cpp
    SolvePeriodAmbiguity.cpp
    SourceGrammarParser.cpp
    InputSymbol.cpp
    # Include generated files
    "${GEN_YACC_CPP}"
    "${GEN_LEX_CPP}"
    # Headers
    AhoKorasick.h
    GLR.h
    GLRTable.h
    GrammarItem.h
    InputSymbol.h
    LR-items.h
    my_stl.h
    SimpleGrammar.h
    SimpleGrammarLib.h
)

# 7. Add Include Directories
target_include_directories(SimpleGrammarLib 
    PUBLIC 
        "${CMAKE_CURRENT_SOURCE_DIR}"
    PRIVATE
        "${FLEX_INCLUDE_DIR}"  # This fixes the C1083 'FlexLexer.h' error
)

# 8. Link Dependencies
target_link_libraries(SimpleGrammarLib
    aot_common
    LemmatizerLib
)

# 9. Platform Specific Definitions
if (WIN32)
    target_compile_definitions(SimpleGrammarLib PRIVATE WIN32 _WINDOWS)
endif()
