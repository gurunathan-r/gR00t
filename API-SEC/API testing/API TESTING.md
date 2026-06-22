

# Passive reccon

- Google Dorking
- Github - swagger.json
- shodan
- wayback

# Active reccon

- Nmap  - (-sC -sV , -p-)
- amass - enum -active -d
- gobuster usinng a wordlist ( --wildcard -b 200)
- postman

# Reverse Engineering an API

mitmproxy and analyse every interaction and then create a swagger yml file

sudo mitmproxy2swagger -i ~/Downloads/flows -o spec.yml -p http://crapi.apisec.ai -f flow
 and remove the ignore for api endpoints
 
 sudo mitmproxy2swagger -i ~/Downloads/flows -o spec.yml -p http://crapi.apisec.ai -f flow --examples

editor.swagger.io


#### API Documentation Conventions

|   |   |   |
|---|---|---|
|Convention|Example|Meaning|
|: or {}|/user/:id<br><br>/user/{id}<br><br>/user/2727<br><br>/account/:username<br><br>/account/{username}<br><br>/account/scuttleph1sh|The colon or curly brackets are used by some APIs to indicate a path variable. In other words, “:id” represents the variable for an ID number and “{username}” represents the account username you are trying to access.|
|[]|/api/v1/user?find=[name]|Square brackets indicate that the input is optional.|
|\||“blue” \| “green” \| “red”|Double bars represent different possible values that can be used.|


