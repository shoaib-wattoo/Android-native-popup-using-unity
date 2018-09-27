### What is Pop-Ups?
Popup is one small screen or some alert message which asks user to take some action.
#### Here we will be creating three types of pop-ups:
- **Message Popup**     Single Action
- **Confirmation Popup**    Two Actions
- **Rate-US Popup**     Three Actions

**Now lets create some simple popup.**

## Step 1 : Setup scene in Unity
Create new Unity project and save scene to your assets folder.

**Create three buttons for three popups:**

![](http://www.theappguruz.com/app/uploads/2016/05/unity-setup.png)

## Step 2 Create script and assign all button reference
Create script and name it as you wish. I have named it **PopupView.cs**. Now let's write some code for adding event listener on button click.

Create three methods for each button and assign reference on button click event. Create enum for message state returned from Android native dialog actions.

```csharp
public enum MessageState
{
    OK,
    YES,
    NO,
    RATED,
    REMIND,
    DECLINED,
    CLOSED
}
    #region PUBLIC_VARIABLES
    // App ID to rate it
    [Tooltip("market://details?id=BUNDLE-ID")]
    public string gameLink = "market://details?id=com.tag.tabletennis3D";
    #endregion 
#region BUTTON_EVENT_LISTENER
    // Dialog Button click event
    public void OnDialogPopUp()
    {
        NativeDialog dialog = new NativeDialog("TheAppGuruz", "Do you wants to know about TheAppGuruz");
        dialog.SetUrlString("http://theappguruz.com/");
        dialog.init();
    }
    // Rate Button click event
    public void OnRatePopUp()
    {
        NativeRateUS ratePopUp = new NativeRateUS("Like this game?", "Please rate to support future updates!");
        ratePopUp.SetAppLink(gameLink);
        ratePopUp.InitRateUS();    
    }
    // Message Button click event
    public void OnMessagePopUp()
    {
        NativeMessage msg = new NativeMessage("TheAppGuruz", "Welcome To TheAppGuruz");
    }
    #endregion 
```

**Now let’s register delegate event listener for native popup actions.**

```csharp
#region UNITY_DEFAULT_CALLBACKS
    void OnEnable()
    {
        // Register all Delegate event listener
        AndroidRateUsPopUp.onRateUSPopupComplete += OnRateUSPopupComplete;
        AndroidDialog.onDialogPopupComplete += OnDialogPopupComplete;
        AndroidMessage.onMessagePopupComplete += OnMessagePopupComplete;
    }
    void OnDisable()
    {
        // Deregister all Delegate event listener
        AndroidRateUsPopUp.onRateUSPopupComplete -= OnRateUSPopupComplete;
        AndroidDialog.onDialogPopupComplete -= OnDialogPopupComplete;
        AndroidMessage.onMessagePopupComplete -= OnMessagePopupComplete;
    }
    #endregion
    #region DELEGATE_EVENT_LISTENER
    // Raise when click on any button of rate popup
    void OnRateUSPopupComplete(MessageState state)
    {
        switch (state)
        {
            case MessageState.RATED:
                Debug.Log("Rate Button pressed");
                break;
            case MessageState.REMIND:
                Debug.Log("Remind Button pressed");
                break;
            case MessageState.DECLINED:
                Debug.Log("Declined Button pressed");
                break;
        }
    }
    // Raise when click on any button of Dialog popup
    void OnDialogPopupComplete(MessageState state)
    {
        switch (state)
        {
            case MessageState.YES:
                Debug.Log("Yes button pressed");
                break;
            case MessageState.NO:
                Debug.Log("No button pressed");
                break;
        }
    }
    // Raise when click on ok button of message popup
    void OnMessagePopupComplete(MessageState state)
    {
        Debug.Log("Ok button Clicked");
    }
    #endregion 
```

## Step 3 : Create script to interact with Android file (.jar file)
Now Create a script to directly make interaction with android code named **AndroidNative.cs**

```csharp
using UnityEngine;
using System.Collections;
public class AndroidNative
{
    public static void CallStatic(string methodName, params object[] args)
    {
        #if UNITY_ANDROID && !UNITY_EDITOR
        try
        {
            string CLASS_NAME = "com.tag.nativepopup.PopupManager";
            AndroidJavaObject bridge = new AndroidJavaObject(CLASS_NAME);
            AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer"); 
            AndroidJavaObject act = jc.GetStatic<AndroidJavaObject>("currentActivity"); 
            
            act.Call("runOnUiThread", new AndroidJavaRunnable(() =>
            {
                bridge.CallStatic(methodName, args);
            }));
        } catch (System.Exception ex)
        {
            Debug.LogWarning(ex.Message);
        }
        #endif
    }
    
    public static void showRateUsPopUP(string title, string message, string rate, string remind, string declined)
    {
        CallStatic("ShowRatePopup", title, message, rate, remind, declined);
    }
    public static void showDialog(string title, string message, string yes, string no)
    {
        CallStatic("ShowDialogPopup", title, message, yes, no);
    }
    public static void showMessage(string title, string message, string ok)
    {
        CallStatic("ShowMessagePopup", title, message, ok);
    }
    public static void RedirectToAppStoreRatingPage(string appLink)
    {
        CallStatic("OpenAppRatingPage", appLink);
    }
    public static void RedirectToWebPage(string urlString)
    {
        CallStatic("OpenWebPage", urlString);
    }
} 
```

## Step 4 : Create scripts to create different popups
#### MESSAGE POPUP

**A)** Create **NativeMessage.cs** for Basic setup of simple message popup:

