---
title: C++ quickstart for Xbox
author: v-thopra
description: This guide will help you make your first PlayFab API call in C++ on an Xbox.
ms.author: v-thopra
ms.date: 02/01/2019
ms.topic: article
ms.prod: playfab
keywords: playfab, c++, windows, xbox, cross-platform
ms.localizationpriority: medium
---

# C++ quickstart for Xbox

This guide will help you make your first **PlayFab API** call in C++ on an **Xbox**.

## Xbox Project Setup

Make sure to follow the **Xbox** setup guide for **Universal Windows Platform (UWP)** applications: [UWP on Xbox One](https://docs.microsoft.com/en-us/windows/uwp/xbox-apps). This will help with setting up your **Xbox**, the **XDK**, and **Visual Studio**, for building an **Xbox** application.

## Using the PlayFab XplatCPP SDK

Let's take a look at how to use the **PlayFab XplatCPP SDK**.

> [!NOTE]
> The main difference for **Xbox** is that it has a separate project file for the solution.

- Download the latest [XPlatCppSdk](https://github.com/PlayFab/XPlatCppSdk). Follow the instructions in the *Readme* to recursively clone the dependencies. Note that this SDK is supported across **Windows**, **Linux**, and **Xbox**. The **Xbox SDK** is currently supported only on **Visual Studio 2017**.

- Now, let's build the dependencies for **Xbox**.
  - Open the **Xbox One XDK Visual Studio 2017 Command Prompt** and run the **build-dependencies-{debug/profile/release}.bat** script. This script can be found under the **build \> Xbox** folder.
  - We will use debug for now, so make sure you run the debug script. The profile and release scripts can be run to build the respective configurations of the external dependencies. If the build fails, make sure that you completed a recursive clone of the repo to also clone the external dependencies.

### Setting up your visual studio project

We will use a Direct3D sample for the guide. This will help you understand how to setup the project. You can later choose to start with a different starting sample.

- Create a new **Direct3D** sample project by clicking on **File \> New \> Project**.
- Under **Installed**, click on **Visual C++ \> Xbox One \> XDK \> July 2018 QFE 2**.
  
  > [!NOTE]
  > Based on the **XDK** you have installed, the version might be different. If you do not see **Xbox One** under **Visual C++**, re-install the **XDK** and try again.

- Select the **Direct3D 11 Game**. Name the project as *PlayFabXboxGuide* and select a desirable location. Hit **OK** to create the project.

- Modify the **Xbox Live Manifest** by editing the **package.appxmanifest** file to explicitly call out port numbers. When using ports internally, we need to make the following change to allow the application to talk to the network:

   1. Refer to the manifest template code example in this documentation [Xbox One exclusive resource application network manifest templates](https://docs.microsoft.com/en-us/windows/uwp/xbox-live/multiplayer/xbox-integrated-multiplayer/xim-manifest#xbox-one-exclusive-resource-application-network-manifest-templates).

   2. Modify the section for the **XML** header `<mx:Extension Category="windows.xbox.networking">` so that it looks like this:

```xml
         ...
         <mx:Extension Category="windows.xbox.networking">
           <mx:XboxNetworkingManifest>
             <mx:SocketDescriptions>
               <mx:SocketDescription Name="PlayFabQosApiClient" BoundPort="0" SecureIpProtocol="Udp">
                 <mx:AllowedUsages>
                   <mx:SecureDeviceSocketUsage Type="Initiate"/>
                   <mx:SecureDeviceSocketUsage Type="SendInsecure"/>
                   <mx:SecureDeviceSocketUsage Type="ReceiveInsecure"/>
                 </mx:AllowedUsages>
               </mx:SocketDescription>
               <mx:SocketDescription Name="PlayFabQosApiServer" BoundPort="3075" SecureIpProtocol="Udp">
                 <mx:AllowedUsages>
                   <mx:SecureDeviceSocketUsage Type="Accept" />
                   <mx:SecureDeviceSocketUsage Type="SendInsecure"/>
                   <mx:SecureDeviceSocketUsage Type="ReceiveInsecure"/>
                 </mx:AllowedUsages>
               </mx:SocketDescription>
             </mx:SocketDescriptions>
             <mx:SecureDeviceAssociationTemplates>
               <mx:SecureDeviceAssociationTemplate Name="PlayFabQosApiTraffic" InitiatorSocketDescription="PlayFabQosApiClient" AcceptorSocketDescription="PlayFabQosApiServer" MultiplayerSessionRequirement="None">
                 <mx:AllowedUsages>
                   <mx:SecureDeviceAssociationUsage Type="InitiateFromMicrosoftConsole" />
                   <mx:SecureDeviceAssociationUsage Type="InitiateFromWindowsDesktop" />
                   <mx:SecureDeviceAssociationUsage Type="AcceptOnOtherDevice" />
                 </mx:AllowedUsages>
               </mx:SecureDeviceAssociationTemplate>
             </mx:SecureDeviceAssociationTemplates>
           </mx:XboxNetworkingManifest>
         </mx:Extension>
```

- At this point, you should be able to deploy your game.
  - Hit **F5** to deploy and run the game. 
  - Make sure you are connected to your **Xbox**. 
  - On the **Xbox One XDK Visual Studio 2017 Command Prompt**, run the `xbconnect \<xbox ip\>` command to connect to your **Xbox**.
  - You should see the game running on your **Xbox**.

### Adding the XPlatCppSdk to your project

- In **Visual Studio**, right click on the **Solution**. Select **Add \> Existing Project**. 
- Navigate to and select **XPlatXbox.vcxproj** under the **build \> Xbox** folder of the **XPlatCppSdk** folder.
- Hit **Open**. You should now see the **XPlatXbox** project as part of your solution.
- Right click on **References** under the *PlayFabXboxGuide* project. 
- Click on **Add Reference**. 
- Under **Project \> Solution**, select the **XPlatXbox** project and hit **OK**. 
- Now your project is linked to the **XPlatXbox SDK** and you should be able to build to make **PlayFab API** calls.

This is a good point to verify that the project builds. Try to build the Solution and make sure you do not have any build errors.

You are all set to write some code and make your first API call.

### Making your first API call

- Add the **PlayFab** header file paths to the **Additional Include Directories**. 
  - Right click on the *PlayFabXboxGuide* project and click **properties**. 
  - Under **Configuration Properties \> C/C++ \> General \> Additional Include Directories**, add the path to the **code**, **code\include** and **external\include** folders. 
  - Once added, the **Additional Include Directories** value should look like this:  
    ` \<path to the sdk root\>\code; \<path to the sdk root\>\code\include; \<path to the sdk root\>\external\include;%(AdditionalIncludeDirectories)`

- Add a new header file called **PlayFabApiCall.h**, and add the following code to the file:

```cpp
#pragma once
void MakePlayFabCall();
```

- Add a new source file called **PlayFabApiCall.cpp** with the following code:

```cpp
#include "pch.h"

#include "PlayFabApiCall.h"
#include <playfab/PlayFabClientDataModels.h>
#include <playfab/PlayFabClientApi.h>
#include <playfab/PlayFabSettings.h>
#include <windows.h>

using namespace PlayFab;
using namespace ClientModels;

bool finished = false;

void print_log(const char* format, ...)
{
    static char s_printf_buf[1024];
    va_list args;
    va_start(args, format);
    vsnprintf(s_printf_buf, sizeof(s_printf_buf), format, args);
    va_end(args);
    OutputDebugStringA(s_printf_buf);
}

void OnLoginSuccess(const LoginResult& result, void* customData)
{
    print_log("Congratulations, you made your first successful API call!\n");
    finished = true;
}

void OnLoginFail(const PlayFabError& error, void* customData)
{
    print_log("Something went wrong with your first API call.\n");
    print_log("Here's some debug information:\n");
    print_log(error.GenerateErrorReport().c_str());
    print_log("\n");
    finished = true;
}

void MakePlayFabCall()
{
    print_log("\n\n=====================Start making your first PlayFab API call========================\n\n");

    PlayFabSettings::titleId = "144";

    LoginWithCustomIDRequest request;
    request.CreateAccount = true;
    request.CustomId = "GettingStartedGuide";

    PlayFabClientAPI::LoginWithCustomID(request, OnLoginSuccess, OnLoginFail);

    while (PlayFabClientAPI::Update() != 0)
        Sleep(1);

    print_log("\n============================End your first PlayFab API call==========================\n\n");
}
```

Now you have the code to make your first API call.

- Calling the **MakePlayFabCall()** function:
  - In **Main.cpp**, add the following include statement:

    ```cpp
    #include "PlayFabApiCall.h"
    ```

- In the **virtualvoid Initialize(CoreApplicationView^ applicationView)** function in **Main.cpp**, add `MakePlayFabCall();` as the last call.
- Hit **F5** to deploy and run the application. 
- In the Output log window, you should be able to see the result of your first **PlayFab API** call from the **Xbox**.

Great! Now that you know how to setup the **XplatCPP SDK** for **Xbox**, you can create your game and start making **PlayFab API** calls.