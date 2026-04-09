cmake_minimum_required(VERSION 3.10)
project(SimpleGrammarLib)

# 1. Ensure tools are found if not provided by GitHub Actions
if(NOT BISON_TOOL)
    find_program(BISON_TOOL win_bison)
endif()
if(NOT FLEX_TOOL)
    find_program(FLEX_TOOL win_flex)
endif()

message("bison_tool = ${BISON_TOOL}")
message("flex_tool  = ${FLEX_TOOL}")

# 2. Fix add_library syntax and file list
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
    "${PROJECT_SOURCE_DIR}/chunker_yacc.cpp"
    "${PROJECT_SOURCE_DIR}/chunker_lex.cpp"
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

target_link_libraries(SimpleGrammarLib
    aot_common
    LemmatizerLib
)

if (WIN32) 
    # Ensure we can find headers if winflexbison uses a specific folder
    if(WIN_FLEX_FOLDER)
        include_directories(AFTER ${WIN_FLEX_FOLDER})
    endif()
endif()

# 3. Custom command for Bison with quoted paths
add_custom_command(
    OUTPUT "${PROJECT_SOURCE_DIR}/chunker_yacc.cpp" "${PROJECT_SOURCE_DIR}/chunker_yacc.hpp"
    DEPENDS "${PROJECT_SOURCE_DIR}/chunker.y"
    COMMAND "${BISON_TOOL}" -d -ochunker_yacc.cpp "${PROJECT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    VERBATIM
)

# 4. Custom command for Flex with quoted paths
add_custom_command(
    OUTPUT "${PROJECT_SOURCE_DIR}/chunker_lex.cpp"
    DEPENDS "${PROJECT_SOURCE_DIR}/chunker.l" "${PROJECT_SOURCE_DIR}/chunker_yacc.hpp"
    COMMAND "${FLEX_TOOL}" -ochunker_lex.cpp -d "${PROJECT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    VERBATIM
)

add_subdirectory(tests)
