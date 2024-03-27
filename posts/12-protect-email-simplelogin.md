---
title: Protect your Email using SimpleLogin
tags:
  - privacy
  - email
  - simplelogin
date: 2024-03-27
description: Enhance your email security with SimpleLogin, a service that enables the use of anonymous aliases.
icon: 📧
---
> [!warning] Work in progress

---

## Intro

Today, **162 billion** spam emails are sent daily. Protecting your email from leaks due to hacks or companies selling your data may seem practically impossible. However, with the increasing number of data breaches and privacy concerns, it's crucial to take proactive measures to protect your personal information.

There are numerous solutions to this problem, each with its own advantages and drawbacks.

### The Simple One

To dodge spam, some use multiple email addresses - one for junk, one for private matters -but it's not very convenient and the risk of leaking your privacy email is always present.

### The Advanced

Some email services offer aliases with a `+` suffix before the `@` symbol, such as `my-email+alias@domain.name`. This feature is convenient as it's directly linked to your main inbox and can reveal where your email alias has been leaked.

However, many internet services bypass this functionality by outright rejecting all email addresses with a `+`, but also, simply removing the alias after the `+` allows one to retrieve the real email address.

### The Ideal

Other services, such as SimpleLogin, enable users to create aliases without special characters, like a normal email address. These aliases redirect all incoming emails to your personal email address.

![[12-simplelogin-basics.excalidraw.png]]

You can enjoy several benefits:

- The source cannot access your actual email address.
- You have the flexibility to deactivate or reactivate the alias at any time, and the service will promptly cease forwarding the corresponding emails.
- You can easily trace which source leaked your email because from your inbox, you can identify the aliases from which the mail originated.

In this post, I will explain how to use SimpleLogin, a free and open source service, to protect your email, then how to configure a custom domain to increase the profits of using SimpleLogin.

> [!Note] Note
> SimpleLogin is currently integrated with Proton, and if you have a Proton premium account, you will also have access to a SimpleLogin premium account.

---
## Prerequisite

To start using SimpleLogin to protect your email address, you should have:

- A SimpleLogin account (Free or Premium or a ProtonMail account)
- A domain name (not required but recommended for the second part)

---

## SimpleLogin Basics

With SimpleLogin, you can create as many aliases as you need. It's a good practice to use one alias per service and to designate the name of the service as the alias, facilitating easy recall.

### Create an alias

By default, when not using a custom domain name, only specific domain names are accessible. Additionally, an alias is suffixed with a random string to prevent any individual from monopolizing desirable aliases.

To create an Alias, click on the `New Custom Alias`, enter an Alias prefix, then select your mailbox.

![[12-create-alias.png|300]]

> [!note] Note
> I'm currently using the Android app, but there's also a web version available.

### Disable or enable an alias

Just switch on or off the button 🤯

![[12-aliases.png|300]]

### Respond with an alias

This is the tricky part: when you receive an email from SimpleLogin in your mailbox, **do not respond directly** using your personal email, as doing so could compromise your identity.

Instead, SimpleLogin offers reverse-aliases, which are email addresses that forward all incoming emails to another specified address.

Each time a new sender is forwarded, a unique reverse-alias is generated. This alias will then forward all emails to the sender using your chosen alias rather than your personal email.

![[12-simplelogin-reverse-alias.excalidraw.png]]

You can find it by clicking on `Send Email`

![[12-reverse-aliases.png|300]]

### Limitations

In practice, this isn't the ideal solution as it requires creating an alias before establishing an account. Additionally, the randomly generated characters in the email make it difficult to remember. Moreover, the domain names can be traced back to the SimpleLogin service.

However, utilizing a custom domain name can resolve all of these issues.

---
## SimpleLogins Custom Domain

### Advantages

Using a custom domain with SimpleLogin provides a smooth and convenient experience, addressing all the previously mentioned issues.

In fact, using a custom domain removes the necessity of the random string suffix, enabling you to create more personalized and unique aliases. It's also significantly more challenging for the source to ascertain that your alias is, indeed, linked to the SimpleLogin service.   
Furthermore, with the **Catch All** functionality, all aliases can be created on the fly, eliminating the need to creating them before use. 🤯
  
So, in practice, if you have configured the domain `hide.me` with SimpleLogin, when you want to subscribe to a service, you simply replace your private email with an alias like `netflix@hide.me`, and automatically, this alias will be generated on your SimpleLogin account. No action needed.

### Configuration

To configure a custom domain name, you currently need to use the web version, as this functionality is not yet available in the mobile version.

Click on `Domains` Tab, then on the `Create` Button.

![[12-create-custom-domain.png]]

Then, you'll need to verify your ownership of the domain by creating a `TXT` record with a special value. SimpleLogin will use this record to confirm that you are indeed the owner of the domain.

After completing this step, you'll need to add additional records to configure your domain to redirect to the SimpleLogin service:
- `MX DNS` records to specify the mail server responsible for receiving emails for your domain.
- `SPF` (Sender Policy Framework) to authenticate your emails and prevent spoofing.
- `DKIM` (DomainKeys Identified Mail) to add a digital signature to your outgoing emails, ensuring their authenticity.
- `DMARC` (Domain-based Message Authentication, Reporting, and Conformance) to further enhance email authentication and protect against email spoofing and phishing attacks.

![[12-configuration-example.png]]

Don't forget to activate the `Catch All` option, then you can begin using your aliases right away!

---
## Ressources

- SimpleLogin [website](https://simplelogin.io/fr/)
- SimpleLogin [documentation](https://simplelogin.io/docs/)
