![Picture](http://www.polljoy.com/assets/images/logo/polljoy-logo-github.png)

In-app polls made easy. Just 2 API calls. Find out why users leave and get instant user feedback. Stats tell you what, polljoy tells you why.



#Intro
Hi friend! Let's add polljoy to your awesome Unity game. It's simple, you'll be up in minutes.

Questions? - email help@polljoy.com and one of our devs will assist!

#Web console
Polls are created and managed through our web interface - https://admin.polljoy.com

#Steps
1. Import NGUI package (if you are not using NGUI in your project. polljoy SDK requires NGUI 3 to work properly)
2. Download the archive to your local drive and unzip it
3. Locate "polljoy_SDK.unitypackage" in the "Dist" folder, and import it to your Unity project. To start using polljoy, please continue reading "Start session" and "Get poll" below.

(optional)
4. Import the "polljoy_testapp.unitypackage" in the "TestApp" folder. The test app allows you to try out a few polls. Open the "TestAppScene" and you are good to go!

###Start session 
polljoy works in the background to avoid interrupting your app’s main thread.

Each app starts a session and gets the **Session ID** for all communications to the API. To have best performance and integration, we recommend registering the session at startup. You’ll need your **App ID** (grab it in the web [admin panel](https://admin.polljoy.com/applications/app))

 To start a session:
 1. On Application Start, call:

 ``` c#

 // ...
 public class ApplicationStart : MonoBehaviour
     // ...
     Polljoy.startSession("YOUR_APP_ID");
     // ...

 ```

The SDK will automatically handle all session control and required information to get the correct poll based on your poll setup in admin panel and save your poll result for analysis. These includes the session ID, session count, time (days) since first call polljoy SDK, device ID, platform, OS version … etc.

Each time you call `startSession`, the SDK will increase the session count by 1. So, you should only call it once for each launch to get the session count correct.

Once the session is started, SDK will cache all app settings including the default image, border image and button image (if any) that you have setup in the [admin panel](https://admin.polljoy.com). After caching, there will be no operation until you request polls from polljoy service.

By default the SDK will assign an unique device ID to each device, if you would however like to override this and assign the device ID yourself, call this method instead

 ``` c#

 // ...
 public class ApplicationStart : MonoBehaviour
     // ...
     Polljoy.startSession("YOUR_APP_ID", "DEVICE_ID");
     // ...

 ```

###Get poll (simple)
After you start the session, you can get polls any time and place you want!

In your program logic, at the point you want to get the poll, call:

  ``` c#
      Polljoy.getPoll();
  // use this if you don't need to handle callbacks from polljoy
  // this will auto show the polls when all polls are ready
  ```
  OR
  ``` c#
    Polljoy.getPoll(delegate);
  // if you want to handle callbacks from polljoy
  ```
`Note: The above two methods are simplified version if you do not have any selection criteria.  If you want to select specific polls to the users based on session, timeSinceInstall, platform, or more , please use the full method call below.

###Get poll (full)
 ``` c#
 // ...
   Polljoy.getPoll(appVersion,
                   level,
                   session,
                   timeSinceInstall,
                   userType,
                   tags,
                   delegate);
 // ...
 ```

In summary:

`appVersion` (optional) Set to null if you prefer not to send it.  Or you can choose to pass it. eg 1.0.35

`level` (optional) Set as 0 if you prefer not to send it. If your app has levels you can pass them here. eg 34 

`session` (optional) Set it as "Polljoy.Session" and the SDK will count it for you.  Or you can manually send it. eg 3 

`timeSinceInstall` (optional) Set it as "PolljoyCore.getTimeSinceInstall()" and the SDK will count it for you.  Or you can manually set it by sending a value for how long the app has been installed (by default, counted in days). eg 5

`userType` Pass back either the user type **Pay** or **Non-Pay**. This is the class `PJUserType` as defined in `PJUserType.cs`

`tags` (optional) Set to null if you aren't using them.  If your game uses tags to select polls, pass them in string format with as many as you want to send - `TAG,TAG, ... ,TAG`.  TAG is either in the format TAGNAME or TAGNAME:VALUE.  They should match what you defined in the web console. An example of sending back player gender, current energy and where the poll is being called from could be: `MALE,ENERGY#18,PVPMENU`

`delegate` (optional) Set to null if not needed. Delegate is the instance to handle all callbacks from polljoy SDK. If used, the delegate should implement `PolljoyDelegate` as defined in `Polljoy.cs`

### Callbacks

To use the delegate and callbacks, please extend the method "PolljoyDelegate" and implement the callback methods.
polljoy will inform delegate at different stages when the polls are downloaded, ready to show, user responded etc. The game can optionally implement the delegate methods to control the app logic. The delegate methods are (defined in PolljoyDelegate.cs):

 ``` c#
 void PJPollNotAvailable(PJResponseStatus status);
 ```

When there is no poll matching your selection criteria or no more polls to show in the current session. The error code is passed back as a enum.

 ``` c#
 void PJPollIsReady(List<PJPoll> polls);
 ```

When poll/s is/are ready to show (including all associated images). Friendly tip - If you are displaying the poll in the middle of an active game or app session that needs real time control, consider to pause your game before presenting the poll UI as needed. 

The polls array returned are all the matched polls for the request. Please refer `PJPoll.cs` for the data structure.
When you’re ready to present the poll, call:

 ``` c#
 Polljoy.showPoll();
 ```

This will present the polljoy UI according to your app style and poll settings. Then the SDK will handle all the remaining tasks for you. These include handling the user’s response, informing delegate for any virtual amount user received, uploading the result to the console … etc.

We recommend you implement this delegate method so you know when polls are ready and call polljoy SDK to show the poll or do whatever control you need.

 ``` c#
 void PJPollWillShow(PJPoll poll);
 ```

The polljoy poll UI is ready and will show. You can do whatever UI control as needed. Or simply ignore this implementation.

 ``` c#
 void PJPollDidShow(PJPoll poll);
 ```

The polljoy poll UI is ready and has shown. You can do whatever UI control as needed. Or simply ignore this implementation.

 ``` c#
 void PJPollWillDismiss(PJPoll poll);
 ```

The polljoy poll UI is finished and will dismiss. You can do whatever UI control as needed. Or simply ignore this implementation. You can prepare your own UI before resuming your game before the polljoy poll UI is dismissed.

 ``` c#
 vvoid PJPollDidDismiss(PJPoll poll);
 ```

The polljoy poll UI is finished and has dismissed. You can do whatever UI control as needed. Or simply ignore this implementation. You can prepare your own UI to resume your game before the polljoy UI is dismissed. This is the last callback from polljoy and all polls are completed. You should resume your game if you have paused.

 ``` c#
 void PJPollDidResponded(PJPoll poll);
 ```

User has responded to the poll. The poll will contain all the poll data including user’s responses. You can ignore this (the results are displayed in the web admin console and able to be exported) or use it as you wish.
If you issue a virtual currency amount to user, you MUST implement this method to handle the virtual amount issued. This is the only callback from SDK that informs the game the virtual amount that the user collected.

 ``` c#
 void PJPollDidSkipped(PJPoll poll);
 ```

 If the poll is not mandatory, the user can choose to skip the poll. You can handle this case or simply ignore it safely.

-
That's it!  Email us at help@polljoy.com if you have questions or suggestions!

ps - love penguins? [good!](https://polljoy.com/world.html)
