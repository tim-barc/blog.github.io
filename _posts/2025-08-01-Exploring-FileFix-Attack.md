# Exploring the FileFix Attack

I would first like to acknowledge that most of this information comes from an incredible blog post by mrd0x, found [here](https://mrd0x.com/filefix-clickfix-alternative/). I am not going to sit here and claim that I have come across a super novel social engineering technique that supersedes a ClickFix attack, this report is meant to serve as me exploring the FileFix attack and what forensic artifacts can be used to investigate it. 

<br>

### **Context: From ClickFix to FileFix**

ClickFix is a type of social engineering attack whereby users are prompted to execute malicious code through the Run Dialog, all under the guise of performing a captcha (see Figure 1). 


<img width="602" height="314" alt="Image" src="https://github.com/user-attachments/assets/af37be64-a132-4f08-80f6-8473ff283ca2" />

*Figure 1 ClickFix Example*

I recommend exploring this [post](https://www.proofpoint.com/au/blog/threat-insight/security-brief-clickfix-social-engineering-technique-floods-threat-landscape) by proofpoint, which explains the ClickFix attack in more detail. If you go through this post, you can determine that this technique is used by several different threat actors, and has been observed to deploy AsyncRAT, Danabot, DarkGate, Lumma Stealer, NetSupport and more. I have personally observed this technique being used to deploy Lumma Stealer on multiple occasions, being extremely effective.

Whilst ClickFix is extremely effective, persuading users to execute weird looking commands in the Run Dialog can challenging. Furthermore, detection of such an attempt is relatively simple, as most EDR tools will notice the suspicious process genealogy, for example, explorer.exe spawning child processes like cmd.exe, powershell.exe, mshta.exe, etc. This is where the FileFix attack comes into play. FileFix is another form of social engineering that involves enticing a user into pasting a malicious command, typically PowerShell, into the file explorer address bar. 

<br>

### **FileFix in Practice**

Within the post by mrd0x is some basic source code that can be used to demonstrate a FileFix attack. If you save that code as a HTML file and open it with a browser, you are presented with the following page:

<img width="602" height="574" alt="Image" src="https://github.com/user-attachments/assets/0e899b92-3356-4df2-b5aa-275befe29881" />

*Figure 2 ClickFix Demo*

In my opinion, this is a relatively professional looking page, and it could certainly trick unsuspecting users into executing malicious commands. The attack chain is simple; the user copies the “file path” and pastes that into the file explorer address bar:

<img width="481" height="82" alt="Image" src="https://github.com/user-attachments/assets/cc9cebce-e4f9-4b8b-94a4-d9a78ccf4abe" />

*Figure 3 File Explorer Address Bar*

As you can see in Figure 3, you are essentially unable to see the malicious command unless you move over to the left. The command in this case is as follows:

```powershell
Powershell.exe -c ping example.com                                                                                                                
# C:\company\internal-secure\filedrive\HRPolicy.docx
```

Everything after the `#` is treated as a comment, therefore, only the PowerShell command in this instance gets executed. In my testing, Windows Defender was able to detect this, and assigned it the following description:

<img width="292" height="65" alt="Image" src="https://github.com/user-attachments/assets/3b2fb61b-1f63-4d16-a71a-7f6388284db9" />

*Figure 4 Windows Defender FileFix Detection*

<br>

### **FileFix Artifacts**

A very simple artifact that we can use to investigate FileFix attacks is called TypedPaths. TypedPaths store what paths or commands a user has entered into the Run dialogue or File Explorer address bar. The TypedPaths key is located in the NTUSER.dat hive, and can be found at:

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

In this instance, I am using a tool called Registry Explorer by Eric Zimmerman to view the NTUSER.dat hive for my user. Upon navigating to the TypedPaths key, we can find the command that was copied into the address bar:

<img width="602" height="153" alt="Image" src="https://github.com/user-attachments/assets/41b754a0-d4e6-4970-a791-dde370984cc9" />

*Figure 5 Registry Explorer TypedPaths Key*

Aside from the TypedPaths key, you can also use a variety of artifacts such as exploring a user’s browsing history to try and find the source hosting the FileFix attack, using something like the Wayback Machine if the site is taken down or offline. 

<br>

### **Conclusion**

The FileFix attack is increasingly observed in the [wild](https://thedfirreport.com/2025/07/14/kongtuke-filefix-leads-to-new-interlock-rat-variant/) and offers threat actors a more convincing method than ClickFix. Both methods are easy to detect, and forensic artifacts such as the RunMRU and TypedPaths can be used to investigate both ClickFix and FileFix attacks respectively.
