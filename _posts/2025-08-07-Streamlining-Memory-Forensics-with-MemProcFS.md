# Streamlining Memory Forensics with MemProcFS

MemProcFS, or **Mem**ory **Proc**ess **F**ile **S**ystem, is a tool that enables you to view memory as files in a virtual filesystem. You can essentially think of MemProcFS as a tool that runs a bunch of Volatility plugins, and outputs everything in a virtual file system, or as Volatility’s more usable counterpart. When most people think of memory forensics, tools like Volatility and Rekall come to mind. Whilst Volatility is incredible, it does lack the ease of use that MemProcFS provides. In my opinion, MemProcFS is a great memory forensics tool for both beginners and experienced practitioners and enables quick triaging of memory images through the “kitchen sink approach”, whilst Volatility enables you to do more targeted analysis. Not only is MemProcFS heavily documented, but it is also so easy to use that it honestly doesn’t require you to do much searching/research beforehand. If you were to process a memory dump using MemProcFS for the first time, with no prior knowledge of the tool, you would still be able to find key information in the output. This is in stark contrast to Volatility, as it requires you to understand the syntax, what plugins you should use, how those plugins work, etc. 

From my little research, MemProcFS lacks the popularly and respect that it deserves. As Richard Davis (13Cubed) puts it in his YouTube video title, “MemProcFS – This Changes Everything”, highlighting how even seasoned practitioners who previously relied on Volatility understand the benefit and value MemProcFS provides. This post is not meant to serve as a deep dive into the tool, if that’s what you came for, make sure to consult the documentation or other blog posts. It is meant to serve as a surface level exploration of the tool, showing off its key benefits. 

### **Installation**

MemProcFS installation is straightforward:

