# Setting Up an New Dev Machine on - AMD Fizen 7 

## Installing Chrome and VSCode using Installer
- Installed Chrome
  - Downloaded: google-chrome-stable_current_amd64.deb
- Installed VS Code
  - Downloaded: code_1.87.2-1709912201_amd64.deb
 
## Installing IntelliJ using tar command
- installed IntelliJ
  - Downloaded: ideaIU-2023.3.6.tar.gz
  - Commands
    ```
    $ sudo tar -xzf ~/Downloads/ideaIU-2023.3.6.tar.gz -C /usr/share/
    $ /usr/share/idea-IU-233.15026.9/bin/idea.sh 
    ```
  - Go for a 30 days license if no license key at hand
  - Use the menu ```Tools >  Create Desktop Entry``` to create the desktop entry

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

