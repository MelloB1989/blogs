# The Story of Juzzt: A Lesson in What Not to Do When Building Apps

## What is This About?

A few days ago, I came across a dating app called **Juzzt**, created by some students from IIT Bombay. Naturally, I was intrigued—dating apps are tricky products to build, requiring a great deal of care and thought when it comes to security, data privacy, and user experience. Unfortunately, what I found was a complete disaster.

Juzzt is not only poorly built, but its developers made critical mistakes that exposed user data, sensitive information, and even their backend infrastructure to exploitation. This blog details the vulnerabilities I discovered, why they’re serious, and what lessons developers should take from this debacle.

---

## Developers, This is Reckless

First off, let me point out the sheer irresponsibility of allowing **real users to register and use the app** while it’s still in development. There was no warning anywhere that this app was a work-in-progress, no disclaimers about potential security risks, and no indication that users’ data might be at risk.

This is especially disappointing given that the app was built by students at **IIT Bombay**, one of the top engineering institutions in the country. I expected more from them.

Even worse, they decided to use **Firebase as their backend**. While Firebase can be convenient for quick prototyping, it’s not exactly known for being robust or secure for production apps—especially when the developers haven’t bothered to configure it properly. If you’re going to use Firebase, at least **do it right**.

---

## Misconfigured Firebase Rules: A Recipe for Disaster

One of the biggest issues I found was with their **Firebase configuration**. It was laughably bad. Let me explain:

1. **No Proper Security Rules**: They didn’t set up security rules to restrict access to user data. This meant that anyone with access to the Firebase endpoint could fetch sensitive user data. I was able to pull data on **300 users**, including their phone numbers, email addresses, and more.

2. **Brute-Force Vulnerability**: Their Firebase API key was publicly exposed, and they didn’t implement multi-factor authentication (MFA) for logins. This makes it trivial for an attacker to brute-force user passwords.

---

### How to Configure Firebase Properly

Here’s a quick primer on how they could have avoided this:

1. **Secure Database Rules**:
   Firebase rules should follow the principle of least privilege. For example:

   ```json
   {
     "rules": {
       "users": {
         "$uid": {
           ".read": "auth != null && auth.uid == $uid",
           ".write": "auth != null && auth.uid == $uid"
         }
       }
     }
   }
   ```
   This ensures that only authenticated users can read or write their own data.

2. **Restrict API Key Access**:
   Use the Firebase console to restrict API keys by IP address or usage context.

3. **Enable Multi-Factor Authentication (MFA)**:
   Enforce MFA for all user accounts to protect against brute-force attacks. Firebase provides built-in support for this—there’s no excuse not to use it.

4. **Use Firebase Functions for Sensitive Operations**:
   Instead of exposing direct database access, sensitive operations should be handled via Firebase Functions with proper authentication and rate limiting.

---

## Hardcoding the Gemini API Key: Why This is Dumb

Another colossal mistake they made was **hardcoding their Gemini API key** directly in the frontend. The API key was:

```
AIzaSyD6RSrE3XvkGhtiN9q257-GNOzLJLdRt7g
```

This is like leaving your house key under the doormat and announcing it to the neighborhood. Anyone who decompiled the app or inspected the network requests could see the key and use it to abuse their services.

**How to Fix This:**

1. **Never Hardcode Keys in the Frontend**:
   API keys should always be stored securely on the backend.

2. **Rate Limit API Access**:
   Even if an attacker gets hold of your key, rate-limiting requests on the server side can minimize the damage.

---

## Public S3 Bucket: A Disaster Waiting to Happen

The app also included a **direct link to its APK** hosted on an **open S3 bucket**. Public S3 buckets are a terrible idea, and here’s why:

1. **Exposed to DDOS Attacks**:
   Anyone can flood your bucket with requests, running up your AWS bill in no time.

2. **Unsecured Access**:
   If someone manages to upload malicious files to your bucket, they can spread malware to unsuspecting users.

**How to Fix This:**

1. **Make S3 Buckets Private**:
   Use IAM roles to ensure that only authorized services or users can access your bucket.

2. **Use CloudFront with Signed URLs**:
   Serve your files via CloudFront and generate time-limited signed URLs for secure access.

---

## Appreciate the Idea, But Execution Matters

To be fair, the idea behind Juzzt isn’t bad. Building a dating app is ambitious, and I can appreciate the effort it takes to execute such a project. But here’s the thing: **execution matters**.

Even if the app is still in progress, these are **basic, level-0 things** that any developer should know. Leaving API keys exposed, misconfiguring Firebase, and failing to secure user data are rookie mistakes that can lead to catastrophic consequences.

The bottom line? **Build responsibly**. Take your users’ security and privacy seriously—because if you don’t, someone else will exploit your mistakes.
