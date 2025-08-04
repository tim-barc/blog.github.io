# Enhancing Investigations in Wireshark through using MaxMind GeoIP databases

Mapping addresses to their geographical locations can be incredibly beneficial when investigating PCAPs in Wireshark. Utilising GeoIP within Wireshark enables analysts and network forensic investigators to analyse network traffic with additional context, enhancing their ability to identify anomalous and malicious traffic. This document outlines the process of setting up GeoIP in Wireshark (for free) and discusses its applications and benefits for cybersecurity professionals. 

### **Setting up GeoIP in Wireshark**

To start, create an account on [GeoLite2](https://www.maxmind.com/en/geolite2/signup?utm_source=kb&utm_medium=kb-link&utm_campaign=kb-create-account): 

<img width="741" height="474" alt="Image" src="https://github.com/user-attachments/assets/ccf25251-84f4-48ff-b534-8533e2dbd156" />

Once you have gone through the account creation process, you will be presented with a page like as follows:

<img width="940" height="234" alt="Image" src="https://github.com/user-attachments/assets/2162ad8c-933e-49ed-825a-c1bbf503d783" />

We want to download all these databases. To do so, simply click the Download Databases button and download those 3 databases as a GZIP file. Make sure to unzip the files, after doing so, you should have something like as follows:

<img width="309" height="139" alt="Image" src="https://github.com/user-attachments/assets/95674b09-d16d-4d6c-8d95-a78c9a93a802" />

<br>

### **Configuring Wireshark**

Open up Wireshark and navigate to Edit -> Preferences:

<img width="429" height="735" alt="Image" src="https://github.com/user-attachments/assets/5af67792-67ae-4e0b-84e8-6b99f36cd148" />

Then navigate to the Name Resolution section and click edit next to ‘MaxMind database directories’:

<img width="614" height="578" alt="Image" src="https://github.com/user-attachments/assets/6a19b041-2bb2-4904-abe8-c2be6429a86e" />

Once you have click edit, all we need to do is click the + button and add the location of the folder where all three databases are installed:

<img width="469" height="73" alt="Image" src="https://github.com/user-attachments/assets/cfd7598b-9e9f-42cc-9379-07cd4c4d9090" />

That’s it! we can now see the GeoIP information for a packet. 
Let’s test this out using a PCAP file which captured a DDoS attack. To see the GeoIP information for a packet, simply expand the IP field in the packet details pane and look at the bottom of its details:

<img width="742" height="353" alt="Image" src="https://github.com/user-attachments/assets/3ec8dcb0-80eb-4ad4-bcad-e736558ead85" />

If you expand the Source GeoIP field, you will see something like the following:

<img width="814" height="631" alt="Image" src="https://github.com/user-attachments/assets/6bd29030-7953-4ebe-8a9c-e41116545016" />

Don’t freak out if you can’t see the GeoIP information, first make sure that you are investigating a packet with external IP addresses as it can identify the geolocation for private IP addresses. The information seen above is very handy, as we could simply add the source city and country as a column and maybe drill down for traffic from unusual countries, like Russia for example. You can also see the GeoIP information in the Statistics -> Endpoints screen:

<img width="940" height="166" alt="Image" src="https://github.com/user-attachments/assets/c29668e5-d678-48ef-894c-1f2a96737497" />

<br>

### **Visualising the Data**

Another great transformation we can do is click the Map button on this Endpoint screen and select open in browser:

<img width="273" height="130" alt="Image" src="https://github.com/user-attachments/assets/1cb32343-23d8-4ffd-9953-36c2b51e4a26" />

<img width="940" height="413" alt="Image" src="https://github.com/user-attachments/assets/b079fb00-3ae5-4946-a912-350c419ab2ec" />

This provides a neat visualisation of all the locations where traffic originated from. 

<br> 

### **Conclusion**

Integrating GeoIP with Wireshark offers significant advantages for cybersecurity professionals. It enhances network traffic analysis by providing geographic context, aiding in incident response, and more. By following the setup steps outlined above, you should be able to leverage GeoIP data to improve your Wireshark investigations.
