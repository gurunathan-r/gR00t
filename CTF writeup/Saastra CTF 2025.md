
Web exploitation:--

BOOCCHI'S  SITE


![[Pasted image 20250108223407.png]]



- In this challenge we were given a site with a description that the site has some hidden info in it so i visited the site and found a static site written in html and css and there were only some tabs in the website based on some anime characters
- As it was mentioned in the description i searched for some hidden content in the website source code to see if i can find something in it 
- And i found something odd from the other contents of the html page which was not displayed in the website


```
<div class="about" notify_true="VTJoaFlYTjBjbUY3WDJKdlkyTm9hVjg9,follow kita">
```

- In this code there is a notify_true value which looks like a base64 encoded value 
- so i headed to https://cyberchef.org  to input the value to if it gives some info
- On decoding it twice with base64 we get the starting of the fl

![[Pasted image 20250108224744.png]]

- No i tried it for each and every tabs in the website with some anime character names and searched for some info and i get to see a hidden class

```
<p class = "hidden"> X3J5b18heWFtYWRhXw==, follow seika</p>
```

- here too i can see that there is a base64 encrypt value so i used base64 to decrypt and got a part of flag 
-                                  ```X3J5b18heWFtYWRhXw==              _ryo_!yamada_```
- i repeated the same for others and got the remaining part of the flag
- now its time to join the flag , i joined the flag in the order i got them and tried the flag 

- ```1. Shaastra{bocchiryo_!yamadakita-chan__seika!!}```
- but it doesnt work after rearranging it i got the correct flag

```
Shaastra{_bocchi__kita-chan__ryo_!yamada__seika!!}
```


