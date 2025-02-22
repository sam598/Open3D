# build
set(GUI_SOURCE_FILES
    Application.cpp
    BitmapWindowSystem.cpp
    Button.cpp
    Checkbox.cpp
    Color.cpp
    ColorEdit.cpp
    Combobox.cpp
    Dialog.cpp
    FileDialog.cpp
    FileDialogNative.cpp
    GLFWWindowSystem.cpp
    Gui.cpp
    ImageLabel.cpp
    ImguiFilamentBridge.cpp
    Label.cpp
    Label3D.cpp
    Layout.cpp
    ListView.cpp
    Menu.cpp
    MenuImgui.cpp
    NumberEdit.cpp
    PickPointsInteractor.cpp
    ProgressBar.cpp
    SceneWidget.cpp
    Slider.cpp
    StackedWidget.cpp
    TabControl.cpp
    Task.cpp
    TextEdit.cpp
    Theme.cpp
    TreeView.cpp
    UIImage.cpp
    Util.cpp
    VectorEdit.cpp
    Widget.cpp
    Window.cpp
    )

if (WIN32)
    list(APPEND GUI_SOURCE_FILES NativeWin32.cpp)
elseif (APPLE)
    list(APPEND GUI_SOURCE_FILES NativeMacOS.mm)
    list(APPEND GUI_HEADER_FILES MenuMacOS.h)
    list(APPEND GUI_SOURCE_FILES MenuMacOS.mm)
else ()
    list(APPEND GUI_SOURCE_FILES NativeLinux.cpp)
endif()

# create object library
add_library(GUI OBJECT ${GUI_SOURCE_FILES})
open3d_show_and_abort_on_warning(GUI)
open3d_set_global_properties(GUI)
open3d_link_3rdparty_libraries(GUI)

# --- build resources ----
file(GLOB GUI_RESOURCE_SOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/Resources/*)
file(GLOB GUI_MATERIAL_SOURCE_FILES "Materials/*.mat")

# copy GUI/Resources -> <output>/resources
set(GUI_RESOURCE_DIR ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/resources)
file(MAKE_DIRECTORY ${GUI_RESOURCE_DIR})
file(COPY ${GUI_RESOURCE_SOURCE_FILES}
     DESTINATION ${GUI_RESOURCE_DIR})

# Compile .mat files and copy to resources directory in output directory
if (IOS OR ANDROID)
    set(GUI_MATC_ARGS --platform mobile)
elseif (NOT WIN32)
    set(GUI_MATC_ARGS --platform desktop)
endif()

set(GUI_MATERIAL_COMPILED_FILES "")
foreach(matsrc IN ITEMS ${GUI_MATERIAL_SOURCE_FILES})
    string(REGEX REPLACE "\\.[^.]*$" ".filamat" matdest ${matsrc})
    string(REPLACE "${CMAKE_CURRENT_SOURCE_DIR}/Materials/" "${GUI_RESOURCE_DIR}/" matdest ${matdest})
    list(APPEND GUI_MATERIAL_COMPILED_FILES ${matdest})
    add_custom_command(OUTPUT ${matdest}
                       COMMAND ${FILAMENT_MATC} ${GUI_MATC_ARGS} -o ${matdest} ${matsrc}
		       MAIN_DEPENDENCY ${matsrc} DEPENDS ${FILAMENT_TARGET})
endforeach()

add_custom_target(GuiResources
                  DEPENDS ${GUI_MATERIAL_COMPILED_FILES})
add_dependencies(GUI GuiResources)

# Export GUI_RESOURCE_FILES to parent CMake context (cpp/open3d/)
set(GUI_RESOURCE_FILES
    ${GUI_RESOURCE_SOURCE_FILES} ${GUI_MATERIAL_COMPILED_FILES}
    PARENT_SCOPE)
set(GUI_RESOURCE_DIR ${GUI_RESOURCE_DIR} PARENT_SCOPE)
