# **Demonstration of the Hayabusa Tool for Windows Event Log Analysis**

In a previous report, I discussed using the DeepBlueCLI tool for analysing Windows Event logs for the purpose of threat hunting. While DeepBlueCLI is an excellent tool, I recently discovered Hayabusa, developed by the Yamato Security Group. According to its GitHub page, Hayabusa is a “Winsows event log fast forensics timeline generator and threat hunting tool created by the Yamato Security group”. Hayabusa supports over 4000 Sigma rules and more than 170 built-in detection rules, facilitating proactive threat hunting and digital forensics and incident response (DFIR). The tool is compatible with Windows, Linux, and macOS. 

<br>

### **Getting Started with Hayabusa**

#### Download and installation:

In this demonstration, I will be using Hayabusa on Windows to demonstrate the tool against a series of Windows event logs. To start, navigate to the releases page on their [GitHub](https://github.com/Yamato-Security/hayabusa?tab=readme-ov-file#startup):

<img width="336" height="217" alt="Image" src="https://github.com/user-attachments/assets/7e0d6763-a980-4d44-b6bd-02d9ad8cab43" />

You can then download the newest release (in my case, I chose the win-x64 version):

<img width="795" height="166" alt="Image" src="https://github.com/user-attachments/assets/42d1b6ff-747f-4af0-bcdf-0fc6a00d3d52" />

Once the zip file has downloaded, make sure to extract it. Now open a PowerShell window and navigate to the download directory:

```powershell
.\hayabusa-2.16.0-win-x64.exe
```

<img width="622" height="459" alt="Image" src="https://github.com/user-attachments/assets/c1762dd6-0df7-4a2e-8871-b1736ff5ad5a" />

<br>

### **Using Hayabusa**

To start using the tool, let’s first make sure it has all the updated rules, we can update the rules by entering:

```powershell
.\hayabusa-2.16.0-win-x64.exe update-rules
```

Let’s now use Hayabusa to create a csv timeline for a given event log:

```powershell
.\hayabusa-2.16.0-win-x64.exe -f <path_to_evtx_file> -U -o hayabusa_out.csv
```

This command uses the csv-timeline option, -f signifies the evtx file or folder containing a series of evtx files, -U makes the time zone set to UTC, and -o signifies the output file. Run the command by clicking enter:

<img width="940" height="187" alt="Image" src="https://github.com/user-attachments/assets/af15a2bd-be06-40da-9b73-d9a6a6e368e9" />

You will then be prompted asking which set of detection rules would you like to load. This is based off of the sigma rules, in these examples I am going to select all event and alert rules (option 5) and enter y for the other prompts. Once it has analysed the evtx file, it will output the results to the terminal and the csv file will be saved to the designated location. I opened the results in notepad:

<img width="940" height="101" alt="Image" src="https://github.com/user-attachments/assets/c730dec9-8ac6-4198-91f8-eaa8a9cf85b3" />

<br>

### **Improved Example**

To provide a better example, I downloaded additional sample evtx files, namely one that recorded the logs generated when using the AtomicRedTeam tool. The output produced this time is extremely informative and useful when threat hunting or performing DFIR (some columns have been removed for clarity):

<img width="940" height="265" alt="Image" src="https://github.com/user-attachments/assets/ac10471c-ac34-49b9-8a83-e1419c0c11ac" />

As you can see, the information provided by Hayabusa would be extremely helpful as we can immediately determine malicious activities are occurring on the MSEDGEWIN10 machine. 

<br>

### **HTML Results Summary**

Hayabusa can also output results in HTML format. To output the results in HTML format, we can use the -H option like as follows:

```powershell
.\hayabusa-2.16.0-win-x64.exe -f <path_to_evtx_file> -U -o hayabusa_out.csv -H report.html
```

The image below shows some of the output, if you click the hyperlinks, you are also provided with the sigma rule that matched:

<img width="683" height="509" alt="Image" src="https://github.com/user-attachments/assets/7ae0f00b-a1fe-42db-970c-24ae8bfd6465" />

<br>

### **Multiline Output**

If you want to import the results into tools like LibreOffice or Timeline Explorer, use the -M option:

```powershell
.\hayabusa-2.16.0-win-x64.exe -f <path_to_evtx_file> -U -o hayabusa_out.csv -M
```

In this case, I am going to import the output into Timeline Explorer, which is a wonderful tool made by Eric Zimmerman:

<img width="940" height="300" alt="Image" src="https://github.com/user-attachments/assets/cd2d45dd-a1b1-4db2-a400-bc166d9d32f9" />

The output is much easier to read using a tool like Timeline Explorer compared to something like Excel or notepad. 

<br>

### **Conclusion**

I tested Hayabusa with various Windows event log files, including those containing Mimikatz activity, privilege escalation, password spraying, and Metasploit. The tool proved extremely valuable for threat hunting and DFIR, providing detailed and actionable insights. 
Hayabusa is a robust tool that enhances the capability to quickly and effectively analyse Windows Event logs. Its compatibility across different platforms and extensive support for Sigma rules make it a must-have for any security professional’s toolkit. Please note that this only touches the surface of this tool, it can be integrated with other popular tools such as Velociraptor and has several other features that weren’t explored in this report.
