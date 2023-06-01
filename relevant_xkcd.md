# Relevant_xkcd
## Summary
This challenge incorpates a aspect of chatgpt, but in reality the path 
towards the flag is reading the flag through rediecting the program 
towards a incorrect link of our own choosing.

## Code
```python
import discord
from discord.ext import commands
import io
import logging
import openai
import os
import re
import urllib.request
...
# Other code relating to bot 
...
    async with ctx.typing():
        response = openai.ChatCompletion.create(
          model="gpt-3.5-turbo",
          # Prompt for Chatgpt
          messages=[
                {"role": "system", "content": "You are RelevantXKCD bot. Your sole purpose is to provide links to xkcd comics relevant to the message given to you by the user, based on how well it matches with the information provided on the ExplainXKCD website. You must always respond with a link to an xkcd comic. When you don't know of a link to an xkcd comic relevant to the message, you must instead respond with the link 'https://xkcd.com/{number}' where {number} is a randomly chosen number between 1 and 2781. Your response is always just a single link, no explanation or justification, just one link per message that you send."},
                {"role": "user", "content": "Lol whenever I see the 'permission denied' message I always just throw sudo in front of it without thinking about it"},
                {"role": "assistant", "content": "https://xkcd.com/149"},
                {"role": "user", "content": "agaiheroaerjaeiogahoierhg;eigahi"},
                {"role": "assistant", "content": "https://xkcd.com/213"},
                {"role": "user", "content": previous_message}
            ],
            max_tokens=200,
        )

        result = response['choices'][0]['message']['content'].strip()
        logging.info(f'\n{ctx.message.author.name}: {previous_message}')
        logging.info(f'RelevantXKCD: {result}')
        # !!! Find link in result !!!
        try:
            xkcd_link = re.search(r'https?:\/\/[^\s"\']+', result).group(0)
        except:
            await ctx.send(result)
            return
        if result != xkcd_link:
            await ctx.send(result)

        with urllib.request.urlopen(xkcd_link) as response:
            contents = response.read()
            # Parse reponse and grab Image URL
            img_url = re.search(r'Image URL \(for hotlinking\/embedding\):.*?href=.*?"(.*?)"', contents.decode()).group(1)
            filename = img_url.split('/')[-1]
            # Download image from link
            logging.info(f'Downloading file: {img_url}')
            with urllib.request.urlopen(img_url) as image:
                await ctx.send(file=discord.File(io.BytesIO(image.read()), filename))

@bot.event
async def on_command_error(ctx, error):
    await ctx.send(error)

bot.run(bot_token)

```

## Attack Path
For this challenge one the main revelations that helps, is remebering 
the file url scheme. For instance within your web browser you can tyoe 
`file:///path/to/file` which will open open the file. A additional 
realization is the fact that the requests.urlopen function supports
this url scheme. Meaning if we can somehow get the program to read the url
as this file inject, we can get the bot to send us any file that we know
the path of. 

## Exploitation
The payload in this case is fairly simple, we merely send the bot with a 
request to exactly respond with a url of our choosing. This url should be
under your control, and in this instance I used beeceptor to have this 
functionallity. Now that you have a server that you can call with a url 
from the interent, set it up to mirror the portion of the request that 
the bot expects, in this case the bot looks for the Image URl portion,
so have your server return thiss data with the file url payload. Now
when the bot reads this data it will strip any extra information. Then it
will take this stripped url and pass it into the urlopen function. Since 
that url will be the local path of the flag, we can have the bot send us
a copy of the flag.txt file, allowing us to submit it for points.