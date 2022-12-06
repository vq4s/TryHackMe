# Day 6: It's beginning to look a lot like phishing 
## Learning Objectives
  * Learn what email analysis is and why it still matters.
  * Learn the email header sections.
  * Learn the essential questions to ask in email analysis.
  * Learn how to use email header sections to evaluate an email.
  * Learn to use additional tools to discover email attachments and conduct further analysis.

## What is Email Analysis?

Email analysis is the process of extracting the email header information to expose the email file details. The email header contains the technical details of the email like sender, recipient, path, return address and attachments. Usually, these details are enough to determine if there is something suspicious/abnormal in the email and decide on further actions on the email, like filtering/quarantining or delivering. This process can be done manually and with the help of tools.

## How to Analyse Emails?
| Field  | Details |
| ------------- | ------------- |
| From  | The sender's address.|
| To  | Timestamp, when the email was sent.|
| Date |Timestamp, when the email was sent.|
| Subject |Timestamp, when the email was sent.|
|Return Path  |The return address of the reply, a.k.a. "Reply-To".  If you reply to an email, the reply will go to the address mentioned in this field.|
|Domain Key and DKIM Signatures|Email signatures are provided by email services to identify and authenticate emails.|
|SPF |Shows the server that was used to send the email.  It will help to understand if the actual server is used to send the email from a specific domain.|
|Message-ID|Unique ID of the email.|
|MIME-Version|Used MIME version.  It will help to understand the delivered "non-text" contents and attachments.|
|X-Headers|The receiver mail providers usually add these fields.  Provided info is usually experimental and can be different according to the mail provider.|
|X-Received|Mail servers that the email went through.|
|X-Spam Status|Spam score of the email.|
|X-Mailer|Email client name.|

## Important Email Header Fields for Quick Analysis
Analysing multiple header fields can be confusing at first glance, but starting from the key points will make the analysis process slightly easier. A simple process of email analysis is shown below.
| Questions to Ask / Required Checks   | Evaluation|
| ------------- | ------------- |
| Do the "From", "To", and "CC" fields contain valid addresses?  | Having invalid addresses is a red flag.|
|Do the "From" and "To" fields are the same?|Having the same sender and recipient is a red flag.|
|Do the "From" and "Return-Path" fields are the same?|Having different values in these sections is a red flag.|
|Was the email sent from the correct server?|Email should have come from the official mail servers of the sender.|
|Does the "Message-ID" field exist, and is it valid?|Empty and malformed values are red flags.|
|Do the hyperlinks redirect to suspicious/abnormal sites?|Suspicious links and redirections are red flags.|
|Do the attachments consist of or contain malware?|Suspicious attachments are a red flag.  File hashes marked as suspicious/malicious by sandboxes are a red flag|

## emlAnalyzer
The emlAnalyzer is a tool designed to parse email headers for a better view and analysis process.

|Query Details|Explanation|
|----------|--------------|
|emlAnalyzer|Main command|
|-i |File to analyse|
|--header|Show header|
|-u |Show URLs|
|--text|Show cleartext data|
|--extract-all|Extract all attachments|

## Other tools
Here, if you find any suspicious URLs and IP addresses, consider using some OSINT tools for further investigation.
|Tool|Purpose|
|--------|----|
|VirusTotal|A service that provides a cloud-based detection toolset and sandbox environment.|
|InQuest|A service provides network and file analysis by using threat analytics.|
|IPinfo.io|A service that provides detailed information about an IP address by focusing on geolocation data and service provider.|
|Talos Reputation|An IP reputation check service is provided by Cisco Talos.|
|Urlscan.io|A service that analyses websites by simulating regular user behaviour.|
|Browserling|A browser sandbox is used to test suspicious/malicious links.|
|Wannabrowser|A browser sandbox is used to test suspicious/malicious links.|

by TryHackMe.

## Answers
1.What is the email address of the sender: **chief.elf@santaclaus.thm** \
2.What is the return address: **murphy.evident@bandityeti.thm** \
3.On whose behalf was the email sent: **Chief Elf**  \
4.What is the X-spam score: **3** \
![image](https://user-images.githubusercontent.com/53142039/205989971-18a12460-0133-4464-90e6-b4ca9003849d.png)\
5.What is hidden in the value of the Message-ID field(base64): **AoC2022_Email_Analysis** \
6.What is the reputation result of the sender's email address: **RISKY** \
![image](https://user-images.githubusercontent.com/53142039/205993613-1cb8477c-a764-4714-8530-c5bce73b805e.png)
7.What is the filename of the attachment: **Division_of_labour-Load_share_plan.doc** \
8.What is the hash value of the attachment: **0827bb9a2e7c0628b82256759f0f888ca1abd6a2d903acdb8e44aca6a1a03467** \
![image](https://user-images.githubusercontent.com/53142039/205985139-f9823875-1b57-4d8d-8df5-42f22fa33553.png) \
9.What is the second tactic marked in the Mitre ATT&CK section: **Defense Evasion** \
![image](https://user-images.githubusercontent.com/53142039/205993630-ff63cbed-4397-4b3b-b1aa-9beebf94674a.png)
10.What is the subcategory of the file: **macro_hunter** \
![image](https://user-images.githubusercontent.com/53142039/205993625-3da8842c-5012-4e1b-906c-15e911128c43.png)
