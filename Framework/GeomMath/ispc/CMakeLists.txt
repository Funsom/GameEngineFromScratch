set(FUNCTIONS CrossProduct MulByElement Transpose Normalize
              Transform AddByElement SubByElement MatrixUtil
              InverseMatrix4X4f DCT Absolute Pow DivByElement 
        )

foreach(FUNC IN LISTS FUNCTIONS)
    IF(${APPLE})
        add_custom_command(OUTPUT ${FUNC}.o
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} -o ${FUNC}.o -h ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMAND sed -i '' 's/const uint32_t count/const size_t count/' ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h
            MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMENT "Building ${FUNC}"
            )
    ELSEIF(${CMAKE_HOST_SYSTEM_NAME} MATCHES FreeBSD)
        add_custom_command(OUTPUT ${FUNC}.o
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} -o ${FUNC}.o -h ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMAND gsed -i 's/const uint32_t count/const size_t count/' ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h
            MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMENT "Building ${FUNC}"
            )
    ELSEIF(${CMAKE_HOST_SYSTEM_NAME} MATCHES Windows)
        add_custom_command(OUTPUT ${FUNC}.o
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} -o ${FUNC}.o -h ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMAND sed -i "s/const uint32_t count/const size_t count/" ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h
            COMMAND rm -f ${GEOMMATH_LIB_HEADER_FOLDER}/sed?????? #quick fix for Gnu for Win32 sed
            MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMENT "Building ${FUNC}"
            )
    ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES Android)
        add_custom_command(OUTPUT ${FUNC}.o
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} --emit-llvm -o ${FUNC}.bc -h ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${LLVM_COMPILER_PATH}" llc -O3 -float-abi=soft -march=arm -mcpu=cortex-a9 -thread-model=posix -filetype=obj -o=${FUNC}.o ${FUNC}.bc
            COMMAND sed -i "s/const uint32_t count/const size_t count/" ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h
            MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMENT "Building ${FUNC}"
            )
    ELSE()
        add_custom_command(OUTPUT ${FUNC}.o
            COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} -o ${FUNC}.o -h ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMAND sed -i 's/const uint32_t count/const size_t count/' ${GEOMMATH_LIB_HEADER_FOLDER}/${FUNC}.h
            MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
            COMMENT "Building ${FUNC}"
            )
    ENDIF(${APPLE})

    list(APPEND OBJECTS ${FUNC}.o)

endforeach(FUNC)

IF(${WIN32})
add_custom_command(OUTPUT ${GEOMMATH_LIB_FILE}
        COMMAND ${CMAKE_AR} ${ISPC_LIBRARIAN_OPTIONS} ${OBJECTS}
        COMMAND rm -v ${OBJECTS}
        DEPENDS ${OBJECTS}
        )
ELSE(${WIN32})
add_custom_command(OUTPUT ${GEOMMATH_LIB_FILE}
        COMMAND ${CMAKE_AR} ${ISPC_LIBRARIAN_OPTIONS} ${OBJECTS}
        COMMAND ${CMAKE_RANLIB} ${GEOMMATH_LIB_FILE}
        COMMAND rm -v ${OBJECTS}
        DEPENDS ${OBJECTS}
        )
ENDIF(${WIN32})

add_custom_target(GeomMathLib
                   DEPENDS ${GEOMMATH_LIB_FILE}
        )

