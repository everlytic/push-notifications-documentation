All methods are accessible on the `EverlyticPush.EverlyticPush.Current` object. The object namespace and/or naming are subject to change.

## Available Methods

All methods are available on the `Everlytic.Instance` static class

```c#
void Initialize(string configurationString);
```
Initializes the SDK in the application. Must be called before any other SDK Methods are called.

`configurationString` is optionally passed as an alternative to setting the SDK Configuration string in your `AndroidManifest.xml` file.
****
```c#
void Subscribe(string email);
void Subscribe(string email, OnResultReceivedDelegate onResultReceivedDelegate);
```
Subscribes a contact using an email address. Accepts optional subscribe success callback.
****
```c#
void Unsubscribe();
void Unsubscribe(OnResultReceivedDelegate onResultReceivedDelegate);
```
Unsubscribes the currently subscribed contact from receiving push notifications. Accepts optional unsubscribe success callback.
****
```c#
bool IsContactSubscribed();
```
Returns `true` if a contact is currently subscribed.
****
```c#
bool IsInitialized();
```
Returns `true` if the SDK has been initialized.
****
```c#
void GetNotificationHistory(OnNotificationHistoryResultsDelegate onNotificationHistoryResultsDelegate);
```
Asynchronously retrieves the notification history for the device and returns the results to an `OnNotificationHistoryResults` delegate
****
```c#
int GetNotificationHistoryCount();
```
Efficiently returns the number of messages stored in the notification history.

## Customization
### Android - Default Notification Icon

Change the default icon by adding a new drawable called `ic_ev_notification_small` to your application

### Android - Default Notification Color (where supported)
Notification color is derived from the `styles.xml` `colorPrimary` value

## Known Issues

- The app may crash if there is a network connection issue, E.g. Everlytic domain cannot be resolved