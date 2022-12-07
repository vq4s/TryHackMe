# Day 7: Maldocs roasting on an open fire
## Learning Objectives
* What is CyberChef
* What are the capabilities of CyberChef
* How to leverage CyberChef to analyze a malicious document
* How to deobfuscate, filter and parse the data

## Using CyberChef for mal doc analysis
### 1.Add the File to CyberChef
Drag the invoice.doc file from the desktop to panel as input, as shown below. Alternatively, the user can add the Division_of_labour-Load_share_plan.doc file by Open file as input icon in the top-right area of the CyberChef page.

![1](https://user-images.githubusercontent.com/53142039/206240129-195790f2-5ffe-49f6-8d10-955c0375dd58.png)

### 2) Extract strings
Strings are ASCII and Unicode-printable sequences of characters within a file. We are interested in the strings embedded in the file that could lead us to suspicious domains. Use the **strings** function from the left panel to extract the strings by dragging it to panel and selecting All printable chars as shown below:

![2](https://user-images.githubusercontent.com/53142039/206240143-b46e822d-5ab5-42a7-9eb0-37576576aa6f.png)

If we examine the result, we can see some random strings of different lengths and some obfuscated strings. Narrow down the search to show the strings with a larger length. Keep increasing the minimum length until you remove all the noise and are only left with the meaningful string, as shown below:

![3a](https://user-images.githubusercontent.com/53142039/206244334-c9f1afcd-1eb9-4f71-8db0-6242de77f18b.png)

### 3) Remove Pattern
Attackers often add random characters to obfuscate the actual value. If we examine, we can find some repeated characters `[ _ ]`. As these characters are common in different places, we can use regex **(regular expressions)** within the `Find / Replace` function to find and remove these repeated characters. 

To use regex, we will put characters within the square brackets `[ ]` and use backslash `\` to escape characters. In this case, the final regex will be `[\[\]\n_]` where \n represents the **Line feed**, as shown below:

![3](https://user-images.githubusercontent.com/53142039/206240153-d48171f3-5664-4321-ad5a-cb11afc37e36.png)

It's evident from the result that we are dealing with a PowerShell script, and it is using base64 Encoded string to hide the actual code.

### 4) Drop Bytes
To get access to the base64 string, we need to remove the extra bytes from the top. Let's use the `Drop bytes` function and keep increasing the number until the top bytes are removed. 

![4](https://user-images.githubusercontent.com/53142039/206240160-3152f000-77f2-4f68-b07c-58a4d10b3ca7.png)

### 5) Decode base64
Now we are only left with the base64 text. We will use the `From base64` function to decode this string, as shown below:

![5](https://user-images.githubusercontent.com/53142039/206240174-aa22313b-d020-4664-bdf4-56dba57f119d.png)

### 6) Decode UTF-16
The base64 decoded result clearly indicates a PowerShell script which seems like an interesting finding. In general, the PowerShell scripts use the `Unicode UTF-16LE` encoding by default. We will be using the `Decode text` function to decode the result into UTF-16E, as shown below:

![6](https://user-images.githubusercontent.com/53142039/206240187-11b6c94d-7d21-4a14-b3c1-560ec715edd2.png)

### 7) Find and Remove Common Patterns
Forensic McBlue observes various repeated characters ``' ( ) + ' ` "`` within the output, which makes the result a bit messy. Let's use regex in the `Find/Replace` function again to remove these characters, as shown below. The final regex will be ``['()+'"`]``.

![7](https://user-images.githubusercontent.com/53142039/206240206-c024190d-ec07-4c24-bc07-e9e3c2b9113c.png)

### 8) Find and Replace
If we examine the output, we will find various domains and some weird letters `]b2H_` before each domain reference. A replace function is also found below that seems to replace this `]b2H_` with `http`. 

![8](https://user-images.githubusercontent.com/53142039/206240213-32f3f967-1148-41e3-9324-746e882b13ba.png)

Let's use the `Find / Replace` function to replace `]b2H_` with `http` as shown below:

![9](https://user-images.githubusercontent.com/53142039/206240215-d12dafba-ea6e-41d5-be72-88b9ddf90ad0.png)

### 9) Extract URLs
The result clearly shows some domains, which is what we expected to find. We will use the `Extract URLs` function to extract the URLs from the result, as shown below:

![10](https://user-images.githubusercontent.com/53142039/206240232-6419804c-160c-4a10-ad08-d20e51beea49.png)

### 10) Split URLs with @
The result shows that each domain is followed by the `@` character, which can be removed using the split function as shown below:

![11](https://user-images.githubusercontent.com/53142039/206240247-428eed59-00bd-410b-a550-a0f4c3faa5af.png)

### 11) Defang URL
Great - We have finally extracted the URLs from the malicious document; it looks like the document was indeed malicious and was downloading a malicious program from a suspicious domain. 

Before passing these domains to the SOC team for deep malware analysis, it is recommended to defang them to avoid accidental clicks. Defanging the URLs makes them unclickable. We will use `Defang URL` to do the task, as shown below:

![12](https://user-images.githubusercontent.com/53142039/206240256-f4127788-7c07-40fc-9001-9883bd39cc97.png)
from TryHackMe.

## Answers
1. What is the version of CyberChef found in the attached VM?: **`9.49.0`**
![ans1](https://user-images.githubusercontent.com/53142039/206240292-ae9a9562-c496-4a7e-a5c9-caf98fe61c75.png)
2. How many recipes were used to extract URLs from the malicious doc: **`10`** 
3. We found a URL that was downloading a suspicious file; what is the name of that malware: **`mysterygift.exe`**
4. What is the last defanged URL of the bandityeti domain found in the last step: **`hxxps[://]cdn[.]bandityeti[.]THM/files/index/`**
5. What is the ticket found in one of the domains? (Format: Domain/<GOLDEN_FLAG>): **`THM_MYSTERY_FLAG`**
