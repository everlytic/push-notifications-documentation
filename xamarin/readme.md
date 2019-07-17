# Everlytic Push Xamarin SDK

- [SDK Reference](./quick_reference.html)
- [Everlytic Push List Setup](../list_setup.html)
- [Change Log](https://everlytic.github.io/push-notifications-sdk-xamarin/changelog.html)
- [Test Scripts](./test_script.html)
- [Sample App](https://github.com/everlytic/push-notifications-xamarin-sample-app)

You can find the source code [here](https://github.com/everlytic/push-notifications-sdk-xamarin).

## Prerequisites
- A Firebase Project set up for your app. See the official Firebase docs [here](https://firebase.google.com/docs).
- **Note:** Make sure that the FCM API is enabled on your Firebase Account.

## Notes

- iOS is currently not implemented. Called methods will throw `NotImplementedException` when run on an iOS device.
- A Test mode is provided during the alpha phase. This mocks HTTP requests to the Everlytic API for basic testing.

## Getting Started

- Add the following to your `AndroidManifest.xml` file, replacing `{config}` field with the SDK Configuration string.
    ```xml
    <application>
      <meta-data android:name="com.everlytic.api.SDK_CONFIGURATION" android:value="{config}"></meta-data>
    </application>
    ```

    - Alternatively, you can pass the configuration string as a parameter to the SDK initialization method (see below).

    _Your SDK Configuration string can obtained from your Push Projects page in Everlytic_
- [Add your Firebase `google-services.json` file in your project](https://firebase.google.com/docs/android/setup?authuser=0#add-config-file). You may need to add the `Xamarin.GooglePlayServices.Base` NuGet package to fully configure your `google-services.json` file.

## Initialize the SDK in your top level Application class

### Xamarin.Forms

```c#

using Com.EverlyticPush;

public class App : Application
{
    public App()
    {
        InitializeComponent();

        MainPage = new MainPage();
        
        // Initialize the Everlytic with a configuration string
        Everlytic.Instance
            .SetTestMode(true) // optional. Default is false
            .Initialize("cD0zNTNmMDg5OS0yY2JiLTQyZTUtYTUyMi04YjgxNzY4ZGQ4NGE7aT1odHRwOi8vbXkudXJsLmNvbQ==");
    }
}

```

### Android

```c#

using Com.EverlyticPush;

[Application]
public class App : Application
{
    public App(IntPtr javaReference, JniHandleOwnership transfer) : base(javaReference, transfer)
    {
    }

    public override void OnCreate()
    {
        base.OnCreate();
            
        // Initialize the Everlytic SDK with a configuration string
        Everlytic.Instance
            .SetTestMode(true) // optional. Default is false
            .Initialize("cD0zNTNmMDg5OS0yY2JiLTQyZTUtYTUyMi04YjgxNzY4ZGQ4NGE7aT1odHRwOi8vbXkudXJsLmNvbQ==");
            
        // Initialize the Everlytic SDK using AndroidManifest.xml configuration string
        Everlytic.Instance.Initialize();
    }
}
```
***
## Using the SDK
### Subscribing a Contact

```c#
public void SubscribeContact(string email) 
{
    // If you don't require a success or fail result
    Everlytic.Instance.Subscribe(email);
    
    // If you want to get the success or failure result of the subscription call
    Everlytic.Instance.Subscribe(email, result => 
    {
        // Handle the result. Note this does not return on the main thread
    });
}
```

### Unsubscribing a Contact

```c#
public void UnsubscribeContact() 
{
    // If you don't require a success or fail result. 
    // Unsubscribes the current contact
    Everlytic.Instance.Unsubscribe();
    
    // If you want to get the success or failure result of the subscription call
    Everlytic.Instance.Unsubscribe(result => 
    {
        // Handle the result. Note this does not return on the main thread
    });
}
```

### Retrieve the Notification History

```c#
public void GetNotificationHistory() 
{   
    Everlytic.Instance.GetNotificationHistory(resultSet => 
    {
        // Handle the result set
    });
}
```

### Basic Customization

- Change the default icon by adding a `ic_ev_notification_small` drawable
- Change the default color by updating the `styles.xml` `colorPrimary` property  