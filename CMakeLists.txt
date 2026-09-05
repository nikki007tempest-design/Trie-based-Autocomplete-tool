cmake_minimum_required(VERSION 3.14)
project(TrieAutocomplete CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Threads REQUIRED)

add_executable(trie_server
    src/main.cpp
    src/Trie.cpp
)

target_include_directories(trie_server PRIVATE include)
target_link_libraries(trie_server PRIVATE Threads::Threads)

# Copy runtime assets (frontend + data) next to the built binary so
# relative paths in main.cpp ("./frontend", "data/words.txt") resolve.
add_custom_command(TARGET trie_server POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/frontend $<TARGET_FILE_DIR:trie_server>/frontend
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        ${CMAKE_SOURCE_DIR}/data $<TARGET_FILE_DIR:trie_server>/data
)
