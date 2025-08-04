# Malicious Word Document Analysis

This document provides an analysis of a suspicious [Microsoft Word document](https://bazaar.abuse.ch/browse.php?search=file_type%3Adocx) flagged as malicious. This analysis was conducted using a combination of static and dynamic analysis techniques, utilising tools to dissect and analyse the file to uncover its behaviour. Please note that I am in no way shape or form an experienced malware analyst, I am simply trying to learn so any feedback or advice is much appreciated. 

### **File Identification**

The file was confirmed to be a Microsoft Word document using tools such as trid and the file utility.

<img width="825" height="166" alt="Image" src="https://github.com/user-attachments/assets/d730022e-14ec-4de9-8899-ca27ac529d6b" />

<br>

### **Macro Detection and Behaviour**

Using [oleid](https://github.com/decalage2/oletools/wiki/oleid), a python script that analyse OLE files like Microsoft Office documents, I was able to discover the presence of embedded VBA macros.

<img width="617" height="478" alt="Image" src="https://github.com/user-attachments/assets/08f3079c-3605-43e1-9264-924cebe453b7" /> 

I confirmed this using another tool called [olevba](https://github.com/decalage2/oletools/wiki/olevba) which is able to detect and extract VBA macros among other pieces of crucial information. 

<img width="816" height="137" alt="Image" src="https://github.com/user-attachments/assets/074c7408-fde0-483d-83d2-b7ef19ede9f0" />

<img width="748" height="472" alt="Image" src="https://github.com/user-attachments/assets/4f0a64a8-8713-497a-b894-ef5bd06267f7" />

To analyse the VBA macros, I used oledump to find the location of these Macros and dump them:

<img width="668" height="199" alt="Image" src="https://github.com/user-attachments/assets/4c5aed40-780d-4167-a054-f452f847d2ae" />

- **Object 7:** Writes and executes a file named auxiliary2.aux, likely a malicious payload. This macro leverages the AutoOpen function for immediate execution upon running the document. 

<img width="749" height="413" alt="Image" src="https://github.com/user-attachments/assets/2f03feeb-c097-403b-b69c-5f3abf0b7ed1" />
 
- **Object 10:** Executes the auxiliary file through a function named Calculate_values. 

<img width="747" height="182" alt="Image" src="https://github.com/user-attachments/assets/0a063085-c1f8-4687-8135-0815a6a80b23" />
 
- **Object 11:** Attaches a malicious template (Base.dotm or Normal.dotm) for persistence. It also employs anti-analysis techniques, such as deleting macro code after execution. 

<img width="582" height="440" alt="Image" src="https://github.com/user-attachments/assets/57b25473-f3d3-4686-a473-68e37e8af8dd" />
 
- **Object 12:** Determines the system architecture (32-bit or 64-bit) and reads binary data, likely to deliver an appropriate payload. 
- **Objects 13 and 14:** Contain obfuscated, decimal-encoded data. After decoding using a custom python script and cyberchef, it was revealed that these macros generate and write a Portable Executable (PE) file to disk.

<img width="444" height="377" alt="Image" src="https://github.com/user-attachments/assets/d4bc6b06-e2c7-4ec9-a7d8-b4b34a25c681" />

<img width="437" height="373" alt="Image" src="https://github.com/user-attachments/assets/162ac0cb-c1fc-4006-a869-fd8d649a9735" />

<br>

### **Obfuscated Payload Analysis**

Using a custom python script as seen below, the obfuscated macros in objects 13 and 14 were decoded, revealing 2 PE files. The extracted file’s SHA256 hashes were checked on VirusTotal, both receiving a large number of detections.

<img width="689" height="403" alt="Image" src="https://github.com/user-attachments/assets/12d24d55-6a22-46a8-a7e6-c6b8c0f9397d" />

<img width="940" height="234" alt="Image" src="https://github.com/user-attachments/assets/eba03197-1f5b-4b86-9f0e-925052f269cf" />

<img width="940" height="132" alt="Image" src="https://github.com/user-attachments/assets/56149cba-8474-4821-82ea-0e188e59c30d" />

**Network Indicators:**
- An IP address embedded in PE file was identified but it had no detection on VirusTotal. 

<img width="940" height="73" alt="Image" src="https://github.com/user-attachments/assets/50c91061-8d45-4410-9b04-3ba4f797d862" />

<br>

### **Dynamic Analysis**

The two extracted PE files were subjected to sandbox testing using Hybrid-Analysis. The dynamic analysis confirmed the malicious nature of the PE file but did not uncover new indicators of compromise (IOCs). 

<img width="940" height="215" alt="Image" src="https://github.com/user-attachments/assets/d2e2e84f-6fa5-4066-89b4-aa3259ccd36b" />
 
Next steps would be to analyse the two PE files using a sandbox like FLARE VM or AnyRun, to uncover more IOCs. 

### **Indicators of Compromise (IOCs)**

| IOC Type           | Details                                                                                       |
|--------------------|-----------------------------------------------------------------------------------------------|
| File Hash (docx file) | SHA256: 7d1fbe79df80ed442093510023b383c42749c4a689c1590f2d288402392e58e0 |
| File Hash (PE file)   | SHA256: b966869ad7c302cd97c8458a75929b57fc385baccd83b23b6d694b78ed085266 |
| File Hash (PE file)   | SHA256: 648f7aeac068f3fabda5ce6a0e56b149c430fe53d9bd2eb3dad330c04087ed90 |
