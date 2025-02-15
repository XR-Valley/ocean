# Copyright (c) Meta Platforms, Inc. and affiliates.
#
# This source code is licensed under the MIT license found in the
# LICENSE file in the root directory of this source tree.

cmake_minimum_required(VERSION 3.26)

if (ANDROID OR IOS OR MACOS)

    set(OCEAN_TARGET_NAME "application_ocean_demo_tracking_featuretracker")

    # Source files
    file(GLOB OCEAN_TARGET_HEADER_FILES "${CMAKE_CURRENT_LIST_DIR}/*.h")
    file(GLOB OCEAN_TARGET_SOURCE_FILES "${CMAKE_CURRENT_LIST_DIR}/*.cpp")

    # Target definition
    add_library(${OCEAN_TARGET_NAME} ${OCEAN_TARGET_SOURCE_FILES} ${OCEAN_TARGET_HEADER_FILES})

    target_include_directories(${OCEAN_TARGET_NAME} PUBLIC ${OCEAN_IMPL_DIR})

    target_compile_definitions(${OCEAN_TARGET_NAME} PUBLIC ${OCEAN_PREPROCESSOR_FLAGS})
    target_compile_options(${OCEAN_TARGET_NAME} PUBLIC ${OCEAN_COMPILER_FLAGS})

    # Dependencies
    target_link_libraries(${OCEAN_TARGET_NAME}
        PUBLIC
            ocean_base
            ocean_devices
            ocean_io
            ocean_math
            ocean_media
            ocean_platform
            ocean_tracking
            ocean_tracking_orb
            ocean_tracking_pattern
    )

    if (MACOS OR IOS)
        target_link_libraries(${OCEAN_TARGET_NAME}
            PUBLIC
                ocean_media_avfoundation
                ocean_media_imageio
        )
    endif()

    if (IOS)
        target_link_libraries(${OCEAN_TARGET_NAME}
            PUBLIC
                ocean_devices_ios
        )
    endif()

    if (ANDROID)
        target_link_libraries(${OCEAN_TARGET_NAME}
            PUBLIC
                ocean_devices_android
                ocean_media_openimagelibraries
        )
    endif()

    if (WIN32)
        target_link_libraries(${OCEAN_TARGET_NAME}
            PUBLIC
                ocean_media_directshow
                ocean_media_mediafoundation
                ocean_media_wic
        )
    endif()

    # Installation
    install(TARGETS ${OCEAN_TARGET_NAME}
        DESTINATION ${CMAKE_INSTALL_LIBDIR}
        COMPONENT lib
    )

    install(FILES ${OCEAN_TARGET_HEADER_FILES}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include/application/ocean/demo/tracking/featuretracker
        COMPONENT include
    )

endif()

if (ANDROID)

    set(OCEAN_TARGET_NAME "application_ocean_demo_tracking_featuretracker_android_native")

    file(GLOB OCEAN_TARGET_HEADER_FILES "${CMAKE_CURRENT_LIST_DIR}/android/*.h")
    file(GLOB OCEAN_TARGET_SOURCE_FILES "${CMAKE_CURRENT_LIST_DIR}/android/*.cpp")

    add_library(${OCEAN_TARGET_NAME} SHARED ${OCEAN_TARGET_SOURCE_FILES} ${OCEAN_TARGET_HEADER_FILES})

    target_include_directories(${OCEAN_TARGET_NAME} PRIVATE "${OCEAN_IMPL_DIR}")

    target_compile_definitions(${OCEAN_TARGET_NAME} PRIVATE ${OCEAN_PREPROCESSOR_FLAGS} "-DOCEAN_RUNTIME_SHARED")
    target_compile_options(${OCEAN_TARGET_NAME}
        PUBLIC
            ${OCEAN_COMPILER_FLAGS}

        PRIVATE
            "-fexceptions"
    )

    # The output name of this target must match the one used in `android/FeatureTrackerActivity.java`
    set_target_properties(${OCEAN_TARGET_NAME} PROPERTIES OUTPUT_NAME OceanDemoTrackingFeatureTracker)

    target_link_libraries(${OCEAN_TARGET_NAME}
        PUBLIC
            application_ocean_demo_tracking_featuretracker
            ocean_base
            ocean_media
            ocean_media_android
            ocean_platform_android
            ocean_rendering_glescenegraph
            "-Wl,--whole-archive"
            ocean_base_jni
            ocean_media_android_jni
            ocean_platform_android_application_jni
            ocean_rendering_glescenegraph_jni
            "-Wl,--no-whole-archive"
    )

    # Installation
    install(TARGETS ${OCEAN_TARGET_NAME}
        DESTINATION ${CMAKE_INSTALL_LIBDIR}
        COMPONENT lib
    )

    install(FILES ${OCEAN_TARGET_HEADER_FILES}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/include/application/ocean/demo/tracking/featuretracker/android
        COMPONENT include
    )

