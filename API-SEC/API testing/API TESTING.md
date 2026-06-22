

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

sudo mitmproxy2swagger -i ~/Downloads/flows -o spec.yml -p
