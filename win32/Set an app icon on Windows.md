# Set an app icon on Windows

When we initialize the window class we can set an icon:
```c++
WNDCLASSA windowClass = {};
windowClass.hIcon = LoadIconA(windowClass.hInstance, MAKEINTRESOURCEA(IDI_MYAPP_ICON));
```
We have an `resource.h` file where we define the `IDI_MYAPP_ICON`:

```c++
#define IDI_MYAPP_ICON 101
```
We have a `resource.rc` file where we write:

```c++
include “resource.h”
101 ICON "path to icon relative to this file"
```
In our `build.bat` file we need to compile the resource (after we `pushd` in build folder)
```bash
pushd .\build
rc /fo icon_name.res /nologo ..\code\resource.rc
```
Now we have an `icon_name.res` file in the build folder. We need to compile it with our `win32_platform.cpp` file:
```bash
cl %COMPILER_FLAGS% ..\code\win32_platform.cpp icon_name.res -Fwin32_platform.map -Fe%EXE_NAME% -link %LINKER_FLAGS%
```

