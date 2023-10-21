cmake_minimum_required(VERSION 3.20)

include(compiler_base.cmake)

# ------------- Project -----------------
#--------------

project(PTM)

# ------------- Sub module
#--------------

# Other library
add_subdirectory(concurrentqueue)

find_package(TBB CONFIG REQUIRED)

# My library
add_subdirectory(workload)
add_subdirectory(concurrent_control)
add_subdirectory(storage_manager)
add_subdirectory(transaction_manager)
add_subdirectory(util)

include(auto_test.cmake)

# other single tests
add_subdirectory(test)

# ------------- Main module
#--------------

FILE(GLOB_RECURSE header_files CONFIGURE_DEPENDS include/*.hpp include/*.h)
FILE(GLOB_RECURSE source_files CONFIGURE_DEPENDS src/*.cpp)

#message(STATUS "${PROJECT_NAME} Configuration")
#message(STATUS "---- Header Files:")
#    foreach(header ${header_files})
#        message(STATUS "----\t ${header}")
#    endforeach()
#message(STATUS "---- Source Files:")
#    foreach(source ${source_files})
#        message(STATUS "----\t ${source}")
#    endforeach()

# Main Function
add_executable(${PROJECT_NAME} ${header_files} ${source_files})

target_include_directories(${PROJECT_NAME}
        PUBLIC ${PROJECT_SOURCE_DIR}/include
        PUBLIC concurrentqueue)

# ------------- Library linkage
#--------------

target_link_libraries(${PROJECT_NAME}
        # Outer library
        PUBLIC pthread
        PUBLIC atomic
        PUBLIC gtest
        PUBLIC pmemobj
        PUBLIC TBB::tbb
        PUBLIC jemalloc
        PUBLIC numa
        # Profiler
        PUBLIC profiler
        PUBLIC papi

        PUBLIC concurrentqueue
        # User library
        PUBLIC workload
        PUBLIC concurrent_control
        PUBLIC storage_manager
        PUBLIC transaction_manager
        PUBLIC util)