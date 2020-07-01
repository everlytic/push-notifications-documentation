# Why we cannot guarantee delivery with Web Push
There are many reasons out of our control that affect delivery rates in Web Push, and thus we cannot guarantee delivery.  

### Delayed Delivery

The following are reasons to cause delayed delivery, and once they are resolved, the notification will be delivered (unless the TTL has elapsed):
* The device is off
* The device does not have the web browser active that was subscribed to the Push service (The device does not have to have the browser open and focused, but it has to at least be running in the background)
* The device has intermittent or no internet connection

### Discarded Delivery

Known issues where the device completely discards the Push Notification causing it to never be delivered:
* The device has power saving mode enabled (This is common on some Android devices, and is enabled by default)
* The browser does not have permission on the device to use data in the background
* The user has changed their preferred browser (They will need to resubscribe to Push Notifications on their new browser)
* The user has unsubscribed from Push Notifications (They can do this in many ways, and we don't know about it until it fails to deliver)

Some of the above areas we are able to detect and we mark the subscription as inactive when we try to send to it, but others we are not sure about. So you may be sending to the user again and again, throwing off the delivery rate.