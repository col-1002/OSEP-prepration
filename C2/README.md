## Overview

A command and control (C2) server is software tasked to execute commands or binaries on a remote computer, or a network of computers. The primary focus of a C2 is to have a centralized management system where the operator can manage access to other machines somewhere in the network. An operator is the one who carries out the (simulated) attack and manages the software. The access can be gained in multiple ways, be it a SQL Injection vulnerability, weak credentials on different services such as SSH, RDP, or access to the initial machine (foothold) given by the client for a red team engagement or a penetration test. Red team is a group of people specializing in researching different ways getting into systems and refining tools, and being a step ahead of defenders.

A C2 server facilitates the creation of a specific executable, and once it is on the target machine, establishes a communication channel between the server and the target when executed. From here on, we are going to refer to these executables as **beacons**.

### Decoy
A **decoy** in red-teaming refers to any asset, such as a file, network service, or even a whole system, that is deliberately placed to mislead attackers or security systems. Decoys are used to:
- **Distract attackers** from valuable assets. By engaging with decoys, attackers reveal their methods, tools, and intentions without causing real harm.
- **Detect breaches** by monitoring interactions with these decoys, since any interaction is likely unauthorized and malicious. These are also known as honeypots or honeynets when referring to decoy systems or networks.
- **Study attacker behavior** by providing targets that, when interacted with, can give insights into the tactics and tools used by attackers.
- Tricks user that is normal

### Beacon
A **beacon** is a type of malware, code or executables that is used by attackers to establish a communication channel back to a C2 server. Beacons are used to:
- **Maintain persistence** on a compromised system. Beacons regularly "phone home" to a C2 server to confirm they are active and to receive further instructions.
- **Signal exfiltration opportunities** by indicating that the beacon has been successfully installed and the system is ready to be manipulated or used for further infiltration.
- **Manage control** over the compromised system by allowing the attacker to send commands and receive data through the beacon's communication channel.

Predominantly, C2 servers are used by the red team. It is a focused, goal-oriented security testing approach to achieve specific objectives. The objectives closely follow the Cyber Kill Chain.

---

### Attack Lifecycle

Tham khảo tài liệu này https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html        
Ngoài ra, còn một khía cạnh nữa chưa được đề cập: OpSec (Operation Security) - Đây là khía cạnh mà kẻ thù giảm thiểu dấu vết để che giấu sự hiện diện của mình trên hệ thống mục tiêu.


Mình xin được phép trích lời anh Bình:

"Điểm mình thích ở Sliver là nó cho phép mình chạy .Net assembly dễ dàng và nhanh chóng còn metasploit thì quá dài (các bạn sẽ cảm nhận rõ khi gặp bài có Applocker khi mà ta phải compile lại để có thể sử dụng với Living Off The Land Binaries, Scripts and Libraries (https://lolbas-project.github.io/)" 

------------Metasploit
```bash
meterpreter > background
use /post/windows/manage/execute_dotnet_assembly/
set SESSION 1
set DOTNET_EXE /opt/SharpCollection/Seatbelt.exe
set ARGUMENTS '-group=system'
run
```

---------Sliver
```bash
seatbelt -- -group=system
execute-assembly /home/kali/tools/Seatbelt.exe -group=system
```

`sideload /home/kali/Desktop/shell/mimikatz/x64/mimikatz.exe '' \"kerberos::list /export\" \"exit\" `

-----------Tất nhiên Sliver cũng có điểm yếu của nó và mình đã dùng metasploit để khắc phục điều đó:
```
1 File stage của sliver là quá lớn => Sử dụng metasploit làm initial access
2 Khả năng sử lí access token của sliver không ổn định => Sử dụng metasploit để steal hay migrate vào tiến trình có token khác
3 Cả 2 tunnel của bọn này đều lởm => Ligono-ng
```




