declare_cmake_min_version()

project(SimpleGrammarLib)

# 1. Locate Bison and Flex
find_program(BISON_EXE NAMES win_bison bison REQUIRED)
find_program(FLEX_EXE NAMES win_flex flex REQUIRED)

# 2. Robust search for FlexLexer.h
get_filename_component(FLEX_EXE_DIR "${FLEX_EXE}" DIRECTORY)
find_path(FLEX_INCLUDE_DIR 
    NAMES FlexLexer.h 
    HINTS "${FLEX_EXE_DIR}/../include" "${FLEX_EXE_DIR}"
    PATHS "C:/ProgramData/chocolatey/lib/winflexbison3/tools"
)

# 3. Define absolute paths for generated files
set(GEN_YACC_CPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.cpp")
set(GEN_YACC_HPP "${CMAKE_CURRENT_SOURCE_DIR}/chunker_yacc.hpp")
set(GEN_LEX_CPP  "${CMAKE_CURRENT_SOURCE_DIR}/chunker_lex.cpp")

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
        "${GEN_YACC_CPP}"
        "${GEN_LEX_CPP}"
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

target_include_directories(SimpleGrammarLib 
    PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}"
    PRIVATE "${FLEX_INCLUDE_DIR}"
)

if (WIN32) 
    include_directories (AFTER ${WIN_FLEX_FOLDER})
endif()

add_custom_command(
    OUTPUT "${GEN_YACC_CPP}" "${GEN_YACC_HPP}"
    COMMAND "${BISON_EXE}" -d -o "${GEN_YACC_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.y"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    VERBATIM
)

# 5. Flex Generation
add_custom_command(
    OUTPUT "${GEN_LEX_CPP}"
    COMMAND "${FLEX_EXE}" -o "${GEN_LEX_CPP}" "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/chunker.l"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
    VERBATIM
)

add_subdirectory(tests)



