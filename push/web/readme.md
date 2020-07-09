# Web Push Introduction
The System provides the ability to send out Push Notifications to mobile devices as well as your website (device agnostic).  

There are three main types of Push Notifications, namely:
1. **Bulk Push Notifications** - Send out a notification to a group of people which is useful for marketing and broadcasts. [See Documentation](http://help.everlytic.com/how-to-send-a-bulk-push-notification/)
1. **Transactional Push Notifications** - Send out an individual notification to a specific person. For example: "Your invoice is due". [See Documentation](../transactional.md)
1. **Push Notifications in a Workflow** - Similar to Transactional Push, but it can be part of a workflow with other types of messages. [See Documentation](../FAQ/workflows/readme.md)
 
## General Setup
The basic overview of how you will set up Push Notifications is the following:
1. [Set up a specific push notifications list in the System](../list_setup.html)
1. [Add our SDK to your website](#getting-started)
1. Get people to subscribe through our SDK on your website
1. [Send out a bulk push notification to a group of people](http://help.everlytic.com/how-to-send-a-bulk-push-notification/) / [send out an individual push notification to a single person](../transactional.html)

# Web Push SDK
This is the Web version of our Push Notification SDK. It enables your website to serve Push Notifications to your user base.

You can find the source code [here](https://github.com/everlytic/push-notifications-sdk-web) 

## Browser Support
Almost all modern browsers are supported on most platforms, with the exception of Apple. See the following table for Push Notification Support:

| Browser     | Windows PC | macOS | Android | iOS (iPhone) |
|:------------|:-----------|:------|:--------|:-------------|
| **Chrome**  | Yes        | Yes   | Yes     | No           |
| **Firefox** | Yes        | Yes   | Yes     | No           |
| **Edge**    | Yes        | *N/A* | No      | *N/A*        |
| **Opera**   | Yes        | No*   | Yes     | No           |
| **Yandex**  | Yes        | Yes   | Yes     | *N/A*        |
| **Safari**  | No         | No    | *N/A*   | No           |

**\* We are working on a fix for these**

## Getting Started
These instructions will get you up and running and receiving Push Notifications on your website in no time.

### Prerequisites
- Website with HTTPS Enabled
- File Access to your website (If you don't have file access to your website, you might be better off using Push Notifications on Landing Pages)

### Installing
**Step 1:** Firstly, will need to create a file called `load-worker.js` at the top level of your website that has the following code:
```javascript
importScripts('https://pushsdk.mailercdn.net/everlytic-push-sw-0.0.1.min.js');
``` 
**It is very important that this file lives in the root of your website and not in a sub folder.**
To ensure that you have added the file correctly, you should be able to go to that file via the URL, e.g: ``https://yoursite.com/load-worker.js``. 

**Step 2:** Next, you will need to add the following code to the `<head>` of your website.
```html
<head>
    <script type="text/javascript" src="https://pushsdk.mailercdn.net/everlytic-push-sdk-0.0.1.min.js" async=""></script>
    <script>
    window.addEventListener('load', function() {
        let SDK = window.EverlyticPushSDK;
        SDK.init({
            hash:"YOUR_HASH_HERE",
            autoSubscribe: true
        });
    });
    </script>
</head>
``` 
The options object that you need to provide to the `init` has the following fields:

- `hash` You will get this hash from the System _(See next section)_.
- `autoSubscribe` Set this option if you would like your website to automatically subscribe people to Push Notifications (You won't need to call the SDK's `subscribe` method manually). _Note that this will pop up a modal asking the contact to enter their email address or to subscribe anonymously._
- `installImmediately` Set this to true if you would like the SDK to be available immediately after initializing it. **If this is set to true, the page will reload the first time a contact lands on the page.** Without this setting, the contact will only be able to be subscribed once the contact has either reloaded the page, or navigated to another part of the website.
- `debug` Set this option if you would like to see debug console output and disable the pre-flight check caching.  

The `init` method returns a promise if you would like to call subscribe immediately after initializing. 

## Getting a hash from the System
The System needs to be set up to have a special list to send Push Notifications to. We recommend using a newly created list for this, but you can also use one of the lists you have already set up.
1. Navigate to the `Push Projects` submenu under `Push Notifications` menu item. If your list is already set up here, you can jump to Step 3.
2. Click on the `Add Push Notification Project` button. This should bring up a modal with some options. Set the `Project Type` to `Web Push`, select your list and save.
3. You should see your list appear in the `Linked List` listing. On the right, click on `SDK Configuration`. This will bring up a modal with the `hash` that you need to initialize the SDK.

## General SDK Usage
There are three main methods that you can call on the SDK. They all return a promise that contains a result:
- `subscribe(contactObject)` This method you can call manually to subscribe a contact using an email address to identify the contact. If the contact doesn't exist, it will create one. There is a second parameter which can disable the stylized double opt-in modal from popping up if you don't want it.

    See the following example:
    ```javascript
    //... This code comes after the SDK init method
    SDK.subscribe({
        'email' : 'example@example.com',
    }, true // This is an optional second parameter to disable the double opt-in modal from popping up. 
    ).then(function(result) {
        console.log(result) // Do something with the result.
    }).catch(function(error){
        console.error(error); // Something went wrong.      
    });
    ``` 
    Typically the email address you supply would come from the logged in user to your website. If you don't have access to the user's email address, we suggest you use the `subscribeAnonymous` or `subscribeWithAskEmailPrompt` methods instead _(See below)_.
    
    
    
- `subscribeWithAskEmailPrompt()` This method is very similar to the first `subscribe` method, but instead of you passing a contact object with the email address, a prompt will ask the contact to enter their email address to subscribe with. Useful if you don't have access to the contact's email details, but don't want to send anonymously. The contact will still be given the option to subscribe anonymously. This method gets called automatically if you added the `autoSubscribe` option in the SDK `init` method.
    ```javascript
    //... This code comes after the SDK init method that did not supply the autoSubscribe option
    SDK.subscribeWithAskEmailPrompt().then(function(result) {
        console.log(result) // Do something with the result.
    }).catch(function(){
        console.error('Something bad happened');      
    });
    ```
    
    - This method takes an optional `options` object, and you can provide a force flag to ignore that the contact has already subscribed, and ask them permission again. For Example:
    
     ```javascript
    //... This code comes after the SDK init method that did not supply the autoSubscribe option
    SDK.subscribeWithAskEmailPrompt({force:true}) 
    //... Rest of the code
    ```

    
- `subscribeAnonymous()` You can call this method if you don't have access to the contact's email address and you also don't want to prompt the contact to enter their email address. It will subscribe all devices under the anonymous contact in the System.
    ```javascript
    //... This code comes after the SDK init method that did not supply the autoSubscribe option
    SDK.subscribeAnonymous().then(function(result) {
        console.log(result) // Do something with the result.
    }).catch(function(error){
        console.error(error); // Something went wrong.      
    });
    ```
    
     
- `unsubscribe()` This method should be called when the user no longer wants to receive Push Notifications. We recommend that you add a button somewhere on your website that allows the user to do so. 
    ```javascript
    //... This code comes after the SDK init method
    SDK.unsubscribe().then(function(result) {
        console.log(result); // Do something with the result.
    }).catch(function(error) {
        console.error(error); // Something went wrong.      
    });
    ``` 
    
## Advanced SDK Usage
The way you want to use the Push Modal on your website can be changed in many ways, using the above options. 

### Customization Options for the preflight modal
If you choose to use the preflight modal, know that it is quite customizable using some hidden `preflight` options. See the following example when setting up the SDK:

```javascript
SDK.init({
    hash:"YOUR_HASH_HERE",
    autoSubscribe: true,
    preflight: {
        title: "We would like to send you notifications for the latest news and updates.",
        message: "Unsubscribe anytime in your browser settings.",
        icon: "https://pushsdk.mailercdn.net/icon.png",
        primaryColour: "#94d229"
    }
});
```
Above are the default variables that get used if you don't provide any `preflight` options. But you may override any of these variables, and they will change on the modal that pops up.

## Sample App
Look at our [Sample App](https://github.com/everlytic/push-notifications-web-sample-app) that implements the SDK and the above methods. Just be sure to replace the hash with the one you get from the System.

## Error Codes
At some point you might get an error response with certain codes. Here are some common codes and what they mean:

| Code          | Description   |
|:--------------|:--------------|
| 403, 404, 410 | Invalid Token |
| 400           | Unknown Error |

## FAQ

* [Why are some of my users not receiving their Push Notifications?](./faq/delivery_problems.html)
* [Why are my delivery rates low?](./faq/delivery_problems.html)
* [How do I set up a Push Notification list](../list_setup.html)
* [How do I add contacts to a Workflow that has Push Notifications?](../FAQ/workflows/readme.md)
* [How do I send a Transactional Push Notification?](../transactional.md)