- Navigate to the Releases section on the MemProcFS [GitHub](https://github.com/ufrisk/MemProcFS):  
  <img width="940" height="536" alt="Image" src="https://github.com/user-attachments/assets/614ced22-2283-4caf-8190-117542c690e3" />

- Download and extract the compressed file suited for your operating system, in my case its Windows:  
  <img width="940" height="535" alt="Image" src="https://github.com/user-attachments/assets/bcf5743b-e946-418e-a29c-1ca1dd88f6c4" />

- Your MemProcFS binary is now ready for use.

<br>

### **Getting Started**

To execute MemProcFS and see the help menu, you can enter the following command:

```powershell
.\MemProcFS.exe
```

<img width="940" height="447" alt="Image" src="https://github.com/user-attachments/assets/9e40007a-7a7a-4ed3-ba39-22487b5130bb" />

Processing a memory dump with MemProcFS is simple, the most common method is by executing the following command:

```powershell
.\MemProcFS.exe -f "<path_to_memory_dump>" -forensic 1
```

This command mounts the virtual filesystem output to the drive letter M. Within this drive, you can find a bunch of important forensic information which I will be exploring shortly. 

<img width="309" height="152" alt="Image" src="https://github.com/user-attachments/assets/a1e0e956-793a-4278-af0e-f458bbdb9440" />

The forensic 1 option performs a bunch of analysis tasks and outputs the result into an SQLite database and in the forensic sub-directory. These analysis tasks include things like MFT scanning, Timeline analysis for processes, registry, MFT, etc, and much more. If you are familiar with KAPE, an incredible tool made by Eric Zimmerman, you can think of the forensic option as modules, which run programs/plugins against artifacts to generate a more digestible output. It is important to note that MemProcFS can take some time to process the memory dump, so be patient. Within the forensic folder, you can find a text file titled progress_percent.txt, this shows you the progress, you can also see when it finishes in the terminal output. 

<br> 

### **Getting Started**

The following explores the key information you can quickly access in the mounted drive. Please consult to the [documentation](https://github.com/ufrisk/MemProcFS/wiki) to understand its full capabilities. Another incredible feature of MemProcFS, is that each forensic folder contains a readme file that details what the folder contains (i.e., what the plugin did).

### **Forensic Output**

Key forensic information provided by the forensic option output include:

- **Recovered Files (M:\forensic\files)**: The `files` module attempts to recover files from file handles in the system, displaying them in a directory structure. Therefore, the `files` folder contains a best-effort reconstructed file system.  
  <img width="566" height="174" alt="Image" src="https://github.com/user-attachments/assets/47ce0437-7b6f-411f-937b-c14c5ea8290b" />

- **Malware Indicators (M:\forensic\findevil)**: The `Find Evil` module attempts to identify signs of malware infection through things like Windows Defender detections, RWX VADs, suspicious process genealogy, and more. Within this folder, you can find a file titled `findevil.txt`, which contains a bunch of useful information that can come in handy when trying to identify malware.  
  <img width="940" height="183" alt="Image" src="https://github.com/user-attachments/assets/055e1f8a-929e-438e-b551-568460cd1359" />

  Whilst this will show a bunch of false positives, in the above example, we can see three **Windows Defender detections**, which have correctly identified malware.

- **NTFS Artifacts (M:\forensic\ntfs)**: The `ntfs` folder contains artifacts related to the NTFS file system. The key artifact within this folder is the `ntfs_files.txt` file, which is the MFT (Master File Table) as extracted from memory.  
  <img width="940" height="360" alt="Image" src="https://github.com/user-attachments/assets/45dd04a8-370d-4f2a-a019-71b50853be6c" />

- **Timeline Analysis (M:\forensic\timeline)**: The `timeline` folder includes a variety of forensic timelines, sorted chronologically.  
  <img width="347" height="467" alt="Image" src="https://github.com/user-attachments/assets/ab80bc1e-2e8c-4b7c-82cd-5514fba743c1" />

- **Web Activity (M:\forensic\web)**: Within the `web` folder, you can find a user’s browsing activity. It currently supports Chrome, Edge, and Firefox.  
  <img width="940" height="100" alt="Image" src="https://github.com/user-attachments/assets/0086214e-f00f-49e5-b8a4-a96ad902e81e" />

<br>

### **Process Information**

Process information includes:

- **Process Listings (M:\sys\proc)**: The `proc` folder contains three files:  
    - `proc.txt`: Lists processes and their parent processes in a tree view.  
    - `proc-v.txt`: Lists processes and their parent processes along with process image path and command line.  
    - `proc-time.txt`: Process list sorted by creation time.  
  <img width="180" height="147" alt="Image" src="https://github.com/user-attachments/assets/3b9c1ac3-f762-4b5e-8337-f073ac7aa6e3" />

  Both `proc.txt` and `proc-v.txt` can be thought of as the output of the `pstree` plugin within Volatility. Here you can find the processes captured from the memory dump in hierarchical order:  
  <img width="940" height="621" alt="Image" src="https://github.com/user-attachments/assets/de9885c6-ac56-4d45-a274-8ccadd7b03df" />

  This is a great way to see what was running on the system, and you can often identify malicious activity through anomalous process genealogy (like `explorer.exe` spawning `svchost.exe`) and so forth.

- **Process-specific data (M:\name or M:\pid)**: The `name` folder contains all the processes ordered by their image name.  
  <img width="114" height="188" alt="Image" src="https://github.com/user-attachments/assets/2e090a84-ff71-43cf-81c0-17342d7c51f1" />

  If you click on any of the folders, you can find a lot of information about the process:  
  <img width="72" height="291" alt="Image" src="https://github.com/user-attachments/assets/64fe2f89-c1fa-4c9c-b02c-77ad018b508c" />

  Including handles, which show what objects (like files, registry keys, processes, etc.) the process was interacting with:  
  <img width="429" height="339" alt="Image" src="https://github.com/user-attachments/assets/fe642f64-c127-4e73-bd62-a93c452500d7" />

  Modules, which show the DLLs used by the program and the process executable itself:  
  <img width="259" height="311" alt="Image" src="https://github.com/user-attachments/assets/fb38e565-7c15-49d6-8972-b956a7a37fab" />

  Memmap, which shows the virtual address descriptors (VADs) and page table. This enables you to detect things like process injection—especially if you see **READWRITE EXECUTE** permissions on a page that has an MZ file header (PE file)—and much more.

  It is important to note that the `pid` folder contains the same information. The only difference is that each folder is named after the PID of the process, rather than the image name followed by the PID.

  The `name` and `pid` folders are useful when drilling down into a specific process.


<br>

### **Network Connections**

Network connections and DNS information includes:

- **Network Connections (M:\sys\net)**: The net folder contains multiple files that show network connections, akin to Volatility’s `netscan` plugin.  
  <img width="940" height="258" alt="Image" src="https://github.com/user-attachments/assets/d8efac14-5703-4e2d-aa10-2a6bfcd4e08e" />

- **DNS Entries (M:\sys\net\dns)**: The dns folder contains a `.txt` file of extracted cached DNS entries.  
  <img width="940" height="241" alt="Image" src="https://github.com/user-attachments/assets/20eb8f01-84f0-4c58-87c9-e656cb71927a" />

<br>

### **Event Logs**

- **Evtx files (M:\misc\eventlog)**: The eventlog directory contains event log files extracted from memory. The event log files can also be found in other locations, however, all event logs extracted from memory are saved to this single location for ease of access.  
  <img width="783" height="528" alt="Image" src="https://github.com/user-attachments/assets/0082228a-5a44-4285-bfa1-441961d4241f" />  

  You can use tools like EvtxECmd to parse these event logs and view the output in Timeline Explorer. Alternatively, you can just use Event Viewer or similar tools.


<br>

### **Registry**

Registry artifacts include (but not limited to):

- **Registry Hive Files (M:\registry\hive_files)**: Hive files reconstructed from memory with a best effort-algorithm. These hive files may be partially corrupted, but you can still explore them using tools like Registry Explorer. 

- **Registry By-Hive (M:\registry\by-hive)**: Each hive contains a ROOT key for the normal registry and an ORPHAN key which may contain registry keys without know parent keys.

- **Registry HKLM (M:\registry\HKLM)**: This folder shows the registry keys stored within HKEY_LOCAL_MACHINE. 

<br>

### **Persistence Checks**

You can hunt for basic persistence mechanisms by checking the following locations:

- **Services (M:\sys\services)**: The services folder contains information about services extracted from the service control manager (SCM).  
  <img width="940" height="151" alt="Image" src="https://github.com/user-attachments/assets/aad5f307-3291-47dc-9d87-f02ff8d34256" />

- **Scheduled Tasks (M:\sys\tasks)**: The tasks folder contains information about scheduled tasks extracted from the registry.  
  <img width="940" height="131" alt="Image" src="https://github.com/user-attachments/assets/6c1e128b-cac8-4d9b-85dd-d57e0ae35fd9" />

<br>

### **System Information**

Comprehensive system information such as hostname, timezone, and system version can be found in several text files within the M:\sys folder:

<img width="284" height="434" alt="Image" src="https://github.com/user-attachments/assets/b8152525-73b2-416a-944b-bad93bd7eb3e" />

<br>

### **Conclusion**

MemProcFS significantly streamlines memory forensics, providing intuitive navigation and actionable insights without extensive prior knowledge. While Volatility remains invaluable for deep, targeted analysis, MemProcFS excels in rapid triaging and employing the kitchen sink approach to memory forensics, positioning it as an essential tool in any forensic analyst’s toolkit. 



