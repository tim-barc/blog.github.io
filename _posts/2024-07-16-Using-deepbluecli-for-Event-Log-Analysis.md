# **DeepBlueCLI: Enhancing Threat Hunting with Windows Event Logs**

Up until recently, I had only a basic understanding of Windows Event Logs and Sysmon logs, primarily knowing that they recorded important events that occur on a system like logins, process creation, and more. Thanks to the TryHackMe SOC Level 1 path, I was able to significantly deepen my understanding of these logs and proficiency using PowerShell cmdlets like ‘Get-WinEvent’ and the Event Viewer (GUI log reader). Through further research, I came across an awesome PowerShell script by Eric Conrad called [DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI/tree/master), designed to enhance threat hunting via Windows Event Logs.
In essence, DeepBlueCLI is a PowerShell script that analyses Windows Event Logs for threat hunting. It extracts and correlates significant events, identifying attacks such as password spraying and more. It is important to note however that it is not an intrusion detection system (IDS) as it is unable to (by default) generate alerts. The screen from the DeepBlueCLI GitHub page below details some, but not all the detectable events:

<img width="695" height="640" alt="Image" src="https://github.com/user-attachments/assets/36f07109-eff5-4bcf-8c61-24186a2d07d3" />

<br>

### **Using DeepBlueCLI**

I will be using DeepBlueCLI on a Windows 10 VM, but you can also use it on your local host since Windows Event Logs themselves are not malicious, nor is the tool. To install the script, navigate to the [GitHub repo](https://github.com/sans-blue-team/DeepBlueCLI/blob/master/READMEs/README-DeepBlueHash.md) and download the entire repository as a ZIP file. This download also includes sample event log files and the actual tool script. 

<img width="412" height="336" alt="Image" src="https://github.com/user-attachments/assets/6eefa323-8cef-4eda-be6f-52d00b49d71d" />

After downloading the ZIP file, extract it and navigate to the installation directory using PowerShell or CMD (then enter PowerShell) as Administrator:

<img width="699" height="451" alt="Image" src="https://github.com/user-attachments/assets/72ba01df-edae-4a8c-8f8a-433c4799f297" />

Luckily for us that’s all we need to do to start using the tool. Let’s now go through an example of using the DeepBlueCLI tool on a sample evtx file. The syntax to analyse a log file in its entirety is .\DeepBlue.ps1 <filename>. Please note, before we execute the script you need to change the execution policy to unrestricted like as follows:

```powershell
Set-ExecutionPolicy unrestricted
```

can now use the DeepBlueCLI tool:

```powershell
.\DeepBlue.ps1 <path_to_evtx_file> 
```

Once you have pressed enter, make sure to type ‘R’:

<img width="902" height="133" alt="Image" src="https://github.com/user-attachments/assets/95c8ee6f-494b-4892-ab5b-8129c0723838" />

After giving it some time to analyse the file, you will be presented with hopefully helpful output like seen below:

<img width="940" height="212" alt="Image" src="https://github.com/user-attachments/assets/6951dea5-d39a-41c5-8849-471fa9900d7c" />

From the output, we can see that the username ‘jwrig’ performed a password spray attack against 41 different user accounts. For context, a password spray attack is when a threat actor tries the same password across multiple usernames. The script also reveals that the same user cleared the log file. This example highlights how helpful DeepBlueCLI is for threat hunting and log analysis, saving time compared to manually exploring log files and correlating events in the Event Viewer or similar tools. 

<br>

### **Detecting Mimikatz**

Let’s use this tool against logs produced after Mimikatz was used to extract credentials. This time, I will output the results in a table format (the full list of output types can be seen in the image below):

<img width="647" height="290" alt="Image" src="https://github.com/user-attachments/assets/8a222daa-6694-4846-bdd6-15844dbde7e5" />
<img width="940" height="26" alt="Image" src="https://github.com/user-attachments/assets/7a9f0e43-fbaf-4eed-b984-37585dd49ef8" />
<img width="940" height="65" alt="Image" src="https://github.com/user-attachments/assets/cde0fa96-d90c-46ab-b466-34bc03309888" />

As seen in the output above, DeepBlueCLI has detected the use of Minikatz which is a popular password dumping tool used by threat actors. 

<br>

### **Decoding Obfuscated/Encoded Commands**

Threat actors often use encoding tactics to obfuscate their attacks, bypassing basic signature based detection. DeepBlueCLI can detect such anomalies, like unusually long sequences of Base64 characters or binary encoding patterns, which are uncommon in legitimate commands. For example, let’s analyse the Powershell-Invoke-Obfuscation-many.evtx file:

<img width="940" height="176" alt="Image" src="https://github.com/user-attachments/assets/d95527c7-76b0-489c-8301-aa09287077c8" />

As seen in the above image, the script was able to identify that there is 500+ consecutive Base64 characters, and it has alerted it as suspicious. You could then simply echo the base64 encoded text and pipe it to base64 decode to see the decoded command. 

<img width="940" height="325" alt="Image" src="https://github.com/user-attachments/assets/5596b5aa-69f8-416c-8075-a08398290e66" />

As hinted earlier, you can see that it also detects binary encoding based off of the large amount of 1s and 0s, something that is not typically seen in non-malicious commands. 

<br>

### **DeepBlueHash**

DeepBlueHash is a separate tool from DeepBlueCLI that parses Sysmon event logs to extract SHA256 hashes from process creation, driver load, and image load events. I will not be going over this script today, but please refer to the following [link](https://github.com/sans-blue-team/DeepBlueCLI/blob/master/READMEs/README-DeepBlueHash.md) if you wish to explore it for yourself (note, you will require a free VirusTotal API key).

<br>

### **Conclusion**

This post covers only a snippet of DeepBlueCLI’s potential. It can also detect frameworks like Metasploit and much more. For more resources on this great tool check out the following:
- https://github.com/strandjs/IntroLabs/blob/master/IntroClassFiles/Tools/IntroClass/deepbluecli/DeepBlueCLI.md
- https://github.com/sans-blue-team/DeepBlueCLI
- https://youtu.be/6sMluvfLsI8?si=aFnkEGmQIcoZ_e_P
I hope you enjoyed this post. DeepBlueCLI is an incredibly fun tool to use and I highly recommend it for those who struggle with analysing event logs using the Event Viewer or PowerShell cmdlets. 
