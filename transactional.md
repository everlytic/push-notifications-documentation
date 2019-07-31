# Everlytic Transactional Push Notifications

Transactional Push Notifications allow you to send out a push notification to an individual contact. This differs from Bulk Notifications and is useful for contact event based notifications. An example would be something like:  

```Hi John, your script is ready for collection at the Randburg Pharmacy. Please collect before 11 July```

You can achieve these once off transactional notifications by calling our API with the message that needs to go out.

## Prerequisites
1. A project set up in Everlytic and on your app/website - [Documentation](./index.html)
1. The contact needs to have already subscribed to Everlytic from your app/website (We need their token to send to)
1. API Access to Everlytic.

## Getting Started
Sending a Transactional Push Notification makes use of our API 3. If you are unfamiliar with how our API works, you can find the documentation [here](http://help.senderguide.com/api-documentation/the-api-endpoints/the-rest-api/).

### Request
| Verb     | Route |
|:------------|:-----------|
| **`POST`**  | `https://[Your URL]/api/3.0/push-notifications/transactional/send`|

#### Request Data
| Parameter                     | Required | Type                        | Description                                                                                                                                                                                    |
|:------------------------------|:---------|:----------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **project_id**                | yes      | `int`                       | The Project ID that you set up in Everlytic. This can be found on the listing of the Projects on the Push Projects page.                                                                       |
| **category**                  | yes      | `string`                    | A tag to give to the message you are sending. All the messages sent under the same tag will be grouped in Everlytic.                                                                           |
| **contact**                   | yes      | `object`                    | The object containing the contact to send to. See below for the different ways to identify a contact                                                                                           |
| contact.**contact_id**        | no       | `int`                       | If this is supplied, the contact will be looked up using their Everlytic ID                                                                                                                    |
| contact.**contact_email**     | no       | `string`                    | If this is supplied, the contact will be looked up using their email address.                                                                                                                  |
| contact.**contact_unique_id** | no       | `string`                    | If this is supplied, the contact will be looked up using their Unique ID. (This Unique ID can be set by you in Everlytic)                                                                      |
| **message**                   | yes      | `object`                    | The object containing the message details to send to the contact.                                                                                                                              |
| message.**title**             | no       | `string`                    | Title of the message to send out. This can be personalized (See example below).                                                                                                                |
| message.**body**              | yes      | `string`                    | Main body text of the message to send out. This can be personalized (See example below).                                                                                                       |
| message.**ttl**               | no       | `int`                       | Number of seconds that the message will wait to get delivered before it gets discarded. This is useful for time sensitive information, defaults to 72 hours.                                   |
| message.**click_action**      | no       | `"open-url"` / `"open-app"` | What must happen when the contact clicks/taps on the Push Notification. If you are using Web Push, only `open-app` is supported.                                                               |
| message.**click_data**        | no       | `string` / `object`         | If you supplied `open-url` as the click_action, you will provide the URL here. If you supplied `open-app`, you can provide a list of custom key-value-pair attributes to be passed to the app. |

#### Simple Example
```json
{
    "project_id": 2,
    "category": "Pharmacy",
    "contact": {
        "contact_email": "test@everlytic.com"
    },
        "message": {
        "body": "Hello {{contact.contact_name}}, your script is ready for collection at Randburg Pharmacy",
        "click_action": "open-url",
        "click_data": "http://www.example.com"
    }
}
```

#### Complex Example
```json
{
    "project_id": 2,
    "category": "Pharmacy",
    "contact": {
        "contact_email": "test@everlytic.com"
    },
        "message": {
        "title": "Script Ready!",
        "body": "Hello {{contact.contact_name}}, your script is ready for collection at Randburg Pharmacy",
        "click_action": "open-app",
        "click_data": {
            "app-category": "pharmacy-notifications"
        }
    }
}
```