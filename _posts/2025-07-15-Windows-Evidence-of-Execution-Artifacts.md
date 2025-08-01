# Windows Evidence of Execution Artifacts 

Evidence of execution artifacts are forensic indicators that a program was run on a system. They are essential for incident responders and digital forensics investigators, offering insights into what was executed and when. 
This report explores the following artifacts:
- Prefetch
- Shimcache/AppCompatCache
- AmCache
- Program Compatibility Assistant (PCA)
- MUICache
- UserAssist 
 SRUM
For a complete list of available evidence of execution artifacts, check out this post by Adam Harrison. 

### Prefetch
#### Location: %SystemRoot$\Prefetch

Windows Prefetch files were introduced in Windows XP; they were designed to speed up the application startup process by preloading a snippet of code in commonly used programs. Prefetch files contain:
- Name of the executable
- A list of DLLs used by that executable
- Count of how many times the executable was run
- Timestamp indicating the last 8 times the program was executed (Windows 8+). 

#### Parsing Tool: PECmd

To parse prefetch files, we can use a tool called Prefetch Explorer Command Line (PECmd), a fantastic tool created by Eric Zimmerman. To parse an entire directory, you can use the following syntax:

```powershell
PECmd.exe -d "C:\Windows\Prefetch" --csv . --csvf prefetch_out.csv
```

Where -d ensures it recursively parses each prefetch file within the given directory, --csv . specifies to save the csv file to the current directory and --csvf specifies the filename to save the CSV formatted results to. 
To parse a single prefetch file, you can use the following syntax:

```powershell
PECmd.exe -f "C:\Windows\Prefetch\<prefetch_file>" --csv . --csvf prefetch_out.csv
```

The syntax is the same, except for the -f switch with specifies the prefetch file you want to parse. 

To analyse the output, I recommend using a tool called Timeline Explorer. This is another one of Eric Zimmerman’s tools that is more suited for forensics than Excel when it comes to CSV files. 

<img width="602" height="37" alt="Image" src="https://github.com/user-attachments/assets/ca58a8bf-c96c-4e95-8c1a-dbedd7253db5" />
<img width="602" height="38" alt="Image" src="https://github.com/user-attachments/assets/bce5054b-36db-4c08-8ac0-5a28ae95aadb" />

#### Limitations:

- Limited to 1024 files on Windows 8+ systems
- Will not identify the user that executed the application
- Requires tools to interpret data
- Can be deleted by a threat actor

#### Resources:
- https://forensics.wiki/prefetch/
- https://youtu.be/f4RAtR_3zcs?si=XgOMKKvivT48IQeI
- https://www.thedfirspot.com/post/artifacts-of-execution-i-know-what-you-did-last-incident
- https://isc.sans.edu/diary/29168

<br>

### Shimcache/AppCompatCache
#### Location: SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache

The purpose of ShimCache, also known as AppCompatCache, is to provide compatibility for old applications. If there is a compatibility issue, ShimCache will attempt to shim the application, modifying the file’s properties to try and make it run on the current system. It logs:
- Executable file name
- File path
- Last modification date and time. 

#### Parsing Tool: AppCompatCacheParser

To parse the ShimCache, you can use a tool called AppCompatCacheParser, another one of Eric Zimmerman’s tools. The syntax is as follows:

```powershell
AppCompatCacheParser.exe -f “<software_hive>” --csv . --csvf shimcache_out.csv
```

Where -f specifies the path to the clean SOFTWARE hive, --csv specifies the output directory, and --csvf specifies the output filename. You can analyse the output using Timeline Explorer:

<img width="602" height="141" alt="Image" src="https://github.com/user-attachments/assets/ab23bf02-61ec-4b28-a5e2-d569c946520e" />

#### Limitations:

- For Windows 10+ systems, Shimcache cannot be used to prove program execution. 
- Timestamps are not always accurate, which can cause the timeline timestamps to be out of order. 

#### Resources:

- https://forensafe.com/blogs/shimcache.html
- https://www.thedfirspot.com/post/evidence-of-program-existence-shimcache
- https://nullsec.us/windows-10-11-appcompatcache-deep-dive/
- https://github.com/WithSecureLabs/chainsaw/wiki/Shimcache-Analysis
- https://youtu.be/7byz1dR_CLg?si=4JYrI_DQ9YdVkLM1

<br>

### AmCache
#### Location: %SystemRoot%\appcompat\Programs\Amcache.hve

The AmCache stores metadata about program installation and execution on Windows 7+ systems for Windows Application Compatibility. Like the ShimCache, the AmCache can be used to prove that a file existed on a system but cannot reliably prove execution of a program. Key fields include:
- Full file path
- Last modified time
- Publisher information
- File size
- SHA1 hash
- Compilation time (sometimes), and more.

#### Parsing Tool: AmcacheParser

We can use a tool called AmcacheParser to parse the AmCache hive:

```powershell
AmcacheParser.exe -f "Amcache.hve" --csv . --csvf amcache_out.csv  
```
Where -f specifies the path to the clean Amcache hive, --csv specifies the output directory, and --csvf specifies the output filename. This outputs multiple CSV files:

<img width="247" height="165" alt="Image" src="https://github.com/user-attachments/assets/f331e4c6-a77a-44cb-9088-7dc376931f1e" />

You can view these files in Timeline Explorer:

<img width="602" height="101" alt="Image" src="https://github.com/user-attachments/assets/64ba8fb5-801f-4f0d-965d-2352a3c16f65" />
<img width="602" height="65" alt="Image" src="https://github.com/user-attachments/assets/1e230ec8-2195-4d97-acd4-b050d2d9d56b" />
<img width="602" height="83" alt="Image" src="https://github.com/user-attachments/assets/b870eae7-2b35-4462-a167-c9df0cadafe9" />

