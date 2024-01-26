# iOS Sideloading: The Rundown

Apple [just announced](https://www.apple.com/newsroom/2024/01/apple-announces-changes-to-ios-safari-and-the-app-store-in-the-european-union/) sideloading for the EU on iOS 17.4 and above. What does this mean for you?

> **Note:** Scroll to the bottom of the article for a TL;DR.

## Access control

In iOS 17.3, a new MobileGestalt key called `DeviceSupportsEUCapabilities` was added, which may have something to do with sideloading. If somehow a sandbox escape is released for 17.4, it might be possible to enable this even if you don't live in the EU. However, it may not be worth it. Let's find out.

## Becoming a Marketplace

> All quotes in this section come from [this article.](https://developer.apple.com/support/alternative-app-marketplace-in-the-eu/)

First of all, the changes aren't "sideloading" per se. Instead, Apple is allowing **registered developers** (the ones who pay a $100 fee yearly) to create an "alternative app marketplace". I'll let Apple themselves explain how someone can do that.

> If you’re interested in becoming a marketplace developer in the EU, the Account Holder of your Apple Developer Program membership will first need to agree to the Alternative Terms Addendum for Apps in the EU. Once they’ve agreed, they can submit a request for the entitlement.

Additionally, you must "provide Apple a stand-by letter of credit from an A-rated financial Institution of **€1,000,000**". This makes it impossible for smaller companies (let alone individuals) to open their own app stores, and essentially crushes any hopes of true sideloading by the end user.

## Codesigning

> All quotes in this section come from [this article.](https://developer.apple.com/support/dma-and-apps-in-the-eu/#ios-app-eu)

As you may know, all binaries that run on iOS require a valid code signature. Let's see how this works for the new sideloading mechanism.

> Developers can submit a single binary and will be able to choose alternative distribution options in App Store Connect.

> Apple will encrypt and sign all iOS apps intended for alternative distribution to help protect developers’ intellectual property and ensure that users get apps from known parties.

These two quotes from Apple signify that you will still have to submit apps to Apple for signing if you want them to run on iOS. 

> **Functionality** — Binaries must be reviewable, free of serious bugs or crashes, and compatible with the current version of iOS. They cannot manipulate software or hardware in ways that negatively impact the user experience.

> **Security** — Apps cannot enable distribution of malware or of suspicious or unwanted software. They cannot download executable code, read outside of the container, or direct users to lower the security on their system or device. Also, apps must provide transparency and allow user consent to enable any party to access the system or device, or reconfigure the system or other software.

These two points mean that it will be impossible to install jailbreaks (or jailed tools) through this new "sideloading" mechanism.

## Conclusion

Apple's recent announcement of allowing third-party app stores for customers in the EU is not much more than a case of malicious compliance. They did the bare minimum to comply with the law, and it shows. If you want a better explanation, I highly recommend [this video](https://youtu.be/_6dbNzFD0zM).

## TL;DR

- The new "sideloading" in iOS 17.4 sucks.
- You have to pay Apple not only $99 a year for the Dev Program, but €1,000,000 so Apple can "trust" you.
- All apps still need to be reviewed by Apple.
- It will be impossible to install jailbreaks/jailed tools with this new sideloading mechanism.

## Updates

- 2023-01-26 (shortly after initial publish)
    - Add conclusion
