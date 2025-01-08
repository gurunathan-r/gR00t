
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
- On decoding it twice with base64 we get the starting of the flag