#### Limitations:

- Should not be used for proof of execution, rather, it should be used to prove the existence of an executable. 
- Requires tools to interpret data.
- Entries within the AmCache can be updated by automated tasks and scanning conducted by the OS, therefore, it isn’t reliable for proving execution of a program.  


#### Resources:

- https://forensics.wiki/amcache/
- https://artifacts-kb.readthedocs.io/en/latest/sources/windows/AMCache.html
- https://www.thedfirspot.com/post/evidence-of-program-existence-amcache


<br>

### Program Compatibility Assistant (PCA)
#### Location: %SystemRoot%\appcompat\pca

PCA (Program Compatibility Assistant) is a newly discovered evidence of execution artifact for Windows 11 Pro systems. Within the given path are three files: 
- PcaAppLaunchDic.txt
- PcaGeneralDb0.txt
- PcaGeneralDb1.txt.
PcaAppLaunchDic.txt contains a file path and timestamp pair that details the last execution of a program:

<img width="602" height="104" alt="Image" src="https://github.com/user-attachments/assets/2eb3ad1d-bbd4-4ee1-a2b2-1904b7f8d3aa" />

PcaGeneralDb0.txt provides information including:
- Runtime
- Run status
- Executable path
- Description of the file
- Software vendor
- File version, and more

<img width="602" height="69" alt="Image" src="https://github.com/user-attachments/assets/cc9a87be-da6c-48fd-a4d3-0ff6bf9e4ba0" />

You can use the following [tool](https://github.com/AndrewRathbun/PCAParser) to parse the PCA.

#### Resources:

- https://aboutdfir.com/new-windows-11-pro-22h2-evidence-of-execution-artifact/
- https://www.sygnia.co/blog/new-windows-11-pca-artifact/

<br>

### MUICache
#### Location: USRCLASS.DAT\Local Settings\Software\Microsoft\Shell\MuiCache

MUI (Multilingual User Interface) enables the Windows OS to have a single application localised for multiple languages. Developers create a .MUI file for each language supported by the application, enabling users to switch the language. The MUI files generate a MUICache key in the registry, which contains information about the files that are executed. Programs executed via Explorer result in MUICache entries being created. 

#### Tools: Registry Explorer

You can easily view this artifact using a tool like Registry Explorer:

<img width="602" height="198" alt="Image" src="https://github.com/user-attachments/assets/b648ce80-d1fd-4e46-8b07-b336ee158136" />

#### Limitations:

- Does not pinpoint the precise time a program was executed. It can only indicate that a program was launched at some point. 
- MUICache entries can be modified or deleted.


#### Resources:

- https://www.youtube.com/watch?v=ea2nvxN878s&t=104s
- https://www.magnetforensics.com/blog/forensic-analysis-of-muicache-files-in-windows/
- https://www.forensafe.com/blogs/muicache.html

<br>

### UserAssist 
#### Location: NTUSER.dat\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist

The UserAssist artifact displays a table of GUI programs executed on a Windows machine. The artifact stores various information about every GUI application that is executed, including:
- Program name
- Run count
- Focus count (number of times the program was set in focus, either by switching to it from other applications, or by making it active in the foreground)
- Focus time (total time the program was in focus)
- Last execution time. 


#### Tools: Registry Explorer

Within the UserAssist key are several subkeys, the ones of interest are:
- {CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}: Executed EXE files.
- {F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}: Executed LNK files.
Each subkey contains a Count subkey, which is where the information regarding executed programs is stored. You can use a tool like Registry Explorer to view UserAssist. 

<img width="562" height="147" alt="Image" src="https://github.com/user-attachments/assets/e7744c14-0fac-4ff2-ba5b-cb21b08e9ea6" />

#### Limitations:

- Inconsistent data. 

#### Resources:

- https://blog.didierstevens.com/programs/userassist/
- https://securelist.com/userassist-artifact-forensic-value-for-incident-response/116911/


<br>

### SRUM
#### Location: %SystemRoot%\System32\sru\SRUDB.dat

SRUM (System Resource Utilisation Monitor) is a feature of Windows 8+ systems that tracks data including:
- Application usage
- Network utilisation
- System energy. 
SRUM Network Usage can be extremely helpful when identifying data exfiltration, as it records bandwidth usage in bytes sent and received by an application. 

#### Parsing Tool:

We can use a tool called SrumECmd to parse the SRUM:

```powershell
SrumECmd.exe -f "SRUDB.dat" -r "<software_hive>" --csv .
```

This results in multiple CSV files being created:

<img width="370" height="198" alt="Image" src="https://github.com/user-attachments/assets/7eef1bbc-ab82-4c81-a656-33009abd02d9" />

You can then view the output in timeline explorer. For example, let’s look at the network usages output:

<img width="602" height="226" alt="Image" src="https://github.com/user-attachments/assets/ce455ee0-ffba-4789-aaab-8aa9042bc12f" />
<img width="602" height="157" alt="Image" src="https://github.com/user-attachments/assets/cb589dcc-7071-4ad7-bf64-fe9620c0eb89" />

#### Resources:

- https://www.magnetforensics.com/blog/srum-forensic-analysis-of-windows-system-resource-utilization-monitor/
- https://youtu.be/Uw8n4_o-ETM?si=tPfuJhKphzDoeVRj
- https://isc.sans.edu/diary/21927
