# Everlytic Push Notification Documentation

Everlytic Provides the ability to send out Push Notifications to mobile devices as well as your website (device agnostic).  

There are three main types of Push Notifications, namely:
1. **Bulk Push Notifications** - Send out a notification to a group of people which is useful for marketing and broadcasts.
1. **Transactional Push Notifications** - Send out an individual notification to a specific person. For example: "Your invoice is due". [See Documentation](./transactional.md)
1. **Push Notifications in a Workflow** - Similar to Transactional Push, but it can be part of a workflow with other types of messages.
 
## General Setup
The basic overview of how you will set up Push Notifications is the following:
1. Set up a specific push notifications list in Everlytic
1. Add our SDK to your app/website.
1. Get people to subscribe through our SDK on your app/website
1. Send out a bulk push notification to a group of people / send out an individual push notification to a single person

## SDK Documentation
- [Web Push Documentation](./web/readme.html)
- [Firebase App Implementation No-SDK Documentation](./no-sdk/readme.html)
- [Android SDK Documentation](./android/readme.html) [Deprecated]
- [Xamarin (Android Only) SDK Documentation](./xamarin/readme.html) [Deprecated]


## FAQ
- [How do I add contacts to a Workflow that has Push Notifications?](./FAQ/workflows/readme.md)
- [How do I send a Transactional Push Notification?](./transactional.md)
