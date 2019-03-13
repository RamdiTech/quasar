---
title: Security Do's and Don'ts
---
**DO** periodically review the security of your application, because any lapse may be putting yourself, your team, your users and even your server at risk of serious exploitation. **DON'T** ignore this page because you think you know everything.

We have collected some best practices for those of you new to the security theater and a few insights for security professionals new to the Vue ecosystem. We will be revising and adding to this document as we become aware of risks through our own research and the publications of the amazing security community.

<center><img src="https://cdn.quasar-framework.org/img/secure-gold-hero.jpg" style="max-width:100%" /></center>

## Vulnerability Reports

If you have found a potential security threat, vulnerability or exploit in Quasar or one of its upstream dependencies, please **DON'T** create a pull-request, **DON'T** file an issue on Github, **DON'T** mention it on Discord and **DON'T** create a forum thread.

**DO** reach out to the team by sending an email to [security@quasar-framework.org](mailto:security@quasar-framework.org) - we will investigate and work with you to triage this issue and help you to report it if appropriate. At the current time we do not have the financial ability to reward bounties, but in extreme cases will at our discretion consider a reward.

<img src="https://cdn.quasar-framework.org/img/audit-sample-badge.png" style="float:right;max-width:15%;min-width:150px;padding-top:50px" />

## Get Help!

Did you know you can apply to book the Quasar team to perform a Security Audit for your project? Contact us to find out more about how to get, validate and publish an official timestamped and version-locked audit badge.



[security@quasar-framework.org](mailto:security@quasar-framework.org)

## Vue Security Risks

### User Input and the Dangers of v-html

