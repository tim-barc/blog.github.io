# **Creating Timelines with Plaso and Log2Timeline**

<img width="1427" height="475" alt="Image" src="https://github.com/user-attachments/assets/a77b3279-64b7-4f70-9beb-5e42167e039f" />

For context, a timeline contains a series of events ordered in chronological order, by their timestamp, typically in UTC time. For the most part, these timelines aggregate a variety of artifacts, normalise them so as to have the same time zone and formats, and saves it to a file. This is where Log2Timeline or Plaso comes into play. To put it simply, Log2Timeline, powered by the Plaso engine, is a tool used to extract timestamps and forensic artifacts from various sources, creating a comprehensive “super timeline”. It aggregates and normalises these artifacts, and can be run against:

- Raw disk images 
- Specific partitions
- Physical devices
- Live mount points, and more. 

In this post, I will walk you through installing Plaso, creating a kitchen-sink timeline, and viewing the output in Timeline Explorer. For extensive information on the tool, please consult its [documentation](https://plaso.readthedocs.io/en/latest/sources/user/Using-log2timeline.html) or refer to the extra resources section.  

<br>

### **Table of Contents**
- [Installation (Ubuntu)](#installation-ubuntu)
- [Getting Started](#getting-started)
- [Inspecting the Storage File](#inspecting-the-storage-file)
- [Converting to CSV with psort](#converting-to-csv-with-psort)
- [Quick Approach with psteal](#quick-approach-with-pstealt)
- [Viewing the Output in Timeline Explorer](#viewing-the-output-in-timeline-explorer)
- [Making Timelines Useful](#making-timelines-useful)
- [Extra Resources](#extra-resources)

<br>

### **Installation (Ubuntu)**

Installation of Plaso is straight forward:  
- First, ensure the universe apt repository is enabled:

```bash
sudo add-apt-repository universe
```

- Add the GIFT PPA for Plaso:

```bash
sudo add-apt-repository ppa:gift/stable
```

- Update your package list and Install Plaso:

```bash
sudo apt-get update
sudo apt-get install plaso-tools
```

<br>

### **Getting Started**

To execute Plaso and see the help menu, you can enter the following command:

```bash
log2timeline.py
```

<img width="940" height="116" alt="Image" src="https://github.com/user-attachments/assets/a71010c1-6013-46d7-a4d1-542f43f04923" />

The kitchen-sink approach pulls every artifact Plaso can find:

```bash
log2timeline.py --storage-file <out_file> <source_file>
```

Where --storage-file specifies the output file name followed by the source file (disk image, mount point, etc). Processing can take minutes to days depending on the size of the source. Once completed, this outputs an SQLite database:

<img width="940" height="59" alt="Image" src="https://github.com/user-attachments/assets/c4b655fc-c099-4301-8667-725e187da15a" />

<br>

### **Inspecting the Storage File**

To extract the metadata associated with this file, you can use pinfo:

```bash
pinfo.py <plaso_dump > 
```

<img width="940" height="818" alt="Image" src="https://github.com/user-attachments/assets/c9aa1acb-b78e-479f-b759-8f7b1fd6ca85" />

<br>

### **Converting to CSV with psort**

We can now convert the Plaso dump file to a CSV by using psort. You can also use psort to specify timezones, time ranges, and much more:

```bash
psort.py -o l2tcsv -w <out_filename>.csv <plaso_dump_file> --output_time_zone utc
```

<br>

### **Quick Approach with pstealt**

If you want to quickly create a super timeline that automates all the steps we did previously, then I recommend using psteal:

```bash
psteal.py --source <source> -w <out>.csv
```

<br>

### **Viewing the Output in Timeline Explorer**

You can view the output using a variety of tools; however, Timeline Explorer is perfect for this. Timeline Explorer is a tool created by Eric Zimmerman that provides a great means of viewing CSV files. To load the CSV file into Timeline Explorer, you can either drag and drop it into the Timeline Explorer window or select the file through navigating to File > Open.

<img width="940" height="263" alt="Image" src="https://github.com/user-attachments/assets/73fe17eb-cc0c-4a38-b2ac-56da12a61b6b" />

As you can see in the above image, Timeline Explorer colour codes these artifacts, if you navigate to Help > Legend, you can see what the colour codes are associated with:

<img width="606" height="394" alt="Image" src="https://github.com/user-attachments/assets/c6411451-253a-4677-9caa-3e76fa0ef9c6" />

<br>

### **Making Timelines Useful**

A super timeline can feel overwhelming, containing potentially millions upon millions of lines. Instead of trying to review every event (which is borderline useless):

- **Target your search:** Filter by file extension, keywords, or time ranges
- **Pivot from key events:** e.g., suspicious file execution or web history. 

Take for example a host that has been compromised with ransomware. If you identify the extension appended to encrypted files (let’s, say .ransomed), you can search for this within the super timeline to see when the files started being encrypted (i.e., you will find a bunch of NTFS events). You can then pivot off this to identify what occurred before and after the ransomware was deployed, this could enable you to identify the actual ransomware binary, and events that occurred prior to it being executed. 

<br>

### **Extra Resources**
- [Getting Started with Plaso and Log2Timeline by 13Cubed](https://youtu.be/sAvyRwOmE10?si=W36kMifC1t6O-c29)
- [How to use log2timeline](https://medium.com/dfclub/how-to-use-log2timeline-54377e24872a)
- [Plaso and WSL 2 by 13Cubed](https://www.youtube.com/watch?v=g9V6OUCe12k)
