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
- select the Network Tab
  - activate: preserve log
- navigate to ```http://localhost:8080/realms/demorealm/account```
- select the application tab
  - delete any keycloak localhost cookies
- select the Network Tab
  - activate: clear all logs
- navigate to ```http://localhost:8080/realms/demorealm/account```
  - analyze network trafic on ```account/ -> http://localhost:8080/realms/demorealm/account/```
  - select headers tab and, in response header:
  - analyze network trafic on  ```init? -> http://localhost:8080/realms/demorealm/protocol/openid-connect/login-status-iframe.html/init```
  - analyze network trafic on  ```auth? -> http://localhost:8080/realms/demorealm/protocol/openid-connect/auth?client_id=account-console&redirect_uri...```
    - instrospect the cookie on jwt.io: KC_RESTART (look at "session_state")
    - look at the cookie: AUTH_SESSION_ID
- login and see the account console with username=demo password=demouserpassword
  - analyze network trafic on  ```authenticate? -> http://localhost:8080/realms/demorealm/login-actions/authenticate?session_code=...```
    - look at cookies sent with the request
    - look at cookies set with the response
      - KC_RESTART is reset
      - KEYCLOAK_IDENTITY is set. Instrospect cookie and look at: "session_state"
      - KEYCLOAK_SESSION is set. compare with session_state claim from KEYCLOAK_IDENTITY
    - response code is rediect to account console.
- On the account console
  - analyze network trafic on
    - ```
      account? -> http://localhost:8080/realms/demorealm/account/?
      state=07a55...a&
      session_state=f996fa51-d1ae-4d...&
      iss=http%3A%2F%2Flocalhost%3A8080%2Frealms%2Fdemorealm&
      code=ffb908fa-f5e7-4895-b666-4ad43685f6f5.f996fa51-d1ae-4d37-8f06-f1cbfea56e1e.a5a408e2-a74c-4196-90c9-47b940df0d83
      ```
    - look at cookies sent with the request
      - KEYCLOAK_IDENTITY is set. Instrospect cookie and look at: "session_state"
      - KEYCLOAK_SESSION is set. compare with session_state claim from KEYCLOAK_IDENTITY
    - look at cookies set with the response
      - KEYCLOAK_IDENTITY is set. Instrospect cookie and look at: "session_state". If you compare, only jti changed from former request.
      - KEYCLOAK_SESSION is set. compare with session_state claim from KEYCLOAK_IDENTITY
    - analyze further requests


## Where is the Token
We did not get any token. Token is obtained and managed by the client: the account console application.

## How do we see a Bearer Token.
Let us activate the direct grant flow on the account console.

```
# In the keycloak docker terminal
bash-5.1$

###########################################
# Login as admin in the master realm
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh config credentials --server http://localhost:8080 --realm master --user admin --password admin
# output: Logging into http://localhost:8080 as user admin of realm master

###########################################
# Query the client id of account console in the demorealm
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh get clients -r demorealm -q clientId=account-console -o --fields 'id,directAccessGrantsEnabled'
# output
# [ {
#  "id" : "<id printed here>",
#  "directAccessGrantsEnabled" : false
# } ]
#

###########################################
# Enable the direct access grant in the account console of the demorealm.
###########################################
bash-5.1$ /opt/keycloak/bin/kcadm.sh update clients/<ID above!!!> -r demorealm -s directAccessGrantsEnabled=true -o --fields 'id,directAccessGrantsEnabled'
# output
# [ {
#  "id" : "<id printed here>",
#  "directAccessGrantsEnabled" : true
# } ]
#
```

## Obtain a token with User=Demo

```
##############################
# New Shell
###############################
> curl -X POST http://localhost:8080/realms/demorealm/protocol/openid-connect/token -d "client_id=account-console" -d "username=demo" -d "password=demouserpassword" -d "grant_type=password" -d "scope=openid"
```

## Result is a bearer token

