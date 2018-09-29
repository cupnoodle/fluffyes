# Set up Push Notification easily using CloudKit



> How do I generate certificates for push notification?

> Device tokens? APNS? Environment?!

> Is there a way to send push notification without relying on a backend? Firebase? 

> Why the Firebase notification doesn't appear on my device?!



A small task of wanting to send push notification can quickly turn into a nightmare of certificates, device tokens, troubleshooting why push notification works in development environment but not on Testflight etc.

Firebase does ease the pain a bit, but you will need to install the huge Firebase SDK library, still need to generate certificates or keys to use it and requires additional step to debug when push notification doesn't arrive.



"I just want to send a generic push notification to all of my users"

The good news is that Apple's own CloudKit can handle these for you (for free too!), you wouldn't need to worry about certificates / keys / device tokens / APNS if you use CloudKit.



This tutorial assume that

1. Your app doesn't rely on a existing backend server to send push notification.
2. You are enrolled in Apple Developer Program, or have access to one
3. You have a physical iOS device (push notification won't work in Simulator)



Table of Contents:




## Step 1  - Enabling Push Notifications and CloudKit capabilities in your app

In your Xcode Project, select a valid team for Signing if you haven't already :





// cancel subscription in app

ibaction 

loop through subscription and remove them



// example conditional subscribing

// like only receive notification if the user is male/female etc

