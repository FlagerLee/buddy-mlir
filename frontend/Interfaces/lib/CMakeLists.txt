if (${BUDDY_DIP_OPT_STRIP_MINING})
    set(SPLITING_SIZE ${BUDDY_DIP_OPT_STRIP_MINING})
elseif (HAVE_AVX512)
    set(SPLITING_SIZE 256)
elseif (HAVE_AVX2)
    set(SPLITING_SIZE 16)
elseif (HAVE_SSE)
    set(SPLITING_SIZE 64)
elseif (HAVE_NEON)
    set(SPLITING_SIZE 64)
endif ()

# find openmp
find_package(OpenMP REQUIRED)
if(OPENMP_FOUND)
    message("Found OpenMP")
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS})
    set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS})
    set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS})
    set(lower_flags
            -lower-dip="DIP-strip-mining=${SPLITING_SIZE}"
            -arith-expand
            -lower-affine
            -convert-scf-to-openmp
            -convert-async-to-llvm
            -convert-memref-to-llvm
            -convert-scf-to-cf
            -convert-openmp-to-llvm
            -convert-math-to-llvm
            -convert-vector-to-llvm
            -convert-func-to-llvm
            -reconcile-unrealized-casts)
else()
    set(lower_flags
            lower-dip="DIP-strip-mining=${SPLITING_SIZE}"
            -async-parallel-for
            -async-to-async-runtime
            -async-runtime-ref-counting
            -async-runtime-ref-counting-opt
            -arith-expand
            -lower-affine
            -convert-async-to-llvm
            -convert-memref-to-llvm
            -convert-scf-to-cf
            -convert-math-to-llvm
            -convert-vector-to-llvm
            -convert-func-to-llvm
            -reconcile-unrealized-casts)
endif()

add_custom_command(OUTPUT DIP.o
        POST_BUILD
        COMMAND ${BUDDY_BINARY_DIR}/buddy-opt ${CMAKE_CURRENT_SOURCE_DIR}/DIP.mlir
        ${lower_flags} |
        ${LLVM_MLIR_BINARY_DIR}/mlir-translate --mlir-to-llvmir |
        ${LLVM_MLIR_BINARY_DIR}/llc
        -mtriple=${BUDDY_TARGET_TRIPLE}
        -mattr=${BUDDY_OPT_ATTR}
        --filetype=obj
        -o ${CMAKE_CURRENT_BINARY_DIR}/DIP.o
        )

add_library(BuddyLibDIP STATIC DIP.o)

SET_TARGET_PROPERTIES(BuddyLibDIP PROPERTIES
  LINKER_LANGUAGE C
  ARCHIVE_OUTPUT_DIRECTORY ${LIBRARY_OUTPUT_DIRECTORY}
  )
