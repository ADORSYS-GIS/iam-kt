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
```


## Activate 
