# Learning oAuth2 with Keycloak


## Start Keycloak in a docker Container

```
# We will run keycloak in a container. No bare metal.
# Mapping the docker port to a local port: -p 8080:8080
# Setting the admin user and password: -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin
% docker run -d -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin -p 8080:8080 --name my-keycloak quay.io/keycloak/keycloak start-dev

# now display the logs
# <49ed.....> this is the container id
# docker logs -f dock 4 (first n unique characters sufficient)
```

## Perform configurations as and Admin

### Start a Shell inside the Keycloak container

```
# new terminal
% docker exec -it 4 /bin/bash
bash-5.1$

###########################################
# Keycloak directory is /opt/keycloak
# Login as admin in the master realm
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh config credentials --server http://localhost:8080 --realm master --user admin --password admin
# output: Logging into http://localhost:8080 as user admin of realm master

###########################################
# Do not operate application in the master realm.
# Create a demoreal for our demo application
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh create realms -s realm=demorealm -s enabled=true
# output: Created new realm with id 'demorealm'

###########################################
# Create a demoreal for our demo application
###########################################
# create a user demo in the demorealm
bash-5.1$ /opt/keycloak/bin/kcadm.sh create users -r demorealm -s username=demo -s firstName=Demo -s lastName=User -s email=demo@user.me -s enabled=true
# output: Created new user with id '1a2b0e90-e783-4e87-9572-bcf68d805f89'

###########################################
# Set the password for demouser
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh set-password -r demorealm --username demo --new-password demouserpassword
# output:
```

## Login to the Account Console

on your browser, start an incognito window
- ativate the developer console
- - select the Application Tab
  - Verify that there is no cookies
  - delete any keycloak localhost cookies (if any)
- select the Network Tab
  - activate: preserve log

- navigate http://localhost:8080/realms/demorealm/account,
  - analyze network trafic on ```account/ -> http://localhost:8080/realms/demorealm/account/```
  - select headers tab and, in response header:
    - instrospect the cookie on jwt.io: KEYCLOAK_IDENTITY (look at "session_state")
    - look at the cookie: KEYCLOAK_SESSION and compare.
- login and see the account console with username=demo password=demouserpassword
  - analyze network trafic

```
###########################################
# First request sent by keycloak
###########################################
curl 'http://localhost:8080/realms/demorealm/protocol/openid-connect/auth......'

###########################################
# Then we login
###########################################
curl 'http://localhost:8080/realms/demorealm/login-actions/authenticate ......'

###########################################
# Then we are reirected to
###########################################
curl 'http://localhost:8080/realms/demorealm/account/?state=6819bef2-a94e-4d12-8cd7-81d78796653a&session_state=00b738f8-41c2-4ff6-9c1b-57085eb94921&iss=http%3A%2F%2Flocalhost%3A8080%2Frealms%2Fdemorealm&code=2cef75c6-7c86-4a9c-8c7f-14ff6a77570a.00b738f8-41c2-4ff6-9c1b-57085eb94921.a5a408e2-a74c-4196-90c9-47b940df0d83' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  -H 'Cache-Control: max-age=0' \
  -H 'Connection: keep-alive' \
  -H 'Cookie: AUTH_SESSION_ID=00b738f8-41c2-4ff6-9c1b-57085eb94921; AUTH_SESSION_ID_LEGACY=00b738f8-41c2-4ff6-9c1b-57085eb94921; KEYCLOAK_IDENTITY=eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI2OWQ3MDhjOS1iZjY0LTQzYWYtOTZmNS1iNTQ0NzIwNjZiNjEifQ.eyJleHAiOjE3MTE4MjE3MTksImlhdCI6MTcxMTc4NTcxOSwianRpIjoiODc2ZmZmMjMtZmY4NC00MDM2LWE1NDQtYTk1ZDM5MTE4MGJhIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9kZW1vcmVhbG0iLCJzdWIiOiIxYTJiMGU5MC1lNzgzLTRlODctOTU3Mi1iY2Y2OGQ4MDVmODkiLCJ0eXAiOiJTZXJpYWxpemVkLUlEIiwic2Vzc2lvbl9zdGF0ZSI6IjAwYjczOGY4LTQxYzItNGZmNi05YzFiLTU3MDg1ZWI5NDkyMSIsInNpZCI6IjAwYjczOGY4LTQxYzItNGZmNi05YzFiLTU3MDg1ZWI5NDkyMSIsInN0YXRlX2NoZWNrZXIiOiIyY1pnNWI4bXlBa0pSSk1tQUhfNFJYSFgxU2pZaXNNclhMczZMajlkaXIwIn0.GxRxwvfgMG2eP7a4Hd99NtoQbiZNjCNuYXMfFG1W6UMTNVFD67Ku6LtwIUQaiuFI63F8TpeVByjJlA357edyPw; KEYCLOAK_IDENTITY_LEGACY=eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI2OWQ3MDhjOS1iZjY0LTQzYWYtOTZmNS1iNTQ0NzIwNjZiNjEifQ.eyJleHAiOjE3MTE4MjE3MTksImlhdCI6MTcxMTc4NTcxOSwianRpIjoiODc2ZmZmMjMtZmY4NC00MDM2LWE1NDQtYTk1ZDM5MTE4MGJhIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9kZW1vcmVhbG0iLCJzdWIiOiIxYTJiMGU5MC1lNzgzLTRlODctOTU3Mi1iY2Y2OGQ4MDVmODkiLCJ0eXAiOiJTZXJpYWxpemVkLUlEIiwic2Vzc2lvbl9zdGF0ZSI6IjAwYjczOGY4LTQxYzItNGZmNi05YzFiLTU3MDg1ZWI5NDkyMSIsInNpZCI6IjAwYjczOGY4LTQxYzItNGZmNi05YzFiLTU3MDg1ZWI5NDkyMSIsInN0YXRlX2NoZWNrZXIiOiIyY1pnNWI4bXlBa0pSSk1tQUhfNFJYSFgxU2pZaXNNclhMczZMajlkaXIwIn0.GxRxwvfgMG2eP7a4Hd99NtoQbiZNjCNuYXMfFG1W6UMTNVFD67Ku6LtwIUQaiuFI63F8TpeVByjJlA357edyPw; KEYCLOAK_SESSION="demorealm/1a2b0e90-e783-4e87-9572-bcf68d805f89/00b738f8-41c2-4ff6-9c1b-57085eb94921"; KEYCLOAK_SESSION_LEGACY="demorealm/1a2b0e90-e783-4e87-9572-bcf68d805f89/00b738f8-41c2-4ff6-9c1b-57085eb94921"' \
  -H 'Sec-Fetch-Dest: document' \
  -H 'Sec-Fetch-Mode: navigate' \
  -H 'Sec-Fetch-Site: same-origin' \
  -H 'Sec-Fetch-User: ?1' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36' \
  -H 'sec-ch-ua: "Google Chrome";v="123", "Not:A-Brand";v="8", "Chromium";v="123"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "macOS"'

```

## Activate 
