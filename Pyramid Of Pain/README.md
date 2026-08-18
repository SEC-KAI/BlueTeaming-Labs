This tryhackme room talks about what the pyramid of pain is.
So the Pyramid of Pain consists of 6 layers, starting from the bottom, which is the easiest for attackers to evade, to the top,
which is the hardest.

Hashes – Defenders can use hash values to identify and block known malicious files. Attackers can bypass this fairly easily 
by slightly modifying the malware, which changes its hash value.

IP Addresses – Defenders can create rules to block specific malicious IP addresses. Attackers can bypass this by switching to 
a different IP address or server. It is not really about spoofing the IP in this case, but more about changing the 
infrastructure they are using.

Domain Names – Defenders can block domains that are known to be malicious. However, attackers can register another domain and 
use that instead, so this is still relatively easy for them to change.

Network Artifacts – Defenders can analyze network traffic and look for suspicious communication patterns, such as unusual GET 
or POST requests, strange URL paths, suspicious User-Agent strings, or repeated connections. This is harder to bypass because 
the attacker may have to change the way their malware communicates over the network instead of simply changing an IP or domain.

Tools – Defenders can analyze the actual tools an attacker is using and create ways to detect those tools, such as fuzzy 
hashing, YARA rules, antivirus signatures, or behavior-based detection. This is different from just using a normal hash 
because defenders can look for similarities in the tool's content, structure, or behavior. Attackers may now have to 
significantly modify the tool, create a new one, or use a different tool altogether.

TTPs – TTP stands for Tactics, Techniques, and Procedures. Defenders can use resources such as MITRE ATT&CK to understand 
how attackers perform certain techniques and then create detection rules around the evidence those techniques leave behind.
This is at the top of the pyramid because the attacker may have to change the actual way they carry out the attack, instead
of just changing a file, IP, domain, or tool.
