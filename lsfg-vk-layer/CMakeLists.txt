set(LAYER_SOURCES
    "src/entrypoint.cpp"
    "src/instance.cpp"
    "src/swapchain.cpp")

add_library(lsfg-vk-layer SHARED ${LAYER_SOURCES})

target_link_libraries(lsfg-vk-layer
    PUBLIC lsfg-vk-common
    PUBLIC lsfg-vk-backend)

target_compile_options(lsfg-vk-layer PRIVATE
    -Wno-unknown-warning-option
    -Wno-unsafe-buffer-usage) # Array indexing

set_target_properties(lsfg-vk-layer PROPERTIES
    CXX_VISIBILITY_PRESET hidden)

# On macOS, do not set SONAME / version – just a plain .dylib
if(APPLE)
    set_target_properties(lsfg-vk-layer PROPERTIES
        SUFFIX ".dylib"
        PREFIX "lib")
endif()

configure_file(
    "VkLayer_LSFGVK_frame_generation.json.in"
    "VkLayer_LSFGVK_frame_generation.json"
    @ONLY)

install(TARGETS lsfg-vk-layer
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")

if(APPLE)
    # On macOS the Vulkan loader searches for layer manifests in:
    #   ~/Library/Preferences/Khronos/vulkan/explicit_layer.d  (explicit)
    #   /etc/vulkan/implicit_layer.d                            (implicit / system)
    # We install into ${CMAKE_INSTALL_DATAROOTDIR}/vulkan/implicit_layer.d
    # which maps to /usr/local/share/vulkan/implicit_layer.d when using a
    # standard CMAKE_INSTALL_PREFIX.  Users can also set VK_LAYER_PATH at
    # runtime to point at the build directory.
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/VkLayer_LSFGVK_frame_generation.json"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/vulkan/implicit_layer.d")
else()
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/VkLayer_LSFGVK_frame_generation.json"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/vulkan/implicit_layer.d")
endif()