```csharp
using UnityEngine;
using System.Collections;
public class NativeMessage
{
    #region PUBLIC_FUNCTIONS
    public NativeMessage(string title, string message)
    {
        init(title, message, "Ok");
    }
    public NativeMessage(string title, string message, string ok)
    {
        init(title, message, ok);
    }
    private void init(string title, string message, string ok)
    {
        #if UNITY_ANDROID
        AndroidMessage.Create(title, message, ok);
        #endif
    }
    #endregion
} 
```
**B)** Create **AndroidMessage.cs** for simple message popup:

```csharp
public class AndroidMessage : MonoBehaviour
{
    #region DELEGATE
    public delegate void OnMessagePopupComplete(MessageState state);
    public static event OnMessagePopupComplete onMessagePopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnMessagePopupComplete(MessageState state)
    {
        if (onMessagePopupComplete != null)
            onMessagePopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string ok;
    #endregion
    #region PUBLIC_FUNCTIONS
    public static AndroidMessage Create(string title, string message)
    {
        return Create(title, message, "Ok");
    }
    public static AndroidMessage Create(string title, string message, string ok)
    {
        AndroidMessage dialog;
        dialog = new GameObject("AndroidMessagePopup").AddComponent<AndroidMessage>();
        dialog.title = title;
        dialog.message = message;
        dialog.ok = ok;
        
        dialog.init();
        return dialog;
    }
    public void init()
    {
        AndroidNative.showMessage(title, message, ok);
    }
    #endregion
    #region ANDROID_EVENT_LISTENER
    public void OnMessagePopUpCallBack(string buttonIndex)
    {
        RaiseOnMessagePopupComplete(MessageState.OK);
        Destroy(gameObject);
    }
    #endregion
}
```

#### CONFIRMATION POPUP

**A)** Create **NativeDialog.cs** for basic setup of dialog message popup:

```csharp
public class NativeDialog
{
    #region PUBLIC_VARIABLES
    string title;
    string message;
    string yesButton;
    string noButton;
    public string urlString;
    #endregion
    #region PUBLIC_FUNCTIONS
    public NativeDialog(string title, string message)
    {
        this.title = title;
        this.message = message;
        this.yesButton = "Yes";
        this.noButton = "No";
    }
    public NativeDialog(string title, string message, string yesButtonText, string noButtonText)
    {
        this.title = title;
        this.message = message;
        this.yesButton = yesButtonText;
        this.noButton = noButtonText;
    }
    public void SetUrlString(string urlString)
    {
        this.urlString = urlString;
    }
    public void init()
    {
        #if UNITY_ANDROID
        AndroidDialog dialog = AndroidDialog.Create(title, message, yesButton, noButton);
        dialog.urlString = urlString;
        #endif
    }
    #endregion
} 
```

**B)** Create **AndroidDialog.cs** for dialog message popup:

```csharp
public class AndroidDialog : MonoBehaviour
{
    #region DELEGATE
    public delegate void OnDialogPopupComplete(MessageState state);
    public static event OnDialogPopupComplete onDialogPopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnOnDialogPopupComplete(MessageState state)
    {
        if (onDialogPopupComplete != null)
            onDialogPopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string yes;
    public string no;
    public string urlString;
    #endregion
    #region PUBLIC_FUNCTIONS
    // Constructor
    public static AndroidDialog Create(string title, string message)
    {
        return Create(title, message, "Yes", "No");
    }
    public static AndroidDialog Create(string title, string message, string yes, string no)
    {
        AndroidDialog dialog;
        dialog = new GameObject("AndroidDialogPopup").AddComponent<AndroidDialog>();
        dialog.title = title;
        dialog.message = message;
        dialog.yes = yes;
        dialog.no = no;
        dialog.init();
        return dialog;
    }
    public void init()
    {
        AndroidNative.showDialog(title, message, yes, no);
    }
    #endregion
    #region ANDROID_EVENT_LISTENER
    public void OnDialogPopUpCallBack(string buttonIndex)
    {
        int index = System.Convert.ToInt16(buttonIndex);
        
        switch (index)
        {
            case 0: 
                AndroidNative.RedirectToWebPage(urlString);
                RaiseOnOnDialogPopupComplete(MessageState.YES);
                break;
            case 1: 
                RaiseOnOnDialogPopupComplete(MessageState.NO);
                break;
        }
        Destroy(gameObject);
    }
    #endregion
} 
```

