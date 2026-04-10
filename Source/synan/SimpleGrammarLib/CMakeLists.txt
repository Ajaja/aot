cmake_minimum_required(VERSION 3.10)
project(SimpleGrammarLib)

# 1. Ensure tools are found if not provided by GitHub Actions
if(NOT BISON_TOOL)
    find_program(BISON_TOOL NAMES win_bison bison)
endif()
if(NOT FLEX_TOOL)
    find_program(FLEX_TOOL NAMES win_flex flex)
endif()

# Check if they were actually found to prevent silent failures later
if(NOT BISON_TOOL OR NOT FLEX_TOOL)
    message(FATAL_ERROR "Bison or Flex not found! BISON_TOOL=${BISON_TOOL}, FLEX_TOOL=${FLEX_TOOL}")
endif()

message(STATUS "Using Bison: ${BISON_TOOL}")
message(STATUS "Using Flex:  ${FLEX_TOOL}")

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
