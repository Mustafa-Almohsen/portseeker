🚀 PortSeeker

**PortSeeker** is a fast, multithreaded Bash tool for discovering open TCP ports on a target IP address. Designed for **bug bounty hunters**, penetration testers, and network enthusiasts who want quick visibility into what ports are live — without installing Nmap or other heavyweight scanners.

-----------------------------------------

🔧 Features

- ✅ Multi-threaded scanning (custom thread count)
- ✅ Highlights open ports in **green** for visibility
- ✅ Option to scan only the top 1000 most common TCP ports
- ✅ Saves all results to a file
- ✅ Simple Bash script no installation required
- ✅ Help menu for easy usage

------------------------------------------

📦 Requirements

- Bash (v5+ recommended)
- curl


-----------------------------------------

🧪 Usage

Make the script executable:

( chmod +x portseeker.sh )


📌 Basic Usage

./portseeker.sh  <target>



⚙️ With Options

./portseeker.sh  192.168.1.1 --threads 30 --output results.txt --top


🛠 Options
Option	Description:

  Target IP or domain to scan (required)
--threads	Number of threads (default: 50)
--output	Save results to a specified file
--top	Scan only the top 1000 most common TCP ports
-h	Show help menu

🧠 Notes

  - If no --top flag is used, it scans all ports from 1-65535.

  - Ports are dynamically generated and scanned concurrently.

  - curl is used for lightweight TCP response probing.
