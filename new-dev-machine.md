# Setting Up an New Dev Machine on - AMD Fizen 7 

## Using Installer
- Installed Chrome
  - Downloaded: google-chrome-stable_current_amd64.deb
- Installed VS Code
  - Downloaded: code_1.87.2-1709912201_amd64.deb
 
## Using tar command
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
 
