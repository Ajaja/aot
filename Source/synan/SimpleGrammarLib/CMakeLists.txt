cmake_minimum_required(VERSION 3.10)
project(SimpleGrammarLib)

find_program(BISON_EXE NAMES win_bison bison REQUIRED)
find_program(FLEX_EXE NAMES win_flex flex REQUIRED)

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

add_custom_command(
    OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.cpp" 
           "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.hpp"
    COMMAND "${BISON_EXE}" -d -o"${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating chunker_yacc.cpp and hpp using Bison"
    VERBATIM
)

add_custom_command(
    OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/chunker_lex.cpp"
    COMMAND "${FLEX_EXE}" -o"${CMAKE_CURRENT_SOURCE_DIR}/chunker_lex.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    COMMENT "Generating chunker_lex.cpp using Flex"
    VERBATIM
)

add_subdirectory(tests)