#### RATE-US POPUP

**A)** Create **NativeRateUS.cs** for Basic setup of rate-us popup:

```csharp
public class NativeRateUS
{
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string yes;
    public string later;
    public string no;
    public string appLink;
    #endregion
    #region PUBLIC_FUNCTIONS
    // Constructor
    public NativeRateUS(string title, string message)
    {
        this.title = title;
        this.message = message;
        this.yes = "Rate app";
        this.later = "Later";
        this.no = "No, thanks";
    }
    // Constructor
    public NativeRateUS(string title, string message, string yes, string later, string no)
    {
        this.title = title;
        this.message = message;
        this.yes = yes;
        this.later = later;
        this.no = no;
    }
    // Set AppID to rate app
    public void SetAppLink(string _appLink)
    {
        appLink = _appLink;
    }
    
    // Initialize rate popup
    public void InitRateUS()
    {
        #if UNITY_ANDROID
        AndroidRateUsPopUp rate = AndroidRateUsPopUp.Create(title, message, yes, later, no);
        rate.appLink = appLink;
        #endif
    }
    #endregion
} 
```

**B)** Create **AndroidRateUsPopUp.cs** for rate-us popup:

```csharp
public class AndroidRateUsPopUp : MonoBehaviour
{
    #region DELEGATE
    public delegate void OnRateUSPopupComplete(MessageState state);
    public static event OnRateUSPopupComplete onRateUSPopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnOnRateUSPopupComplete(MessageState state)
    {
        if (onRateUSPopupComplete != null)
            onRateUSPopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string rate;
    public string remind;
    public string declined;
    public string appLink;
    #endregion
    #region PUBLIC_FUNCTIONS
    public static AndroidRateUsPopUp Create()
    {
        return Create("Like the Game?", "Rate US");
    }
    public static AndroidRateUsPopUp Create(string title, string message)
    {
        return Create(title, message, "Rate Now", "Ask me later", "No, thanks");
    }
    public static AndroidRateUsPopUp Create(string title, string message, string rate, string remind, string declined)
    {
        AndroidRateUsPopUp popup = new GameObject("AndroidRateUsPopUp").AddComponent<AndroidRateUsPopUp>();
        popup.title = title;
        popup.message = message;
        popup.rate = rate;
        popup.remind = remind;
        popup.declined = declined;
        
        popup.init();
        return popup;
    }
    public void init()
    {
        AndroidNative.showRateUsPopUP(title, message, rate, remind, declined);
    }
    #endregion
    #region ANDROID_EVENT_LISTENER
    public void OnRatePopUpCallBack(string buttonIndex)
    {
        int index = System.Convert.ToInt16(buttonIndex);
        switch (index)
        {
            case 0: 
                AndroidNative.RedirectToAppStoreRatingPage(appLink);
                RaiseOnOnRateUSPopupComplete(MessageState.RATED);
                break;
            case 1:
                RaiseOnOnRateUSPopupComplete(MessageState.REMIND);
                break;
            case 2:
                RaiseOnOnRateUSPopupComplete(MessageState.DECLINED);
                break;
        }
        Destroy(gameObject);
    }
    #endregion
} 
```

### Note
In this classes (B section of each popup) we have created gameobject and we are using this gameobject-name to get event callback. We are using this names in **UnitySendMessage()** to get callback from android. So do not change name of Gameobject.

## Step 5 : Setup Android file

Awesome! you are done with basic code!

Now lets write code to create popups in native Android using Android Studio or Eclipse editor.

To do that, Create new Android project. If you do not know about android studio and how to create new project then please refer Create New Project in Android Studio.

Do not worry about code for now. Just copy it and paste it in your file.

### Note
If you face any issues in creating project or file then you can download source code from the bottom of this blog post. And once you are done with downloading the project, you can copy **Plugins** folder into your Unity project.

Coming back to Android Studio, create new file named PopupManager.

