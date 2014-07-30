![Picture](http://www.polljoy.com/assets/images/logo/polljoy-logo-github.png)

In-app polls made easy. Just 2 API calls.


#Intro
Hi friend! Let's add polljoy to your awesome Unity game. It's pretty simple, you should be up in minutes.

Questions? - email help@polljoy.com and one of our devs will assist!

#Web console
Polls are created and managed through our web interface - https://admin.polljoy.com

#Steps
1. Download `Dist` folder to your local drive
2. Import polljoy_SDK.unitypackage
3. Import NGUI package (if you haven't already. polljoy SDK requires NGUI 2)

###Implement polljoy in the project

polljoy works in the background to avoid interruption to your app’s main thread.

Each app starts a session and gets the **Session ID** for all communications to the API. To have best performance and integration, we recommend registering the session at startup. You’ll need your **App ID** (found in the web [admin panel](https://admin.polljoy.com/applications/app)

 To start a session:
 1. On Application Start, call:

 ``` c#

 // ...
 public class ApplicationStart : MonoBehaviour
     // ...
     Polljoy.startSession("YOUR_APP_ID");
     // ...

 ```

 polljoy SDK will automatically handle all session control and all required information to get the correct poll based on your poll setup in admin panel and save your poll result for analysis. These includes the session ID, session count, time (days) since first call polljoy SDK, device ID, platform, OS version … etc.

 Each time you call `startSession`, SDK will increase the session count by 1. So, you should only call it once for each app launch to get the session count correct.

 Once the session is started, SDK will cache all app settings including the default image, border image and button image (if any) that you have setup in the [admin panel](https://admin.polljoy.com). After caching, there will be no operation until you request polls from polljoy service.

### Get polls

After you started the session, you can get polls at any time and place you want!

In your program logic, at the point you want to get poll, call:

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

`appVersion`: your app's version to be used as a poll selection criteria. This should match with your poll setting. Or set it as nil if you are not using.

`level`: if your app is a game app, this is your game level. This should match with your poll setting. Or set it as 0 if you are not using it.

`session`: This is used to keep track of how many times the user has started a polljoy session. If your app keeps track of how many times the app has been launched, you can pass that variable here. Otherwise leave it to 0 and the polljoy SDK will handle it.

`timeSinceInstall`: If your app tracks how long the app has been installed, pass the variable here. Leave it to 0 will let polljoy does the job. (We count in days).

`userType`: your app user type either **Pay** or **Non-Pay**. This is the `ENUM PJUserType` as defined in `Polljoy.java`

`tags`: If you app uses tags to select polls, you pass the tags here. Please remember this has to match your settings in admin panel. Tags are passed in the string format `TAGNAME,TAGNAME#RANGE` For example, if you want to ask the specific question to male users that are 18, you can put a tag like `MALE,AGE#18`

`delegate`: the instance to handle all callbacks from polljoy SDK. The delegate should implement `PolljoyDelegate` as defined in `Polljoy.java`

`NOTE: if you don’t use any poll selection criteria, you can simply call the following method and let the SDK handle everything.

  ``` c#
  // if you DON’T need to handle callbacks from Polljoy
  // this will auto show the polls when all polls are ready
      Polljoy.getPoll();
  // ...
  ```
  OR
  ``` c#
  // if you need to handle callbacks from Polljoy
    Polljoy.getPoll(delegate);
  // ..
  ```

### Handle callbacks from SDK

polljoy will inform delegate at different stages when polls are downloaded, ready to show, user responded etc. App can optionally implement the delegate methods to control the app logic. The delegate methods are (defined in PolljoyDelegate.cs):


 ``` c#
 void PJPollNotAvailable(string status);
 ```

When there is no poll match with your selection criteria or no more polls to show in the current session.

 ``` c#
 void PJPollIsReady(ArrayList<PJPoll> polls);
 ```

After you request for poll and poll/s is/are ready to show (including all defined images are downloaded). Friendly tip - If you are displaying the poll in the middle of an active game or app session that needs real time control, consider to pause your app before presenting the poll UI as needed.

polls array returned are all the matched polls for the request. Please refer `PJPoll.h` for the data structure.
When you’re ready to present the poll, call:

 ``` c#
 Polljoy.showPoll();
 ```

This will present the polljoy UI according to your app color and poll settings. Then polljoy SDK will handle all the remaining tasks for you. These include handling the user’s response, informing delegate for any virtual amount user received, upload result to polljoy service … etc.

We highly recommend you implement this delegate method so that you know polls are ready and call polljoy SDK to show the poll or do whatever control you need.

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

The polljoy poll UI is finished and will dismiss. You can do whatever UI control as needed. Or simply ignore this implementation. You can prepare your own UI before resuming your app before the polljoy poll UI is dismissed.

 ``` c#
 vvoid PJPollDidDismiss(PJPoll poll);
 ```

The polljoy poll UI is finished and has dismissed. You can do whatever UI control as needed. Or simply ignore this implementation. You can prepare your own UI to resume your app before the polljoy UI is dismissed. This is the last callback from polljoy and all polls are completed. You should resume your app if you have paused.

 ``` c#
 void PJPollDidResponded(PJPoll poll);
 ```

User has responded to the poll. The poll will contain all the poll data including user’s responses. You can ignore this (the results are displayed in the polljoy.com admin console and able to be exported) or use it as you wish.
If you issue a virtual currency amount to user, you MUST implement this method to handle the virtual amount issued (especially if your app is game). This is the only callback from SDK that informs the app the virtual amount that the user collected.

 ``` c#
 void PJPollDidSkipped(PJPoll poll);
 ```

 If the poll is not mandatory, user can choose to skip the poll. You can handle this case or simply ignore it safely.

-
#### Got questions? Email us at help@polljoy.com
