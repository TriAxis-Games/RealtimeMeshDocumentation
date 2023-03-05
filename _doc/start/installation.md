---
title: Installation
sections:
  - Installation from Marketplace
  - Installation from GitHub
  - 'Using RMC From Code'
  - 'Using RMC from Blueprint'
---

### Installation from Marketplace

1. Like any other plugin from the Marketplace, you will need to install it to the engine first through the marketplace and Epic Games Launcher.
   
2. For Blueprint usage, you're basically ready to go!

3. For C++ usage, you will want to follow the steps [Below](#using-rmc-from-code) to get C++ access!

### Installation from GitHub

1. You will need to start with a code project. This does mean you'll need all the build dependencies necessary to create c++ projects in Unreal. However this doesn't mean you'll need to use C++ at all, just that the engine needs to be able to build the plugin. To turn a blueprint project into a code project, you can go to "Add New C++ Class" in the editor and it should set everything up for you!

2. You can download the RMC from one of the links above.

3. Navigate to your project folder, and assuming the folder doesn't already exist, create a 'Plugins' folder in the root of your project.

4. Within the plugins folder create a folder called RealtimeMeshComponent and copy the contents of the version you downloaded into this folder, making sure not to nest subfolders, all the files/folders in the directory of the .uplugin file should be in your newly created /Plugins/RealtimeMeshComponent folder.
   
5. At this point you should be able to relaunch the project and the editor should detect the new plugin and compile it for you!

6. You should be able to start using the RMC!

### Using RMC From Code

1. To make the RMC available in your C++ project, you must first install it from either the marketplace or GitHub as described above.

2. Next, in your projects build file 'YourProjectName.Build.cs' you must add this line:

    ```c++
    PublicDependencyModuleNames.Add("RealtimeMeshComponent");
    ```

3. Now you should be able to #include the headers like any other plugin and use the RMC from C++!

### Using RMC from Blueprint

Using the RMC from Blueprint is simple, once the plugin is installed to the engine or project it's ready to go!

