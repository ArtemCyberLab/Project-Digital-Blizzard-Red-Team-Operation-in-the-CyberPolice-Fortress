Project Goal
To conduct covert reconnaissance and infiltration of an internal web application running on a non-standard port of a device within the CyberPolice network, minimizing noise and detection, and gathering data for further access to core systems.

Project Description
This project simulates a red team attack on an abandoned device in the CyberPolice network (IP: 10.10.243.11), forgotten during a tech upgrade and described as a “lone iceberg.” The task is to identify available services, find hidden web applications, and gain access while bypassing security measures and minimizing logging. Special focus is placed on investigating the web service on port 50628, analyzing HTTP responses, and discovering hidden directories.

Part 1. Operation “Digital Blizzard”: Initial Recon and Entry Point Discovery
Slide 1. Initial Recon with nmap
bash
Copy
Edit
nmap -sC -sV -vv 10.10.243.11
Ports found: 22 (ssh), 23 (telnet), 8080 (http).

Apache 2.4.57 Debian on port 8080, access restricted (403).

Suspicious port 50628 discovered later.

Slide 2. Directory Enumeration on Port 8080 (gobuster)
bash
Copy
Edit
gobuster dir -u http://10.10.243.11:8080 -w /usr/share/wordlists/dirb/common.txt -t 30
Multiple directories protected (403), /demo accessible.

/vendor redirects.

Slide 3. Full Port Scan (nmap -p-)
bash
Copy
Edit
nmap -p- -T 5 10.10.243.11 -vv
Additional open port 50628 found.

Ports 22, 23, 8080 confirmed.

Slide 4. Attempts to Access Port 50628
Simple requests to http://10.10.243.11:50628/ result in connection errors (EOF).

Working URL manually found: /en/login/asp.

Slide 5. Fuzzing Attempts with gobuster and ffuf
Gobuster fails when fuzzing /en/ and /.

ffuf with large wordlist and extensions .asp, .aspx, .php results in all requests failing with connection errors.

Slide 6. Situation Analysis — “Silent Hunter” Tactics
Server blocks flooding and automated scanning.

Manual analysis of /en/login/asp required for next steps.

Important to minimize noise and focus on specific path.

Slide 7. Plan for Next Stage
Analyze HTML login form at /en/login/asp.

Search for vulnerabilities to bypass authentication silently.

Attempt low-noise credential brute-forcing.

Use results to advance further into the CyberPolice network.

Conclusion Part 1
The first infiltration phase — reconnaissance and data gathering — revealed server defenses but identified a key entry point. The next step is detailed analysis and exploitation of this point while maintaining stealth.

PART2  
Today I performed reconnaissance on the web server at 10.10.0.29 to find accessible directories and files.

1. Initial scan for common PHP files and directories
I used a wordlist of common PHP files and directories. All requests returned 403 Forbidden, except the /vendor/ directory, which returned a 301 Redirect.

ffuf -w common-php-files.txt -u http://10.10.0.29/FUZZ -mc 200,301,403
Result:

/vendor/ → 301 (redirect)

Others → 403 (forbidden)

2. Scanning common directories under root /
I checked a list of common directories at the site root.

Command:
ffuf -w common-directories.txt -u http://10.10.0.29/FUZZ/ -mc 200,301,403
Result:

All returned 403 Forbidden, including /vendor/.

3. Searching for files and directories under /en/ with extensions .php, .asp, .aspx
I tried common filenames with different extensions in the /en/ directory:

ffuf -w common-filenames.txt -u http://10.10.0.29/en/FUZZ.php -mc 200,301,403
ffuf -w common-filenames.txt -u http://10.10.0.29/en/FUZZ.asp -mc 200,301,403
ffuf -w common-filenames.txt -u http://10.10.0.29/en/FUZZ.aspx -mc 200,301,403
Result:

All requests either returned 403 Forbidden or no response.

4. Connection refused on port 80
When trying to connect to HTTP on port 80, I encountered an error:

curl http://10.10.0.29
Response: connection refused — indicates the service is not running on this port or it is blocked.

5. Next steps
Run a port scan with nmap to identify active services:

nmap -sC -sV -p- 10.10.0.29
After finding the open port with the web server, repeat the directory search on the correct port.

Use more comprehensive wordlists and different bypass methods for access restrictions.
