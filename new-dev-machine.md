# Setting Up a New Dev Machine on - Thinkpad - AMD Ryzen 7 - Ubuntu

## Installing Chrome and VSCode using Installer
- Installed Chrome
  - Downloaded: google-chrome-stable_current_amd64.deb
- Installed VS Code
  - Downloaded: code_1.87.2-1709912201_amd64.deb
 
## Installing IntelliJ using snap command
- install IntelliJ Community
    ```
    $ sudo snap install intellij-idea-community --classic --edge
    ```
- install IntelliJ Ultimate
    ```
    $ sudo snap install intellij-idea-ultimate --classic --edge
    ```
- run IntelliJ Community
    ```
    $ intellij-idea-community
    ```
- run IntelliJ Ultimate
    ```
    $ intellij-idea-ultimate
    ```
  
 ## For large mvn build
 Open you the file ```~/.profile``` with your prefered editor. In th botom, add the line
 ```
export MAVEN_OPTS="-Dmaven.build.cache.enabled=true"
```
sae and close the file.

## Installing Git using APT
From a terminal window:
```
$ sudo apt update
$ sudo apt install git
$ git --version
$ git config --global user.name "Your Name e.g.Francis Pouatcha"
$ git config --global user.email "Your adorsys email e.g fxxxxx.pxxxxx@adorsys.com"
```

## Installing OpenJDK from the command line
From a terminal window:
```
$ sudo apt update
$ sudo apt install openjdk-21-jdk-headless
```

## Cleanup the machine
If there are some packages that are no longer needed.
```
$ sudo apt autoremove
```

## Install podman
```
$ sudo apt update
$ sudo apt install podman
```

