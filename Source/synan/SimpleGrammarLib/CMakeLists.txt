cmake_minimum_required(VERSION 3.10)
project(SimpleGrammarLib)

# 1. Resolve absolute paths for Bison and Flex
# Using REQUIRED ensures the configuration fails early with a clear message 
# if the tools are not found in the PATH.
find_program(BISON_EXE NAMES win_bison bison REQUIRED)
find_program(FLEX_EXE NAMES win_flex flex REQUIRED)

# Print paths to the log for verification
message(STATUS "Found Bison: ${BISON_EXE}")
message(STATUS "Found Flex:  ${FLEX_EXE}")

# 2. Define generated files as absolute paths to avoid MSBuild resolution errors
set(GEN_YACC_CPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.cpp")
set(GEN_YACC_HPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.hpp")
set(GEN_LEX_CPP  "${CMAKE_CURRENT_SOURCE_DIR}/chunker_lex.cpp")

# 3. Custom command for Bison
# We use -o with the absolute path to ensure output goes exactly where expected.
add_custom_command(
    OUTPUT "${GEN_YACC_CPP}" "${GEN_YACC_HPP}"
    COMMAND "${BISON_EXE}" -d -o "${GEN_YACC_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating Bison parser (chunker_yacc.cpp/hpp)"
    VERBATIM
)

# 4. Custom command for Flex
add_custom_command(
    OUTPUT "${GEN_LEX_CPP}"
    COMMAND "${FLEX_EXE}" -o "${GEN_LEX_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating Flex lexer (chunker_lex.cpp)"
    VERBATIM
)

# 5. Define the library
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

# 6. Include directories
# Ensure the library can find its own generated headers (chunker_yacc.hpp)
target_include_directories(SimpleGrammarLib 
    PUBLIC 
        "${CMAKE_CURRENT_SOURCE_DIR}"
)

# 7. Linking
target_link_libraries(SimpleGrammarLib
    aot_common
    LemmatizerLib
)

# 8. Platform specific settings
if (WIN32) 
    # Ensure standard Windows defines are present
    target_compile_definitions(SimpleGrammarLib PRIVATE WIN32 _WINDOWS)
    
    # If winflexbison needs to find its own headers like FlexLexer.h
    find_path(FLEX_INCLUDE_DIR NAMES FlexLexer.h)
    if(FLEX_INCLUDE_DIR)
        target_include_directories(SimpleGrammarLib PRIVATE "${FLEX_INCLUDE_DIR}")
    endif()
endif()