The `v-html` directive is a wonderful way to programmatically render markup, but even the Vue docs come with [this warning](https://vuejs.org/v2/guide/syntax.html#Raw-HTML):
> "Dynamically rendering arbitrary HTML on your website can be very dangerous because it can easily lead to XSS vulnerabilities. Only use HTML interpolation on trusted content and never on user-provided content."

If you don't know what that means, take a quick look at what OWASP has to say about [XSS (aka cross-site scripting)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)).

To be fair, this *is* good advice, but **DON'T** be all hand-wavy. **DO** think like an attacker who will innovate, social engineer, lie, phish and steal their way into your systems. What if a webpack loader exploit arises and changes your page in an evil way? What if someone makes a dastardly and ill-intentioned PR? What if suddenly a third party API changes and instead of plaintext starts sending the same structure but with different content? What if the systems you thought were safe turn out to actually have been backdoored? What if a junior dev makes an accidental and fundamentally threatening change to the code that isn't reviewed properly? (Yes, idiocy is sometimes as dangerous as bad intentions!) The point is, **DO** anticipate the unexpected by preparing for the absolute worst case scenario and hardening all of your systems.

**DO** use the `v-pre` directive if you need to take extra precaution.

### vue-i18n
The quasi-official internationali(s/z)ation package for Vue allows you to store html in your key's values and [potentially render them](https://kazupon.github.io/vue-i18n/guide/formatting.html#html-formatting). If users can't modify these values, you should be ok - but make sure you trust (aka review) the translators! Our recommendation (although it is more work and will slow-down HMR) is **DO** use [template interpolation](https://kazupon.github.io/vue-i18n/guide/interpolation.html#basic-usage).

### eval()
Although you may be tempted to use `eval()`, even if you know what you are doing, just **DON'T**.
<center><img src="https://cdn.quasar-framework.org/img/dont-be-eval.png" style="max-width:100%" /></center>



## Quasar Components
Some Quasar components can be enabled to prevent the rendering of "insecure content". This is an opt-in feature (available starting in `quasar@1.0.0-beta.10`) that is performed by adding the `sanitize` boolean prop to the component. These components are discussed below.

### QSelect
If you are not customizing menu-related scoped-slots (i.e. `option` scoped slot), **DO** prevent the component from rendering HTML in the labels and sublabels with the `sanitize` property. Generally speaking, this is not user-supplied data. If you are customizing this slot, it is your responsibility to do sanitization yourself.

### QChat & Emoji
The `QChatMessage` component can similarly be prevented from passing html to the browser by using the `sanitize` property.

::: tip
There have been a number of recent exploits (especially for older Android and iOS devices) where certain emoji and non-standard UTF-8 actually triggered mobile device restarts and boot-screen loops. **DO** consider a devland integration of markdown parsing in a plain-text type of input field and render it to HTML on the server side before you pass it to the chat recipients.
:::

### QInput
Any field that enables users to enter keystrokes, paste from the buffer or drop a file is a security risk. We won't go into the nitty-gritty details of this, but just remember it is YOUR responsibility to maintain safety. Only you can prevent help-desk fires!

### QEditor
This component allows the users to actually create HTML (and even paste it). If you are going to be saving this and showing it to other users, care will be needed on the server-side to validate it. In that case **DO** strip out `<script></script>` and `<iframe></iframe>`. You can visit the [v-html vs. double-moustache](/vue-components/editor#Example--Default-editor) example in the docs to play around with the QEditor component and see what it the two rendering methods will provide. There is NO `sanitize` tag for QEditor. Further, if you create custom buttons, it is your responsibility to make them safe. You have been warned.

## Dealing with Files
So how do you validate and sanitize a file? Well, although this is a bit out of scope for a "front-end-framework", we know that many of you reading this will also be storing user-created files on servers. If you are just storing them (and not processing them in any way), **DO** validate that the file is of the appropriate type by detecting the [magic numbers](https://en.wikipedia.org/wiki/List_of_file_signatures). **DO** consider using ClamAV to check files for known viral signatures.

### Images
If you are allowing users to upload images to your server, you should know that many commonly used modules merely check the file suffix. It is trivial to craft an image that only superficially appears to be an image. **DO** verify that the file is what it claims to be by checking the magic numbers and for this consider using [is-image](https://github.com/sindresorhus/image-type). While you could check the magic number in the browser [with this method](https://medium.com/the-everyday-developer/detect-file-mime-type-using-magic-numbers-and-javascript-16bc513d4e1e), another approach is to let the user load an image into a canvas and then upload directly from the canvas. [Vue-croppa](https://github.com/zhanziyang/vue-croppa) is great front-end tool to do this.

### Archives
Archive decompression attacks for directory traversal are a real security issue and are virtually impossible to detect without decompressing the file. If you can get away with NOT accepting this type of media, then do it. Otherwise, on linux **DO** use the humble `less` / `lesspipe` and `.lessfilter` for preprocessing these files with your custom workflows.

## Passwords
**DON'T** save passwords in plaintext, in fact - **DON'T** save them at all. **DO** save secure hashes and compute them in memory according to a scheme using secure salt and proper algorithms. **DON'T** limit the length of passwords. **DO** consider a highly secure application flow for password resetting and enable the user to configure it according to their preferences. This is a process unique to every project, so we can't tell you how to solve the problem. Nevertheless here are a few good links:

- [OWASP cheatsheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Choosing_and_Using_Security_Questions_Cheat_Sheet.md)
- [FIDO Guidelines](https://fidoalliance.org/recommended-account-recovery-practices/)

## Cryptography
 - **DON'T** create your own cryptographic solution
 - **DON'T** store personal information in plaintext
 - **DON'T** create your own cryptographic solution
 - **DON'T** ignore any aspect of implementation details
 - **DON'T** create your own cryptographic solution
 - **DON'T** use MD5 or SHA1
 - **DON’T** create your [own cryptographic solution](https://about.unimelb.edu.au/newsroom/news/2019/march/researchers-find-trapdoor-in-swissvote-election-system)

A great place to read about this topic and properly choose an industrial strength solution is [libsodium](https://download.libsodium.org/doc/)


## Distribution

::: tip
If someone wants to change something in your database or add some file to the server and they are not using an SSH key, **DO** validate && sanitize the input.
:::

### Web
- **DO** Use https / wss
- **DON'T** use http
- **DO** manually audit your certificates
- **DO** validate users
- **DO** remember that JWT isn't encrypted per sé
- **DON'T** store sensitive data in JWT
- **DO** use JWE instead of JWT and use AES256 CBC + HMAC SHA512
- **DO** double-down and perform the complete OWASP web audit

### Cordova
- **DON'T** use iframes
- **DO** sign all your builds
- **DON'T** package for Android Gingerbread
- **DO** encrypt all data at rest

The [Cordova Docs Page](https://cordova.apache.org/docs/en/latest/guide/appdev/security/) goes into detail about securing Cordova, and although it seems outdated, the information is mostly still on point.

### Electron
Electron is a very special case, because XSS and remote code injection can actually lead to complete compromise of the end-user's (or even developer's) device.
- **DON'T** disable `websecurity`
- **DON'T** enable remote code execution
- **DO** read our guidelines for enhanced [Electron Safety](/quasar-cli/developing-electron-apps/electron-security-concerns).

## Environmental Safety
Being more safe means taking many things into consideration, and the more of the following guidelines you respect, the smaller the attack footprint will be.

<center><img src="https://cdn.quasar-framework.org/img/lets-encrypt.jpg" style="max-width:100%"/></center>

### Operational Security
Audit how your development systems work:
 - **DO** use an OS and distro with a smaller footprint and security features enabled (like SELinux for example)
 - **DO** make sure ALL software on your machine is up to date (especially NODE)
 - **DO** use a password manager
 - **DO** Use 2FA everywhere possible
 - **DON'T** retain unneeded software

Audit how your production environment works:
 - **DO** turn off password and root access to your server
 - **DO** use secure transfer protocols (SSH, HTTPS, SFTP, WSS)
 - **DO** install fail2ban and rkhunter
 - **DO** regularly analyze your logs
 - **DO** encrypt data at rest
 - **DO** use advanced media-type analysis
 - **DO** use ClamAV to detect infected files
 - **DO** undertake regular system maintenance
 - **DO** remove old ciphers from permitted / available types
 - **DON'T** think security through obscurity will help you when you are under attack
 - **DON'T** leave unneeded ports open
 - **DON'T** pretend containers or VM's keep you safe by their nature
 - **DON'T** ever stop being paranoid

### Organizational & Repository Security

This is something that every team should have on their radar and put some thought into. **DO** consider who has access to your repositories, how commits are merged and how assets are published. Here are some good things to remember:

 - **DO** require a review before merging to master
 - **DO** require 2FA for reviewers / code committers
 - **DO** require signed commits
 - **DO** take Github Security Warnings seriously
 - **DO** undertake deep code reviews
 - **DO** review critical third-party libraries, especially any working with real files
 - **DO** pin versions of critical libraries
 - **DO** commit package lock files
 - **DO** Add `.env` files to your `.gitignore`
 - **DON'T** put sensitive data in your source code
 - **DON'T** ignore `yarn audit` or `npm audit` reports
 - **DON'T** blindly rely on third-party services

## Final Note
Security is not peace of mind, it is a practical application of knowledge that requires vigilance and awareness. **DON'T** stop being concerned about security and **DON'T** think you are doing enough. There is always more you can undertake, there are constantly new vulnerabilities to be aware of. But the biggest security threat of them all is laziness, so put your outside shoes on, scroll back up the page and **DO** read the [OWASP link about XSS](/quasar-cli/security#User-Input-and-the-Dangers-of-v-html). We won't tell anybody.