endif()

if (MACOS)

    set(OCEAN_TARGET_NAME "application_ocean_demo_tracking_featuretracker_osx")

    # Source files
    file(GLOB OCEAN_TARGET_HEADER_FILES "${CMAKE_CURRENT_LIST_DIR}/osx/*.h")
    file(GLOB OCEAN_TARGET_SOURCE_FILES "${CMAKE_CURRENT_LIST_DIR}/osx/*.mm")

    # Target definition
    add_executable(${OCEAN_TARGET_NAME} MACOSX_BUNDLE ${OCEAN_TARGET_SOURCE_FILES} ${OCEAN_TARGET_HEADER_FILES})

    set(MACOSX_DEPLOYMENT_TARGET "10.12")
    set(PRODUCT_BUNDLE_IDENTIFIER "com.meta.ocean.application.demo.tracking.featuretracker.osx")
    set(EXECUTABLE_NAME ${OCEAN_TARGET_NAME})
    set(PRODUCT_NAME ${OCEAN_TARGET_NAME})
    configure_file(${CMAKE_CURRENT_LIST_DIR}/osx/Info.plist ${CMAKE_CURRENT_BINARY_DIR})
    set_target_properties(${OCEAN_TARGET_NAME} PROPERTIES
        BUNDLE True
        MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_BINARY_DIR}/Info.plist"
    )

    target_include_directories(${OCEAN_TARGET_NAME} PRIVATE ${OCEAN_IMPL_DIR})

    target_compile_definitions(${OCEAN_TARGET_NAME} PRIVATE ${OCEAN_PREPROCESSOR_FLAGS})
    target_compile_options(${OCEAN_TARGET_NAME} PRIVATE ${OCEAN_COMPILER_FLAGS})

    # Dependencies
    target_link_libraries(${OCEAN_TARGET_NAME}
        PRIVATE
            application_ocean_demo_tracking_featuretracker
            ocean_base
            ocean_platform_apple_macos
    )

    # Custom command to compile Application.xib to Application.nib
    set(APPLICATION_XIB "${OCEAN_IMPL_DIR}/../res/application/ocean/demo/tracking/featuretracker/osx/Application.xib")
    add_custom_command(TARGET ${OCEAN_TARGET_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/Resources
        COMMAND ${CMAKE_COMMAND} -E env "IBTOOL=${CMAKE_PROGRAM_PATH}" "${CMAKE_COMMAND}" -E remove ${CMAKE_CURRENT_BINARY_DIR}/Resources/Application.nib
        COMMAND ${CMAKE_COMMAND} -E env "IBTOOL=${CMAKE_PROGRAM_PATH}" "${CMAKE_COMMAND}" -E echo "Compiling XIB file..."
        COMMAND ${CMAKE_COMMAND} -E env "IBTOOL=${CMAKE_PROGRAM_PATH}" ibtool --warnings --notices --errors --output-format human-readable-text --compile ${CMAKE_CURRENT_BINARY_DIR}/Resources/Application.nib "${APPLICATION_XIB}"
        COMMENT "Compiling XIB file..."
        VERBATIM
    )

    # Installation
    install(TARGETS ${OCEAN_TARGET_NAME} DESTINATION bin)

    # Install Application.nib
    install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/Resources DESTINATION bin/${OCEAN_TARGET_NAME}.app/Contents)

    # Install needed dependencies in app bundle.  Overwrite app's code signature as workaround for invalid signature error.
    install(CODE "
        include(BundleUtilities)
        fixup_bundle(\"${CMAKE_INSTALL_PREFIX}/bin/${OCEAN_TARGET_NAME}.app\" \"\" \"${OCEAN_MACOSX_BUNDLE_LIBRARY_PATHS}\")
        execute_process(COMMAND codesign -s - -f --deep \"${CMAKE_INSTALL_PREFIX}/bin/${OCEAN_TARGET_NAME}.app\")
        " COMPONENT Runtime)

endif()

# TODO: enable WIN32 configuration when "ocean_media_directshow" becomes available
#
#if (WIN32)
#
#    set(OCEAN_TARGET_NAME "application_ocean_demo_tracking_featuretracker_win")
#
#    # Source files
#    file(GLOB OCEAN_TARGET_HEADER_FILES "${CMAKE_CURRENT_LIST_DIR}/win/*.h")
#    file(GLOB OCEAN_TARGET_SOURCE_FILES "${CMAKE_CURRENT_LIST_DIR}/win/*.cpp")
#
#    # Target definition
#    add_executable(${OCEAN_TARGET_NAME} ${OCEAN_TARGET_SOURCE_FILES} ${OCEAN_TARGET_HEADER_FILES})
#
#    target_include_directories(${OCEAN_TARGET_NAME} PRIVATE ${OCEAN_IMPL_DIR})
#
#    target_compile_definitions(${OCEAN_TARGET_NAME} PUBLIC ${OCEAN_PREPROCESSOR_FLAGS} -DUSE_OCEAN_DEVICES_VRS)
#    target_compile_options(${OCEAN_TARGET_NAME} PUBLIC ${OCEAN_COMPILER_FLAGS})
#
#    # Dependencies
#    target_link_libraries(${OCEAN_TARGET_NAME}
#        PUBLIC
#            application_ocean_demo_tracking_featuretracker
#            ocean_base
#            ocean_platform
#            ocean_platform_win
#    )
#
#    # Installation
#    install(TARGETS ${OCEAN_TARGET_NAME} DESTINATION bin)
#
#endif()


if (IOS)

    set(OCEAN_TARGET_NAME "application_ocean_demo_tracking_featuretracker_ios")

    find_library(UIKIT_LIB UiKit)

    file(GLOB OCEAN_TARGET_HEADER_FILES
        "${CMAKE_CURRENT_LIST_DIR}/*.h"
        "${CMAKE_CURRENT_LIST_DIR}/ios/*.h"
    )
    file(GLOB OCEAN_TARGET_SOURCE_FILES
        "${CMAKE_CURRENT_LIST_DIR}/*.cpp"
        "${CMAKE_CURRENT_LIST_DIR}/ios/*.mm"
    )

    file(GLOB OCEAN_TARGET_STORYBOARDS "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/ios/*.xib")
    file(GLOB OCEAN_TARGET_IMAGE_RESOURCES
        "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/*.bmp"
        "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/*.jpg"
        "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/*.obj"
        "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/*.png"
        "${CMAKE_SOURCE_DIR}/res/application/ocean/demo/tracking/featuretracker/*.txt"
    )

    add_executable(${OCEAN_TARGET_NAME}
        ${OCEAN_TARGET_HEADER_FILES}
        ${OCEAN_TARGET_SOURCE_FILES}
        ${OCEAN_TARGET_STORYBOARDS}
        ${OCEAN_TARGET_IMAGE_RESOURCES}
    )

    set_source_files_properties(${OCEAN_TARGET_STORYBOARDS} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    set_source_files_properties(${OCEAN_TARGET_IMAGE_RESOURCES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

    set(ALL_APP_RESOURCES
        ${OCEAN_TARGET_STORYBOARDS}
        ${OCEAN_TARGET_IMAGE_RESOURCES}
    )

    set_target_properties(${OCEAN_TARGET_NAME}
        PROPERTIES
            RESOURCE "${ALL_APP_RESOURCES}"
            MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_LIST_DIR}/ios/Info.plist"
    )

    set_xcode_property(${OCEAN_TARGET_NAME} PRODUCT_BUNDLE_IDENTIFIER "com.meta.ocean.app.demo.tracking.featuretracker.ios" All)

    target_include_directories(${OCEAN_TARGET_NAME}
        PUBLIC
            ${OCEAN_IMPL_DIR}
        PRIVATE
            ${CMAKE_CURRENT_LIST_DIR}
    )

    target_link_libraries(${OCEAN_TARGET_NAME}
        PUBLIC
            ocean_base
            ocean_devices
            ocean_devices_ios
            ocean_io
            ocean_math
            ocean_media
            ocean_media_avfoundation
            ocean_media_imageio
            ocean_platform
            ocean_platform_apple
            ocean_platform_apple_ios
            ocean_rendering_glescenegraph_apple
            ocean_tracking
            ocean_tracking_orb
            ocean_tracking_pattern
            ${UIKIT_LIB}
    )
endif()