```
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJIdDdsN3RnajFBTFNpNi05Zkdyd0RaSkxtSlEwb0syRVJMS1l1NVhwNGVNIn0.eyJleHAiOjE3MTE3OTI1OTUsImlhdCI6MTcxMTc5MjI5NSwianRpIjoiMThiYTRmYjctZWI1MS00NDQyLThmZjAtMjNhZDE2NGY4NjE2IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9kZW1vcmVhbG0iLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiMWEyYjBlOTAtZTc4My00ZTg3LTk1NzItYmNmNjhkODA1Zjg5IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiYWNjb3VudC1jb25zb2xlIiwic2Vzc2lvbl9zdGF0ZSI6ImU4MjU2NThiLTU1YzktNDBkYi05MDE5LTlhNGFmYmQ4MDIzMyIsImFjciI6IjEiLCJyZXNvdXJjZV9hY2Nlc3MiOnsiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIl19fSwic2NvcGUiOiJvcGVuaWQgcHJvZmlsZSBlbWFpbCIsInNpZCI6ImU4MjU2NThiLTU1YzktNDBkYi05MDE5LTlhNGFmYmQ4MDIzMyIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkRlbW8gVXNlciIsInByZWZlcnJlZF91c2VybmFtZSI6ImRlbW8iLCJnaXZlbl9uYW1lIjoiRGVtbyIsImZhbWlseV9uYW1lIjoiVXNlciIsImVtYWlsIjoiZGVtb0B1c2VyLm1lIn0.1ovbtLpQvuZa-TvzSbx4Jfxyevl_d9PAcHu_N_bqgB4XuQ1bL9JwHhRML8hY-NZKpOlzE_LAgXVzxKy2eHbx47os3puCdmGV5YCbggdNgMxrV6B-tiwk94_gQG72m29Oy0bPDz0mqkUR1gt1V9Vgzv8juH-y91HFiQiucHIw23QtPhmdAaTOqV-GhSVwrNIQendLjGpO-KQsyBn6LUHWeo71pjDiisy5dP-8iyRUZf1oYL-Pyatqg5aK1bvmkHt6mOpz23t5h2O23XpNhPEiJ-qzEmXQfXjPjWSZSCIxUZfHemVI_CWlZc4PitN_v-wmPwkVljrHkbP1l_-FRBwGqA","expires_in":300,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI2OWQ3MDhjOS1iZjY0LTQzYWYtOTZmNS1iNTQ0NzIwNjZiNjEifQ.eyJleHAiOjE3MTE3OTQwOTUsImlhdCI6MTcxMTc5MjI5NSwianRpIjoiZTA1ZWRjNTAtNWE4ZC00NTY0LWJjOWQtNGJhOGIwY2Y0YWExIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy9kZW1vcmVhbG0iLCJhdWQiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvcmVhbG1zL2RlbW9yZWFsbSIsInN1YiI6IjFhMmIwZTkwLWU3ODMtNGU4Ny05NTcyLWJjZjY4ZDgwNWY4OSIsInR5cCI6IlJlZnJlc2giLCJhenAiOiJhY2NvdW50LWNvbnNvbGUiLCJzZXNzaW9uX3N0YXRlIjoiZTgyNTY1OGItNTVjOS00MGRiLTkwMTktOWE0YWZiZDgwMjMzIiwic2NvcGUiOiJvcGVuaWQgcHJvZmlsZSBlbWFpbCIsInNpZCI6ImU4MjU2NThiLTU1YzktNDBkYi05MDE5LTlhNGFmYmQ4MDIzMyJ9.4T9QY58P8ulRjd4PCCDpkkvKAiN4oa4VPa_soX45GCqivkHTWolktc9F0VS6rqYsVUpOhHBxpqfaCf7p1yMqrg","token_type":"Bearer","id_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJIdDdsN3RnajFBTFNpNi05Zkdyd0RaSkxtSlEwb0syRVJMS1l1NVhwNGVNIn0.eyJleHAiOjE3MTE3OTI1OTUsImlhdCI6MTcxMTc5MjI5NSwiYXV0aF90aW1lIjowLCJqdGkiOiIxNTUzNWE5Yi0zNWUzLTQzNDYtYjIyYS0yOGNlNTY4MTk4ZmIiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODAvcmVhbG1zL2RlbW9yZWFsbSIsImF1ZCI6ImFjY291bnQtY29uc29sZSIsInN1YiI6IjFhMmIwZTkwLWU3ODMtNGU4Ny05NTcyLWJjZjY4ZDgwNWY4OSIsInR5cCI6IklEIiwiYXpwIjoiYWNjb3VudC1jb25zb2xlIiwic2Vzc2lvbl9zdGF0ZSI6ImU4MjU2NThiLTU1YzktNDBkYi05MDE5LTlhNGFmYmQ4MDIzMyIsImF0X2hhc2giOiI5SS02bG4wV09vN3RXazNibFA3Z2VBIiwiYWNyIjoiMSIsInNpZCI6ImU4MjU2NThiLTU1YzktNDBkYi05MDE5LTlhNGFmYmQ4MDIzMyIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6IkRlbW8gVXNlciIsInByZWZlcnJlZF91c2VybmFtZSI6ImRlbW8iLCJnaXZlbl9uYW1lIjoiRGVtbyIsImZhbWlseV9uYW1lIjoiVXNlciIsImVtYWlsIjoiZGVtb0B1c2VyLm1lIn0.XqRQiC-H7NgFHVO4WxzpGaRK6lfOcq-fPef8VbRJMRl5t7hjWGmHa1EKyTXmsE8CrIt-w2TOmA4PYL4LnwS1cdtE4OfqS2feinGYRp6eHrWy-gOMJzrFKVaAf5mA_szF-n4PJOyqpFITAyEiRPG1GyS2LpwI5wviCHGn3fpAZHoe66jY9BcPwiVkxwLl6EK4XM8ugIL8Jlc0HlrfroruWyK_tecu469G9SUVbCQhEHsOwGQG1Zqv-9e1fz0_ehkC22NJFPHNTjWcDrbCakm3XBe6sqo93ryHdSl_Ue_bfHt2LbTY6KTWIu-iKFn-0R4Vd_R2mCpXN1l0WZLx5q8vmw","not-before-policy":0,"session_state":"e825658b-55c9-40db-9019-9a4afbd80233","scope":"openid profile email"}
```

ToDo: analyze the bearer token.
