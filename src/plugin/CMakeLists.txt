add_custom_target(plugin)
set(TARGET_NAME amirstan_plugin)
set(SHARED_TARGET ${TARGET_NAME})
set(STATIC_TARGET ${TARGET_NAME}_static)

enable_language(CUDA)
set(TARGET_DIR ${CMAKE_CURRENT_SOURCE_DIR})

include_directories(common)

set(PLUGIN_LIST adaptivePoolPlugin
                batchedNMSPlugin
                carafeFeatureReassemblePlugin
                deformableConvPlugin
                deformablePoolPlugin
                delta2bboxPlugin
                gridAnchorDynamicPlugin
                gridSamplePlugin
                groupNormPlugin
                roiExtractorPlugin
                roiPoolPlugin
                torchBmmPlugin
                torchCumMaxMinPlugin
                torchCumPlugin
                torchEmbeddingPlugin
                torchGatherPlugin
                torchNMSPlugin
                torchUnfoldPlugin
                )

set(INFER_PLUGIN_SRC InferPlugin.cpp)

set(INFER_PLUGIN_LIB ${TENSORRT_LIBRARY})

foreach(PLUGIN_NAME ${PLUGIN_LIST})
add_subdirectory(${PLUGIN_NAME})
set(INFER_PLUGIN_LIB ${INFER_PLUGIN_LIB} ${PLUGIN_NAME}_static)
endforeach()

if (WITH_DEEPSTREAM)                                                                                                        
    MESSAGE(STATUS "Adding NvDsInferParseMmdet")
    set(INFER_PLUGIN_LIB ${INFER_PLUGIN_LIB} ${TENSORRT_LIBRARY_INFER_PLUGIN} )
    set(INFER_PLUGIN_SRC ${INFER_PLUGIN_SRC} NvDsInferParseMmdet.cpp)
endif ()

cuda_add_library(${SHARED_TARGET} SHARED ${INFER_PLUGIN_SRC})
target_link_libraries(${SHARED_TARGET} ${INFER_PLUGIN_LIB})
target_link_libraries(${SHARED_TARGET} cublas)
target_include_directories(${SHARED_TARGET} PRIVATE ${PROJECT_SOURCE_DIR}/common)
