# Getting started

### Preparation

*   Installation of Development Environment

    Follow the instructions in the [Development Environment Setup](https://github.com/DspreadOrg/qpos-linux-tools/blob/main/EnvironmentBuilding/DevelopEnvironmentGuide.md) to install the QPOS Plus Linux development environment.&#x20;

    ```shell
    git clone https://github.com/DspreadOrg/qpos-linux-tools.git
    ```
*   Clone qpos plus linux sdk and demo&#x20;

    In the command-line interface, navigate to your working directory and run the following command to clone the repository.

    ```sh
    git clone https://github.com/DspreadOrg/qpos-linux.git
    ```

### Project Structure

```cmake
qpos-linux-main/
├── app_demo/
│   └── linux_pos_app/
│       ├── lib/
│       └── src/
│           ├── common_api/
│           ├── custom_app/
│           │   ├── http/
│           │   ├── ota/
│           │   ├── setting/
│           │   ├── trans/
│           │       ├── app_trans.c
│           │       ├── app_trans.h
│           │       ├── ui_card.c
│           │       ├── ui_card.h
│           │       └── ui_emvSelectMultiApp.c
│           └── project/
│               ├── lvgl_porting.c
│               ├── proj_cfg.h
│               ├── task_handle.c
│               └── ui_main.c
└── qpos-linux-tools/
```



### Compiling and Running Your First Application

Below is a list of essential commands for managing apps with the PKG extension on VScode.

* **PKG-STOP APP**: Stops the app linked to the project on the connected device.
* **PKG-RUN APP**: Runs the app for the project on the connected device, provided it is installed.
* **PKG-UNINSTALL APP**: Uninstalls the project's app from the connected device.
* **PKG-INSTALL APP**: Installs the generated .apk file on the connected device.
* **PKG-CLEAN**: Deletes the output file from the previous compilation.
* **PKG-PACKAGE**: Packages compiled and resource files to create an installable APK.
* **PKG-RECOMPILE**: Recompiles the project, cleans previous outputs, and creates new compilation files.
* **PKG-COMPILE**: Compiles the project to produce the necessary compilation file.



### Debugging and Deployment

**Installing Application to Device**

Ensure the device is connected via USB and ADB is enabled:\
`adb install -r  trio-linux\app_demo\linux_pos_app\release\linux_pos_app.apk`\


**Running Application**\
`adb shell am start-n linux_pos_app/linux_pos_app`\


**Viewing Logs**\
`adb logcat -s linux_pos_ap`