![](http://www.theappguruz.com/app/uploads/2016/05/create-new-class.png)

**Now paste following code in this newly created file:**

```csharp
package com.tag.nativepopup;
import android.annotation.SuppressLint;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.DialogInterface.OnKeyListener;
import android.net.Uri;
import android.os.Build;
import android.util.Log;
import android.view.ContextThemeWrapper;
import android.view.KeyEvent;
import com.unity3d.player.UnityPlayer;
public class PopupManager
{
    public static void ShowMessagePopup(String title, String message, String okButtonText) {
         AlertDialog.Builder messagePopup = new AlertDialog.Builder(new ContextThemeWrapper(UnityPlayer.currentActivity, GetTheme()));
         messagePopup.setTitle(title);
         messagePopup.setMessage(message);
         messagePopup.setPositiveButton(okButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidMessagePopup", "OnMessagePopUpCallBack", "0");
            }
        });
         messagePopup.setOnKeyListener(KeyListener);
         messagePopup.setCancelable(false);
         messagePopup.show();
    }
    
    public static void ShowDialogPopup(String title, String message, String yesButtonText, String noButtonText) {
         AlertDialog.Builder dialogPopupBuilder = new AlertDialog.Builder(new ContextThemeWrapper(UnityPlayer.currentActivity, GetTheme()));
         dialogPopupBuilder.setTitle(title);
         dialogPopupBuilder.setMessage(message);
         dialogPopupBuilder.setPositiveButton(yesButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidDialogPopup", "OnDialogPopUpCallBack", "0");
            }
        });
         dialogPopupBuilder.setNegativeButton(noButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidDialogPopup", "OnDialogPopUpCallBack", "1");
            }
        });
         dialogPopupBuilder.setOnKeyListener(KeyListener);
         dialogPopupBuilder.setCancelable(false);
         dialogPopupBuilder.show();
    }
    
    public static void ShowRatePopup(String title, String message, String yesButtonText, String laterButtonText, String noButtonText) {        
         AlertDialog.Builder ratePopupBuilder = new AlertDialog.Builder(new ContextThemeWrapper(UnityPlayer.currentActivity, GetTheme()));
         ratePopupBuilder.setTitle(title);
         ratePopupBuilder.setMessage(message);
         ratePopupBuilder.setPositiveButton(yesButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidRateUsPopUp", "OnRatePopUpCallBack", "0");
            }
         });
         ratePopupBuilder.setNegativeButton(noButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidRateUsPopUp", "OnRatePopUpCallBack", "2");
            }
        });
        
         ratePopupBuilder.setNeutralButton(laterButtonText, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                UnityPlayer.UnitySendMessage("AndroidRateUsPopUp", "OnRatePopUpCallBack", "1");
            }
        });
         ratePopupBuilder.setOnKeyListener(KeyListener);
         ratePopupBuilder.setCancelable(false);
         ratePopupBuilder.show();
    }
    
    @SuppressLint("InlinedApi")
    private static int GetTheme(){
        int theme = 0;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            theme = android.R.style.Theme_Material_Light_Dialog;
        } else {
            theme = android.R.style.Theme_Holo_Dialog;
        }
        return theme;
    }
    
    public static void OpenAppRatingPage(String url) {
         Uri uri = Uri.parse(url);    
         Intent intent = new Intent(Intent.ACTION_VIEW, uri);
         UnityPlayer.currentActivity.startActivity(intent);
    }
    
    public static void OpenWebPage(String webUrl){
        Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse(webUrl));
        UnityPlayer.currentActivity.startActivity(browserIntent);
    }
    
    private static DialogInterface.OnKeyListener KeyListener = new OnKeyListener() {
     @Override
        public boolean onKey(DialogInterface dialog, int keyCode, KeyEvent event) {
            if (keyCode == KeyEvent.KEYCODE_BACK) {
                Log.d("AndroidNative", "AndroidPopUp");
                UnityPlayer.UnitySendMessage("AndroidMessagePopup", "OnMessagePopUpCallBack", "0");
                UnityPlayer.UnitySendMessage("AndroidDialogPopup", "OnDialogPopUpCallBack", "1");
                UnityPlayer.UnitySendMessage("AndroidRateUsPopUp", "OnRatePopUpCallBack", "2");
                dialog.dismiss();
            }
            return false;
        }
    };
}
```

### Note

In this class we are using **UnityPlayer.UnitySendMessage()** to send message to Unity and we are using gameobject-name as parameter. This must match with gameobject which is created in particular popup class(C# file).

Now export that file as .jar file from Eclipse/ AndroidStudio and put this .jar file to **Plugins >> Android folder**.

![](http://www.theappguruz.com/app/uploads/2016/05/export-jar-file.png)

*If you face any issues in creating Android project or .jar file then you can download source code and just import(Drag and drop) Plugins folder to your unity project.*
