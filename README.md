![Untitled](https://github.com/user-attachments/assets/8c6a6fe1-f7b6-48c7-8c76-0313fd2a6d1c)
----
# 🕵️♂️ Investigating a Possible Local File Inclusion (LFI) Attack with Let's Defend SIEM

## **Introduction**  
Hello there! 👋 I'm excited to share another case from my cybersecurity journey — an investigation into a suspicious alert flagged by the Let's Defend SIEM platform. Whether you're a cybersecurity enthusiast or just curious about how attacks happen behind the scenes, I’ll walk you through every step in simple, clear language.  

Let’s dive in! 🚀  

**🔥 First, Let's start by defining a Local File Inclusion (LFI) Attack?**  
What if you visit a website and ask it to show you a picture or a document. Normally, the website should only let you see safe files it’s supposed to show. In an LFI attack, a hacker tricks the website into showing files it shouldn’t — files that are private and critical to the system, like user account details.  

In this case, the attacker tried to access the server’s `/etc/passwd` file — a critical file on Linux systems that holds user account information. Even though modern systems protect passwords better, just accessing this file can give attackers a big foothold.  

In short:  
🔹 **LFI** = Tricking a website into giving access to private files.  
🔹 **Why it's bad?** = Private information can leak, and bigger attacks can follow.  

**🛎️ Overview of the Alert We Investigated**  
Here’s the snapshot of the suspicious event Let's Defend flagged:  

![1745784757741](https://github.com/user-attachments/assets/6e81c5ec-a523-443f-9518-2430c7ed7d9f)

## **🧠 Step 1: Why Understanding the Alert is Crucial Before Investigation**  
Before rushing into action, I made sure to first understand why the alert was triggered.  
🔹 **Why?** Without knowing the background, you risk making wrong assumptions and wasting time.  
**Lesson:** Always understand why the alarm went off first!  

In this case, the rule name itself ("Passwd Found in Requested URL") hinted it was likely about trying to access sensitive files. This already smelled like a Directory Traversal or Local File Inclusion attempt!  

## **🕵️♂️ Step 2: Digging Deeper - Filtering Logs for the Suspicious Traffic**  
I jumped over to the Log Management section. Using the suspicious IP `106.55.45.162`, I filtered HTTP traffic to focus only on what matters.  

![1745784923111](https://github.com/user-attachments/assets/b8f526cb-deaa-4f26-8174-c334b349a941)

And there it was — the very suspicious URL:  
`?file=../../../../etc/passwd`  

**💬 What’s happening here?**  
The `../../../../` part shows the attacker trying to climb up directories — like stepping out of a room and sneaking into the admin office!  

In tech terms:  
This is a **Directory Traversal Attack**, trying to break out of the web directory to access critical files.  

## **🎯 Step 3: Is This Traffic Malicious?**  

![1.745785218282](https://github.com/user-attachments/assets/747493ba-7a04-462e-8ab6-9b43d7087ef4)

✅ Based on everything seen — the pattern, the target file (`/etc/passwd`), and the technique — it is highly likely this is malicious.  

**Attacker's goal?**  
- Steal sensitive server files.  
- Learn about the system’s users for future attacks.  

## **🏷️ Step 4: Confirming the Attack Type**  

![1745785263245](https://github.com/user-attachments/assets/0dccc4e8-b203-46fa-95e4-a62596f743c0)

🧩 **Attack Type:** → Local File Inclusion (LFI) through Directory Traversal.  

**💡 Signs to recognize an LFI attack:**  
- URLs showing suspicious file paths like `../../`.  
- Requests trying to access `/etc/passwd` or other server files.  
- Unexpected file parameter manipulations.  

## **📬 Step 5: Could This Have Been a Penetration Test?** 
.
![1745785398683](https://github.com/user-attachments/assets/61bce536-d137-4b20-95be-5b12af7655ee)

In cybersecurity, companies sometimes simulate attacks to test their defenses. Before raising an alarm, we must check if this was a drill.  

✅ I headed to the Email Security section and searched using key details:  
- Hostname (`WebServer1006`)  
- IP address (`172.16.17.13`)  
- Keywords like "local file inclusion", "etc/passwd", "traversal test" etc.

![1745785895280](https://github.com/user-attachments/assets/de5d5a5a-c092-45bf-b68b-1cf611789d6a)

📬 **I found no indication of a penetration test.**  
🚨 **This was a real attack, not a drill!**  

## **🌍 Step 6: What About the Source of the Traffic?**  

![1745786006683](https://github.com/user-attachments/assets/3005a92a-5f89-47c7-a2c4-9757cfa11649)

Let’s look closer:  
- Source IP: `106.55.45.162`  
- Destination IP: `172.16.17.13`  

🔹 **How do we know the source IP is external (from the internet)?**  
Private IP ranges (internal networks) are usually:  
- `10.0.0.0 – 10.255.255.255`  
- `172.16.0.0 – 172.31.255.255`  
- `192.168.0.0 – 192.168.255.255`  

`106.55.45.162` doesn’t fall into any private range → It’s a public IP, likely from outside the organization.  

🌐 **Direction of Traffic:**  
From Internet ➡️ Company’s Network.  

## **📏 Step 7: Was the Attack Successful?**  

![1745787766943](https://github.com/user-attachments/assets/32e907a1-932f-4abe-a57f-e7f3f3fe9b94)

A critical part: Did the attacker succeed?  

To know, I examined the HTTP response code and response size:  
- HTTP Response Code: `500` (Server Error)  
- Response Size: `0 bytes`  

💬 **What does this mean?**  
- `500 Error` = Server couldn’t process the malicious request (good sign!).  
- `0-byte response` = No data was leaked.

![1745786966479](https://github.com/user-attachments/assets/4d6e4b18-977c-4b8f-acbe-92eddaaae9c0)

✅ **Conclusion:** The attack failed. No files were exposed.  

## **📋 Step 8: Documenting the Artifacts**  

![1745787009258](https://github.com/user-attachments/assets/82379f1c-5ca5-4173-a3c0-6ca3082f0c0b)

Even though the attack failed, documenting evidence is critical for future reference and threat intelligence.  

📄 **Artifacts documented:**  
- Malicious IP Address: `106.55.45.162`  
- Malicious URL: `https://172.16.17.13/?file=../../../../etc/passwd`  

Proper documentation helps in pattern recognition if similar attacks happen again.  

## **✍️ Step 9: Writing the Analysis Comment**  
I summarized the investigation in the incident's comment section. Here’s a snippet of the comment I added:  
> *"Detected LFI attack attempt via Directory Traversal. Attempted access to /etc/passwd file. Traffic originated externally. No successful data exfiltration. IP Address documented for threat tracking."*  

Simple, clear, and concise. It saves time for anyone reviewing the case later.  

## **✅ Step 10: Closing the Alert**  
Since it was a real attack but not successful, I closed the alert as a **True Positive**. This shows that the SIEM rules were correct and that our detection capability is solid!  

![1745787485530](https://github.com/user-attachments/assets/532085cd-743c-4296-9acd-07fd81479b5e)

## **🚀 Conclusion**  
The investigation confirmed this was a **true positive Local File Inclusion (LFI) attack attempt**. An external attacker (IP: `106.55.45.162`) tried to access sensitive files through directory traversal but failed, as shown by the HTTP `500` error and zero response size.  

There was no evidence of a planned penetration test, confirming it was a real attack. Since the attempt was unsuccessful and no data was compromised, escalation to Tier 2 was not necessary.  

The malicious artifacts were documented, a summary was added to the incident comments, and the alert was closed appropriately. This underscores the importance of careful alert handling and thorough investigation to strengthen security defenses.  


No changes were made to the original text. Let me know if you'd like any further assistance!
