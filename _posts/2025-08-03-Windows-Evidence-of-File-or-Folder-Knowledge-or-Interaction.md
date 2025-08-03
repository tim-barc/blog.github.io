# Windows Evidence of File/Folder Knowledge/Interaction

Evidence of file or folder knowledge/interaction refers to artifacts that demonstrate a user was aware of, accessed, or interacted with a file or directory. This report explores the following artifacts:
- [RecentDocs](#recentdocs)
- [Office Recent Files](#office-mru)
- [ShellBags](#shellbags)
- [Comdlg32](#comdlg32)
- [TypedPaths](#typedpaths)
- [WordWheelQuery](#wordwheelquery)
- [LNK Files](#lnk-files)
- [Jump Lists](#jumplists)
- [Activities Cache Database](#activities-cache-database-activitescachedb)
- [ThumbCache](#thumbcache)

### **RecentDocs**
#### Location: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs

RecentDocs is a registry key that contains information about recently accessed files or folders and is used to populate data in recent menus of the Start Menu (Windows XP) or the Recent Items folder in the User directory (Windows 10). The RecentDocs key contains:
- File extension
- File Name
- LNK Name
- MRU Position
- Opened On timestamp, and
- Extension Last Opened timestamp 
For context, RecentDocs tracks the last 150 files or folders opened, with the MRU list keeping track of the order in which each file/folder was opened. 

#### Tool: Registry Explorer

After loading the NTUSER.dat hive and navigating to the registry key, you will see something like as follows:

<img width="940" height="248" alt="Image" src="https://github.com/user-attachments/assets/5b3185d9-0cbf-486d-ac69-0843d229ecf8" />

#### Limitations:

- Doesn’t track all file types equally. Some file types might be tracked more reliably than others. 
- Data will not be recorded if a user opts out of Windows tracking recently accessed docs. 

#### Resources:

- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://www.magnetforensics.com/blog/what-is-mru-most-recently-used/
- https://www.forensafe.com/blogs/recentdocs.html
- https://forensic4cast.com/2019/03/the-recentdocs-key-in-windows-10/

<br> 

### **Office MRU**
#### Location: NTUSER.DAT\Software\Microsoft\Office\Version
NTUSER.DAT\Software\Microsoft\Office\VERSION\UserMRU\Live_ID_####\File MRU
NTUSER.DAT\Software\Microsoft\Office\VERSION\UserMRU\Live_ID_####\Place MRU

Microsoft Office programs use this key to track recently opened files, making it easier for users to remember the last file they were editing. The RecentDocs key enables a forensic analyst to show what kind of documents were accessed by a threat actor. The File MRU key contains:
- Last opened timestamp
- Last closed timestamp
- Filename

#### Tool: Registry Explorer

After loading the NTUSER.dat hive and navigating to the registry key for FileMRU, you will see something like as follows:

<img width="940" height="163" alt="Image" src="https://github.com/user-attachments/assets/d4e81ffb-f25d-465a-bb43-9d292179cd38" />

The File MRU key stores recently opened files. On the other hand, the Place MRU key stores recently opened folders:

<img width="940" height="96" alt="Image" src="https://github.com/user-attachments/assets/8367f1d7-7aa3-43d7-bc38-75765f5a89b6" />

#### Resources:
- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://www.cybertriage.com/artifact/office-mru-registry/

<br>

### **Shellbags**
#### Location: Accessed via File Explorer:
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagsMRU
#### Location: Accessed via Desktop:
NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags
NTUSER.DAT\Software\Microsoft\Windows\Shell\BagsMRU

Shellbags show which folders were accessed on the local machine, network and/or removable devices. Shellbags are valuable for forensic analysts because they record a user’s interaction with folders, even after the folders are deleted. Shellbags contain crucial information including:
- Shell Type (either File or Directory)
- MRU Position
- Folder/file name
- Created on, Modified on, Accessed On, and First Interacted timestamps

#### Parsing Tool: ShellBags Explorer

ShellBags Explorer, like many Eric Zimmerman tools, allows you to load the live registry or hives from disk. In my case, I loaded the live registry:

<img width="940" height="114" alt="Image" src="https://github.com/user-attachments/assets/9d74cff5-a8b0-416e-a712-d78f265def4e" />

#### Limitations:
- Shellbags don’t record files within folders

#### Resources:
- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://medium.com/ce-digital-forensics/shellbag-analysis-18c9b2e87ac7
- https://youtu.be/M1nyMIu1Y18?si=-BaKtmj2zDrCD-1W
- https://youtu.be/YvVemshnpKQ?si=Za1A6y_eIBC0Nf8q
- https://www.forensafe.com/blogs/shellbags.html

<br>

### **Comdlg32**

#### Location: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePidlMRU
OpenSavePidIMRU tracks files that have been opened or saved within a Windows shell dialog box. When you open or save a file, a dialogue box appears asking us where to save or open that file from. Once you open/save a file at a specific location, Windows remembers that location. This key is also responsible for tracking auto-complete terms for the same dialog box. As you can imagine, this is a big data set as it tracks files that have been opened or saved using applications like web browsers, text editors, etc. The OpenSavePidlMRU key contains:
- File Extension
- MRU Position
- Absolute path (includes the filename)
- Opened on timestamp

#### Location: NTUSER.dat: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedMRU

LastVisitedMRU tracks the executable used by an application to open the files documented in the OpenSavePidIMRU key. Each value also tracks the directory location for the last file that was accessed by the application. The LastVisitedMRU key contains:
- MRU Position
- Executable used to open the files documented in the OpenSavePidIMRU key
- Absolute Path
- Opened on timestamp

#### Tool: Registry Explorer

Let’s start by checking out the OpenSavePidlMRU key:

<img width="940" height="167" alt="Image" src="https://github.com/user-attachments/assets/be1916a0-ef67-4a63-8b4e-331314a2b43f" />

The LastVisitedMRU key contains similar information:

<img width="940" height="114" alt="Image" src="https://github.com/user-attachments/assets/39010fc0-2d5b-4c4e-b177-adb294c37315" />

You can interpret the above image as follows; thumbcache_viewer was used to open the AppData\Local\Microsoft\Windows\Explorer folder on August 2nd, 2025, at 08:11:49. 


#### Limitations:
- The Opened On timestamp does not always indicate the last time the folder or file was accessed by the application, just the last time it was accessed though the common dialog (open/save). For example, if you drag and drop a file to Notepad, it will not be recorded. 

#### Resources:
- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://www.cybertriage.com/blog/ntuser-dat-forensics-analysis-2025/
- https://www.forensafe.com/blogs/opensavemru.html
- https://www.forensafe.com/blogs/lastvisitedmru.html

<br>

### **TypedPaths**
#### Location: NTUSER.dat: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths

The TypedPaths key stores the last 26 paths typed into the File Explorer address bar, where Url1 represents the most recently typed path. 

#### Tool: Registry Explorer

Upon navigating to the registry key, you will see something like as follows:

<img width="940" height="97" alt="Image" src="https://github.com/user-attachments/assets/6eaba591-d139-4be6-8c12-b85812368e19" />

#### Limitations:
- Only paths that exist (at the time it was typed into the address bar) are recorded in this key. 
- Data will not be recorded if the user opts out of Windows tracking app launches. 

#### Resources:
- https://www.4n6post.com/2023/02/registry-typedpath.html
- https://www.cybertriage.com/blog/ntuser-dat-forensics-analysis-2025/
- https://forensafe.com/blogs/typedpaths.html

<br>

### **WordWheelQuery**
#### Location: NTUSER.dat: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery

The WordWheelQuery key stores a list of keywords that users have searched for in the Explorer search dialog. 

#### Tool: Registry Explorer

Upon navigating to the WordWheelQuery key, you will be presented with something like as follows:

<img width="940" height="89" alt="Image" src="https://github.com/user-attachments/assets/c997f4b5-8984-4f01-a603-d21092fa41fe" />

#### Resources:
- https://www.cybertriage.com/blog/ntuser-dat-forensics-analysis-2025/
- https://www.4n6post.com/2023/02/registry-wordwheelquery.html

<br>

### **LNK Files**
#### Location: C:\Users\%USERNAME%\AppData\Roaming\Microsoft\Windows\Recent

LNK files are Windows shortcut files that are automatically created by Windows whenever a user opens files. These LNK files are used by Windows to ensure quick access to certain files. LNK files can be used as evidence that a user accessed a file. LNK files contain information including:
- Name of the file
- Original path to the target file (the file the shortcut is referencing)
- MAC timestamps of the target file and the .lnk file
- The size of the target

#### Parsing Tool: LECmd
LECmd is a tool created by Eric Zimmerman that parses LNK files. To parse an entire directory of LNK files, execute the following command:

```powershell
.\LECmd.exe -d "<path_to_lnk_directory>" --csv . --csvf <out_filename>.csv
```

- Where -d specifies the directory containing LNK files
- --csv . specifies to output as CSV file in the current directory, and
- --csvf specifies the output filename
You can then use a tool like Timeline Explorer to view the output:

<img width="940" height="172" alt="Image" src="https://github.com/user-attachments/assets/9003c498-3841-496d-9fb3-6029eb735390" />

To parse a single LNK file, you can use the -f option:

```powershell
.\LECmd.exe -f "<path_to_lnk_directory>" 
```

<img width="543" height="669" alt="Image" src="https://github.com/user-attachments/assets/5b0d048d-58cd-4b7b-88a1-43510f304ec4" />

The images above only show a small snippet of the wealth of information you can get from LNK files. 

#### Limitations:
- Timestamps are not always accurate

#### Resources:
- https://www.thedfirspot.com/post/a-lnk-to-the-past-utilizing-lnk-files-for-your-investigations
- https://www.youtube.com/watch?v=wu4-nREmzGM&pp=0gcJCfwAo7VqN5tD

<br>

### **JumpLists**
#### Location: C:\%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations

The Windows task bar (Jump List) was created to enable users to “jump” to items they have frequently or recently used. The AutomaticDestinations Jump List files are created automatically when the user opens a file or an application. 

#### Location: C:\%UserProfile%\AppData\Roaming\Microsoft\Windows\Recent\CustomDestinations

The CustomDestinations Jump List files are custom made Jump Lists, created when a user pins a file or an application to the Taskbar or Start Menu. 

#### Tool: JumpList Explorer

JumpList Explorer is another Eric Zimmerman tool that can be used to view both AutomaticDestinations and CustomDestinations. Upon loading the files, you will be presented with something like as follows:

<img width="940" height="168" alt="Image" src="https://github.com/user-attachments/assets/444a56fd-8661-49bb-bc7d-0f99f75e6838" />

#### Resources:
- https://www.youtube.com/watch?v=wu4-nREmzGM&pp=0gcJCfwAo7VqN5tD
- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://www.magnetforensics.com/blog/exploring-the-significance-of-jump-lists-in-digital-forensic-examinations/
- https://www.forensafe.com/blogs/jumplist.html

<br>

### **Activities Cache Database (ActivitesCache.db)**
#### Location: C:\Users\%USERNAME%\AppData\Local\ConnectedDevicesPlatform\L.%USERNAME%\ActivitiesCache.db

The Windows activity history keeps track of application and services usage, files opened, and websites browsed for Windows 10 machines. 

#### Parsing Tool: WxTCmd
WxTCmd (Windows 10 Timeline database parser) is a tool created by Eric Zimmerman that we can use to parse the activities cache database:

```powershell
.\WxTCmd.exe -f "ActivitiesCache.db" --csv . 
```

#### Resources:
- https://medium.com/@joeforensics/digital-forensics-windows-10-timeline-activitescache-db-82b4bc9f715a
- https://artifacts-kb.readthedocs.io/en/latest/sources/windows/ActivitiesCacheDatabase.html
- https://binaryforay.blogspot.com/2018/05/introducing-wxtcmd.html

<br>

### **ThumbCache**
#### Location: C:\Users\%USERNAME%\AppData\Local\Microsoft\Windows\Explorer

Thumbcach is a feature on Windows which is used to cache thumbnail images of files for Windows Explorer view. Thumbnail cache entries are created when a user switches a folder to thumbnail mode or views pictures via a slideshow. Thumbcache files can also be used to prove that a file was stored on a Windows system event if deleted. 

#### Parsing Tool: ThumbCache Viewer

You can use a tool called [Thumbcache viewer](https://thumbcacheviewer.github.io/) to extract thumbnail images from the thumbcache db. 

<img width="940" height="231" alt="Image" src="https://github.com/user-attachments/assets/1550f6c6-0adf-4edc-98f3-abb6f0eb9201" />

If you click on a row, you can see the actual thumbnail image. To try and map the file paths, click Tools > Map File Paths. 

#### Limitations:
- Lack timestamps and path data for the original file, making it difficult to determine when a file or folder was last accessed or its file path (Thumcache viewer can’t map the file path unless the file is still present on disk). 

#### Resources:
- https://darkcybe.github.io/posts/DFIR_Evidence_of_File_and_Folder_Interaction/#internet-explorer-ie-and-edge-file-history
- https://thumbcacheviewer.github.io/
- https://www.ghacks.net/2014/03/12/thumbnail-cache-files-windows/
- https://youtu.be/5efCp1VXhfQ?si=ljTfC2tHPY1Jc7wp



