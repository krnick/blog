---
title: Android Malware Analysis - Roaming Mantis
date: 2021-09-30 18:02:23
tags:
- android
- malware
---

# Analyze Roaming Mantis with Quark-Engine

![](https://i.imgur.com/HSJF3nj.png)

# Introduction

Roaming Mantis is a notorious malware that was first discovered in 2018 and aims at the Asian region. In the past two years, it has evolved and spread around the world. This malware intends to steal personal sensitive data (e.g. account information, SMS messages, and voice calls). Also, the malware bypassed the two-factor authentication by monitoring SMS messages.

According to the report from Kaspersky Lab, the main distribution of this malware is using DNS hijacking through a compromised router. As long as the user connects to the router, their DNS lookup will be redirected to the malicious URL. After the user connects to the malicious website, they will be prompted to download the Google update application, which turns out to be the Android malware.

In this report, we focus on the APK downloaded by victims who were redirected to the malicious URL. We aim at showing how malware analysts can use Quark Engine to quickly understand what this malware does to the victim.

This malware contains a DEX file encoded by the base64 algorithm. Therefore, we will first demonstrate how we can use sets of detection rules to quickly find where to decode the base64 algorithm and where to load the DEX file in the malware.

After decrypting the DEX file. We then further investigate malicious activities in the DEX file with sets of our detection rules. Also, we prove that obfuscation techniques are useless due to the magic design of Quark Engine.

All in all, we show that by using Quark and detection rule sets, malware analysis can be so much fun.


# Investigating the Android APK

## Summary Report for the APK

![](https://i.imgur.com/Ycu9DTH.png)

In this report, our engine found 10 potential malicious activities with detection rules accordingly. As for the confidence, scores, and weight, please take a look at our talk at DEFCON Blue Team Village videos on YouTube. We explain everything there.

{%youtube XK-yqHPnsvc %}

The scoring system will only take effect if we have enough detection rules. Before we accumulate enough rules, we set the scores and weights all the same. Therefore, the risk levels and total scores are for reference only.

After generating the summary report, we then use an automatic technique to classify these 10 potential malicious activities.


## Exploring Malicious Activities
### Rule Classification: onCreate
![](https://i.imgur.com/8Q1Ajf7.png)

The picture above shows that 5 suspicious activities were found under the function onCreate.

This picture can help malware analysts to understand the malware in an easy way.

As shown above, we found five behaviors in function onCreate. Despite that rules are not listed in the right order, we can still piece them together and tell a story.

With the descriptions in the table, we can simply and quickly guess that this function decodes and load the suspicious payload.

After validating through reading the smali-like source codes, our guess is right!

And the right order of the behaviors is:
1. Get the absolute path of the file and store it in a string.
2.  Open a file from the given absolute path of the file.
3. Read a file from the assets directory.
4. Write file after Base64 decoding.
5. Load additional DEX files dynamically.

### Rule Classification: a
![](https://i.imgur.com/cB0ke4D.png)

As shown above, it is obvious that function a is about to do method reflection, MAGIC!

### Rule Classification: run
![](https://i.imgur.com/oJVy5b3.png)

Anther method reflection detected!! MAGIC!

# Decrypting the DEX file

As mentioned above, we know that the Roaming Mantis reads a file from the assets directory and uses Base64 to decode it. For further investigation, we find the file of the DEX payload.

![](https://i.imgur.com/8isbtIy.png)

After unzipping the "assets/db" file, we then use the Base64 to decode it and rename it Roaming_Mantis.dex.

![](https://i.imgur.com/jsyEBYd.png)

Now we have the DEX payload!.

---

## Summary Report of the DEX file

Now, let's do the summary report again for the DEX payload, we found 37 suspicious activities.

![](https://i.imgur.com/CrY5STX.png)

We simply summarize these suspicious behaviors into twelve categories.

1. Connect to the remote server
2. Start a web server
3. Monitor/Delete/Send SMS/MMS
4. Access network information
5. Access phone information
6. Access personal information
7. Record audio/video
8. Load external class
9. Access currently running applications and installed packages
10. Make a phone call
11. Open a web page
12. Install other APKs from the file

Next, we will introduce some interesting and highly suspicious activities to you based on the above categories.

## 1. Connect to the remote server
### Rule Classification: a/b;a
![](https://i.imgur.com/H2w1mJw.png)

C2 connections are common in malware. This is a clue for further C2 investigation.

## 2. Start a web server
### Rule Classification: b/g;run
![](https://i.imgur.com/4Wk6bYC.png)

Our investigation proves that the malware starts a web server and tricks users into filling credentials like username, password, etc.

## 3. Monitor/Delete/Send SMS/MMS
### Rule Classification: com/n;b
![](https://i.imgur.com/ZDz9lKw.png)
### Rule Classification: com/Loader$s;onReceive
![](https://i.imgur.com/HdtYquG.png)
### Rule Classification: com/Loader;start
![](https://i.imgur.com/bbiqb9e.png)

Our investigation proves that these operations concerning SMS might launch activities like:

1. Steal verification code for the two-factor authentication.
2. Steal verification code during online purchasing.


## 4. Access phone information
### Rule Classification: a/a;a
![](https://i.imgur.com/4q30DC6.png)
### Rule Classification: com/Loader$ag$1;a
![](https://i.imgur.com/qE24Cjr.png)

Our investigation proves that these operations concerning "access phone information" might launch activities like:

1. Query the IMEI number to targeting the Asian region.
2. Check the SIM card status just make sure the victim's phone works.


## 5. Record audio/video
### Rule Classification: com/j;a
![](https://i.imgur.com/7RCWH27.png)

Thrilling! This malware records your audio/video!


# Conclusion

This report shows how malware analysts can use the quark engine to quickly guess behaviors of malware and to quickly validate their guess through call graphs and the classification table.

In this report, we show that Quark Engine bypassed the obfuscation techniques used in Roaming Mantis. Also, this time we provide some useful rule sets for the detection. E.g. detecting payload decryption, dex loader, method reflection, SMS operation, potential c2 connection etc. And all these rules are generated by using our auto-generate tools.

We're proud of our work and we love to play around with it.

So, if you want to take a sip of the quark engine. Please visit our GitHub  repository:
* https://bit.ly/2ISYG2s

And the rules used in this report.
* https://bit.ly/3jAOkAv

You can generate rules by yourself if you can't wait for our next rule release!
* https://bit.ly/2IJNxkE
