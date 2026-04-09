declare_cmake_min_version()

project(SimpleGrammarLib)


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
		${PROJECT_SOURCE_DIR}/chunker_yacc.cpp
		${PROJECT_SOURCE_DIR}/chunker_lex.cpp
        AhoKorasick.h
        GLR.h
        GLRTable.h
        GrammarItem.h
        InputSymbol.h
		LR-items.h
		my_stl.h
		SimpleGrammar.h
		SimpleGrammarLib.h
 "InputSymbol.cpp")

target_link_libraries(SimpleGrammarLib
    aot_common
    LemmatizerLib
)

if (WIN32) 
    include_directories (AFTER ${WIN_FLEX_FOLDER})
endif()

if(NOT BISON_TOOL)
    find_program(BISON_TOOL win_bison)
endif()
if(NOT FLEX_TOOL)
    find_program(FLEX_TOOL win_flex)
endif()

message ("bison_tool = " ${BISON_TOOL})
add_custom_command(
    OUTPUT "${PROJECT_SOURCE_DIR}/chunker_yacc.cpp" "${PROJECT_SOURCE_DIR}/chunker_yacc.hpp"
    DEPENDS "${PROJECT_SOURCE_DIR}/chunker.y"
    COMMAND "${BISON_TOOL}" -d -ochunker_yacc.cpp "${PROJECT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    VERBATIM
)

add_custom_command(
    OUTPUT "${PROJECT_SOURCE_DIR}/chunker_lex.cpp"
    DEPENDS "${PROJECT_SOURCE_DIR}/chunker.l" "${PROJECT_SOURCE_DIR}/chunker_yacc.hpp"
    COMMAND "${FLEX_TOOL}" -ochunker_lex.cpp -d "${PROJECT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
    VERBATIM
)

add_subdirectory(tests)



