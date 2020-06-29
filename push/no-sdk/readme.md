# Implement System/Firebase Push Notifications in your app 
You can implement Push Notifications in your app by interpreting the payload received and manually calling our API for us to record stats.

## Overview
The way we send push notifications to your app is through [Firebase](https://firebase.google.com/). The basic flow is the following:
1. Grab the Firebase service key for your Firebase project, assign it to a list in the System.
1. Get the configuration string given to you by the System, to use in your app.
1. Call the System's subscribe API endpoint in your app to subscribe contacts to the System.
1. Send a push notification From the System. We use the Firebase key to call Firebase's API to deliver the push notification to your app.
1. Interpret the payload received from Firebase in your app.
1. Call our stats API endpoint to tell the System about a delivery.
1. Call our click/dismissal API endpoints if a contact clicked or dismissed the notification.

## Prerequisites
- A Firebase Project set up for your app. See the official Firebase docs [here](https://firebase.google.com/docs/android/setup).
- **Note:** Make sure that the FCM API is enabled on your Firebase Account.
- A Push Notification Project set up inside the System

## Using the the System configuration string
Copy your Push Project SDK configuration string from the System. See [Setting Up Your Push Project](../list_setup.md).

This configuration string is simply a base64 encoded string that contains settings you are going to need to continue.

Example:
The following configuration string is pulled from the System: 

```
cD14eHh4eHh4eHgteHh4eC14eHh4LXh4eHgteHh4eHh4eHh4eHg7aT1odHRwczovL2xpdmUxMC5ldmVybHl0aWMubmV0
```

When we base64 decode this string, we get the following:

```
p=xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx;i=https://live10.everlytic.net
```

This is a string of variables separated by semicolons.

| Variable Name | Description                                                                                                                                            |
|:--------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `p`           | The 36 character UUID of the project. You will need to use this for all API calls as a HTTP Header and also as a parameter for the `subscribe` method. |
| `i`           | This is the URL that you will use with the endpoints listed below.                                                                                     |


## Interpreting the payload of the Push Notification
When you receive a push notification from the System through Firebase, you will need to interpret the payload that your app receives from Firebase. We use the `Data Message` Firebase message type. See the [Firebase Docs](https://firebase.google.com/docs/cloud-messaging/concept-options) for more information on this type.

The data message payload may have the following parameters:

| Parameter Name      | Description                                                                                                                                        |
|:--------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------|
| title               | The title of the Push Notification set up in the System during composition                                                                          |
| body                | The body of the Push Notification set up in the System during composition                                                                           |
| message_id          | The unique identifier of the message being received. This will be used when calling our API to let us know which message to tie the stats back to. |
| @default            | This is a deprecated field, and you can safely ignore it                                                                                           |
| $<custom_attribute> | All custom attributes set up during composition will start with a $ symbol. So these are completely dynamic.                                       |

Here is an example of the data message payload:
```json5
{
  // ...There may be other data above this
  "data": {
    "title": "Pharmacy Notification",
    "body": "Hi Joe, your script is ready for collection at the Randburg Pharmacy.",
    "message_id": "13",
    "$category": "Pharmacy",
    "@default": "launch="
  }
}
```

## Calling our API
You will need to call the the System API to record stats for the following:
- Subscribe (Call this when you need to subscribe a contact for Push Notifications)
- Unsubscribe (Call this when you need to unsubscribe a contact from Push Notifications)
- Deliveries (Call this when the notification gets displayed)
- Clicks (Call this when the user clicks on the notification)
- Dismissals (Call this when the user closes / swipes away the notification)

### How to call our API
- All of the API methods used for Push Notification use their own API, which is different from the System's standard API. We use the Project UUID (From earlier) to authenticate you rather than a username and API key.
- You will need to provide the Project UUID as the custom HTTP Header `X-EV-Project-UUID`. You can see examples of this down below.
- You will also need to set the `Content-Type` HTTP header to `application/json` 

### Subscribe
The subscribe method can be called whenever you want to subscribe the contact to Push Notifications. The System will try and find a contact with either the Email Address or Unique Identifier and if the contact exists, it will subscribe them to Push Notifications, else it will create a new contact and subscribe this new contact to Push Notifications using the Email Address given.

| Endpoint                                   |
|:-------------------------------------------|
| ```POST /servlet/push-notifications/subscribe``` |

Here is a list of the parameters and their descriptions:

| Parameter Name      | Description                                                                                                                                                                                                                                                                                                                               |
|:--------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| push_project_uuid   | The Project UUID from the project settings from earlier                                                                                                                                                                                                                                                                                   |
| contact             | This is the contact object that is used to uniquely identify the contact on the System, it contains 2 required options and 1 optional parameter                                                                                                                                                                                            |
| contact.push_token  | This is the FCM push token provided by the Firebase SDK in your app.                                                                                                                                                                                                                                                                      |
| contact.email       | The email address used to identify the contact in the System. If you would prefer to use the unique ID, you can do that too.                                                                                                                                                                                                               |
| contact.unique_id   | Set this parameter if you would like to use the unique ID to identify the contact in the System instead of the email address. You must still pass the email address along with this so that if the contact doesnt exist, we can add it using the email address.                                                                            |
| platform            | The platform object contains details about the platform                                                                                                                                                                                                                                                                                   |
| platform.type       | The type of platform used. Example: "Android"                                                                                                                                                                                                                                                                                             |
| platform.version    | The version of the platform used.                                                                                                                                                                                                                                                                                                         |
| device              | The device object containing details about the device.                                                                                                                                                                                                                                                                                    |
| device.id           | A UUID for this device that you need to uniquely generate for each device. You need to use UUID Version 4 to generate this, which is a 36 character string. We recommend you generate this once and store it for all future requests. [See the UUID Docs](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_(random)) |
| device.manufacturer | The manufacturer of the device, e.g. Samsung                                                                                                                                                                                                                                                                                              |
| device.model        | The model of the device, e.g. S5                                                                                                                                                                                                                                                                                                          |
| device.type         | The type of device, e.g. Handset                                                                                                                                                                                                                                                                                                          |
| metadata            | The metadata object. This is currently not used, but it is still required, so just provide an empty object for this.                                                                                                                                                                                                                      |
| datetime            | The date and time that the request was made. This needs to be in the format of [ISO8601](https://en.wikipedia.org/wiki/ISO_8601)                                                                                                                                                                                                          |

Here is an example of calling the subscribe method:

```
POST /servlet/push-notifications/subscribe HTTP/1.1
X-EV-Project-UUID: xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
Content-Type: application/json; charset=utf-8
Host: live10.everlytic.net
Connection: close
{
    "push_project_uuid":"xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "contact":{
        "email":"example@senderguide.com",
        "push_token":"dzAeHJ58jnU:APA91bFK-AupLmYnlrWpNwh9fgnAKcnyHvlpATVi2fqG04rcTjYUOJ0tVVdGOzhzME0l_nxCiWtPAnkWXq47361qW2K20tpm57tdfQMklAqoDw1_L_Kg50_SRRQwgFHjAM3gFLsOJzqy"
    },
    "platform":{
        "type":"android",   
        "version":"5.0.1"
    },
    "device":{
        "id":"95fe9437-7251-44bb-9522-adc00be89944",
        "manufacturer":"macos",
        "model":"paw",
        "type":"handset"
    },
    "metadata":{},
    "datetime":"2018-11-23T11:05:47.000Z"
}
```

If your request is valid, you will get a JSON Result that will be structured like the following:

| Result Variable Name             | Description                                                                                                                                                                                                                                                                                         |
|:---------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| status                           | This will either be "success" or "error". "success" means that a new subscription was created, but "error" does not necessarily mean that the request failed. "error" will also appear if the subscription already exists, but you will still get a subscription object being returned (see below). |
| data                             | The object that contains any additional data about the response                                                                                                                                                                                                                                     |
| data.messages                    | An array that contains more information about the status of the request.                                                                                                                                                                                                                            |
| data.subscription                | The subscription object that was stored in the System                                                                                                                                                                                                                                                |
| data.subscription.pns_id         | The subscription identifier (You will need to save this and use it for all the other API requests)                                                                                                                                                                                                  |
| data.subscription.pns_contact_id | The System's contact identifier                                                                                                                                                                                                                                                                    |
| data.subscription.pns_status     | The status of the subscription in the System. This can either be "active" or "inactive". Most of the time this will be "active" because this is a result from the subscribe method.                                                                                                                  |

And here is an example of a Response that gets returned from the System for a brand new subscription:
```json
{
    "status": "success",
    "data": {
        "subscription": {
            "pns_id": "123",
            "pns_list_id": "233",
            "pns_customer_id": "1",
            "pns_contact_id": "5718",
            "pns_device_id": "1c11941d-ad50-47f8-aefc-8434cd05d567123",
            "pns_status": "active",
            "pns_token_hash": "15c2c03c1252a3d29a4dcd56756444b04ad21cdd43d5d0bc8793ae4b7c497c47"
        }
    }
}
```

Another example of a Response that gets returned from the System if the contact has already been subscribed. (This is still a valid request, even though the status is "error", you will still get a subscription object back)
```json
{
    "status": "error",
    "data": {
        "messages": [
            "This token is already subscribed to Push Notifications on this list."
        ],
        "subscription": {
            "pns_id": "123",
            "pns_contact_id": "5718",
            "pns_customer_id": "1",
            "pns_list_id": "233",
            "pns_device_id": "1c11941d-ad50-47f8-aefc-8434cd05d567",
            "pns_status": "active",
            "pns_token_hash": "15c7c03c1252a3d29a4dcd56756444b04ad21cdd43d5d0bc8793ae4b7c497c47"
        }
    }
}
```

Note: You will need to store the subscription ID (pns_id), so that you can use it for the other API calls.

### Unsubscribe
The unsubscribe method can be called whenever you want to unsubscribe the contact from Push Notifications. (Example: when the user logs out of the app) 

| Endpoint                                     |
|:---------------------------------------------|
| `POST /servlet/push-notifications/unsubscribe` |

Here is a list of the parameters and their descriptions:

| Parameter Name  | Description                                                                                                                      |
|:----------------|:---------------------------------------------------------------------------------------------------------------------------------|
| subscription_id | The ID of the subscription you are unsubscribing. (This is the ID returned from the subscribe method)                            |
| device_id       | This is the same device ID that you generated for the subscribe method, to uniquely identify the device.                         |
| datetime        | The date and time that the request was made. This needs to be in the format of [ISO8601](https://en.wikipedia.org/wiki/ISO_8601) |
| metadata        | The metadata object. This is currently not used, but it is still required, so just provide an empty object for this.             |

Here is an example of calling the unsubscribe method:

```
POST /servlet/push-notifications/unsubscribe HTTP/1.1
X-EV-Project-UUID: xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
Content-Type: application/json; charset=utf-8
Host: live10.everlytic.net
Connection: close
{
    "subscription_id":"123",
    "device_id":"95fe9437-7251-44bb-9522-adc00be89944",
    "metadata":{},
    "datetime":"2018-11-23T11:05:47.000Z"
}
```

This will return a status of either "success" or "error", letting you know whether the unsubscribe was successful or not.

### Events
There are three events that all have the same data parameters, but they just have different endpoints. 

| Endpoint                                      | Description                                                                                |
|:----------------------------------------------|:-------------------------------------------------------------------------------------------|
| `POST /servlet/push-notifications/deliveries` | The endpoint to call when the notification gets displayed                                  |
| `POST /servlet/push-notifications/clicks`     | The endpoint to call when the contact clicks/taps on on the notification                   |
| `POST /servlet/push-notifications/dismissals` | The endpoint to call when the user closes or swipes away the notification (No interaction) |

| Parameter Name  | Description                                                                                                                                                        |
|:----------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| subscription_id | The ID of the subscription you are unsubscribing. (This is the ID returned from the subscribe method)                                                              |
| message_id      | This is the message id that is used to tie the stats to the correct message. You will get this from the data payload of the push notification when you recieve it. |
| datetime        | The date and time that the request was made. This needs to be in the format of [ISO8601](https://en.wikipedia.org/wiki/ISO_8601)                                   |
| metadata        | The metadata object. This is currently not used, but it is still required, so just provide an empty object for this.                                               |

Here is an example of calling the deliveries method:

```
POST /servlet/push-notifications/deliveries HTTP/1.1
X-EV-Project-UUID: xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
Content-Type: application/json; charset=utf-8
Host: live10.everlytic.net
Connection: close
{
    "subscription_id":"123",
    "message_id":"12",
    "metadata":{},
    "datetime":"2018-11-23T11:05:47.000Z"
}
```
This will return a response with a status of either "success" or "error"

### History
There is a Push History API Method that you can call to get a list of messages sent to a contact. Note that this will only pull messages sent to the project used in the authentication.

| Endpoint                                           |
|:---------------------------------------------------|
| `POST /servlet/push-notifications/history/contact` |


| Parameter Name    | Description                                                                                                                                                                                                                                                     |
|:------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| contact           | This is the contact object that is used to uniquely identify the contact on the System. One of the following sub parameters are required.                                                                                                                        |
| contact.email     | The email address used to identify the contact in the System. If you would prefer to use the unique ID, you can do that.                                                                                                                                         |
| contact.unique_id | Set this parameter if you would like to use the unique ID to identify the contact in the System instead of the email address.                                                                                                                                    |
| from_datetime     | *Optional parameter* If you want to only pull messages since a particular date, you can provide this parameter. It accepts most formats, but we recommend using ISO8601, (e.g. "2019-09-28T10:25:23+02:00") or Just simple Date Time (e.g. "2019-09-28 10:25"). |

Here is an example of calling the history method:

```
POST /servlet/push-notifications/history/contact HTTP/1.1
X-EV-Project-UUID: xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
Content-Type: application/json; charset=utf-8
Host: live10.everlytic.net
Connection: close
{  
    "contact":{
        "email":"example@senderguide.com"
    },
   "from_datetime" : "2019-09-28 10:20"
}
```

If your request is valid, you will get a JSON Result that will be structured like the following:

| Result Variable Name | Description                                                                                                                                                                         |
|:---------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| status               | This will either be "success" or "error". "error" will appear if you have supplied parameters incorrectly, or the contact doesn't exist or is not subscribed to Push Notifications. |
| data                 | The object that contains any additional data about the response                                                                                                                     |
| data.messages        | An array that contains more information about the status of the request.                                                                                                            |

Each message in the array has the following properties:

| Message Property | Description                                                                                                                                    |
|:-----------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|
| title            | The title of the Push Message                                                                                                                  |
| body             | The main body of the Push Message                                                                                                              |
| date             | The date the message was created by the System User. (This is NOT the sent date)                                                            |
| click_action     | Will be either "open-url" (For Web Push) or "open-app" (For App Push)                                                                          |
| click_action_url | The URL that the browser must go to when a user clicks on the Push. (This is only applicable for Web Push)                                     |
| additional_data  | An array that contains the additional data set up during composition. (This is only applicable for App Push) See example for more information. |

Example of may be returned:

```json
{
    "status": "success",
    "data": {
        "messages": [
            {
                "title": "Test Push",
                "body": "This is a Test Push Notification",
                "date": "2019-09-28T10:25:23+02:00",
                "click_action": "open-url",
                "click_action_url": "http://www.test.com",
                "additional_data": [
                    {
                        "name": "asd",
                        "value": "asd"
                    }
                ]
            },
            {
                "title": "Test",
                "body": "Another test Push Notification",
                "date": "2019-09-28T10:32:02+02:00",
                "click_action": "open-url",
                "click_action_url": "",
                "additional_data": [
                    {
                        "name": "Category",
                        "value": "Promotions"
                    },
                    {
                        "name": "ExternalWebLink",
                        "value": "https://example.com/example"
                    }
                ]
            }
        ]
    }
}
```


