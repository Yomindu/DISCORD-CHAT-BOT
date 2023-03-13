# DISCORD-CHAT-BOT
AI baced chat bot for discord servers

Pre-requisites:
Discord account and server
OpenAI API Key
Replit Account
Python
Create a Discord Bot
There are plenty of Discord bot tutorials out there, but let’s walk through things together, step-by-step.

Create A Discord Server
Log into your free Discord account. At the bottom of Discord left side panel, you should see an option to “Add a Server”


Give it a fun name.

Create a New App in Discord Developer Portal
Visit the Discord Developer Portal here. Create a new Application.


Give it yet another memorable name.


In the left rail Settings, click on Bot.


Add a new bot and double confirm you want to proceed.


Take a moment to give it an icon then reset the token and save down your Discord Bot Token ID somewhere that you won’t forget it. You’ll need to add this to both an .env file and Repl secret later.


Edit Bot Permissions
Toggle on the Privileged Gateway Intents and Save Changes.


Authorize and Invite to Server
Under the OAuth2 settings tab, navigate to the URL Generator and give it a bot scope.


Give your Bot necessary bot permissions then copy the generated URL.


Paste the URL in a new tab.


Follow the steps to Authorize your new chat bot.


If successful you should reach this Authorized screen and see your new chat bot welcomed to your server.



Build a GPT3 Bot in Python
You are going to build your bot locally to start to make sure everything works as expected. First thing you need to do is install the packages in your local virtual environment.

pip install discord openai python-dotenv
Your code will use the openai library to interact with the OpenAI API, and the discord library to interact with the Discord API. You will use the load_dotenv library to manage your super secret API keys so you don’t accidentally publish them for the world to see (full disclosure I have done this before!)

Create an .env file and add your discord token to it

DISCORD_TOKEN = “YOUR-DISCORD-TOKEN-GOES-HERE”
Create a bot.py file and add the following code

# bot.py
import os
import discord
from dotenv import load_dotenv

load_dotenv()

TOKEN = os.getenv('DISCORD_TOKEN')

intents = discord.Intents.all()
client = discord.Client(command_prefix='!', intents=intents)

@client.event
async def on_ready():
  print('We have logged in as {0.user}'.format(client))

# start the bot
client.run(TOKEN)
Run your code with $ python bot.py

This will help you confirm you can connect to your Discord server with your authorization key. If successful, you should see the following response in your terminal.


Next you can test sending a structured message and getting a structured response from the bot.

# this is the code we will use first to test the connection
@client.event
async def on_message(message):
  if message.author == client.user:
    return
  if message.content.startswith(‘Hello Mr. Botface’):
    await message.channel.send(‘Howdy Stranger’)
Great. Now let’s rewire the bot to connect to OpenAIs GPT instead of your pre-meditated response.

You need to access our OpenAI API Key from their website.


Add your new secret key to your .env file and call it OPENAI_KEY

OPENAI_KEY = “YOUR-SUPER-SECRET-API-KEY”
You now need to import openai and then access your OPENAI_KEY from the env file.

import openai
import discord

load_dotenv()
TOKEN = os.getenv('DISCORD_TOKEN')
OPENAI_KEY = os.getenv('OPENAI_KEY')

# Set up the OpenAI API client
openai.api_key = OPENAI_KEY

intents = discord.Intents.all()
client = discord.Client(command_prefix='!', intents=intents)
You’ll set up the Discord client by calling discord.Client() and assigning it to client.

@client.event
async def on_message(message):
# Only respond to messages from other users, not from the bot itself
  if message.author == client.user:
    return
    
    # Use the OpenAI API to generate a response to the message
    response = openai.Completion.create(
    engine="text-davinci-002",
    prompt=f"{message.content}",
    max_tokens=2048,
    temperature=0.5,
    )
   # Send the response as a message
   await message.channel.send(response.choices[0].text)

# Start the bot
client.run("YOUR_DISCORD_BOT_TOKEN")
The on_message event is triggered whenever a message is sent in any of the Discord channels the bot is invited to.

To enable the bot to speak you’ll use the OpenAI API Completion.create() method. This will generate a response for any given message with the prompt coming from said message.

Then you need to add a max_tokens to limit the length of the chat bot response and tune the temperature parameter to match however crazy you want the chat bot to be. The lower it is the more of a loose cannon it is…

Finally, the generated response is then sent back to the channel as a message by using the message.channel.send() method.

You need to start the bot by calling client.run() and passing in your discord bot token. You’re now ready to run this file locally and give your bot a try.

Next you can modify the code so it only replies if you mention the bot with an @ mention. You can do this by adding a check to see if the bot name is mentioned in the message.

# trying to modify the code to respond only to the bot mention
@client.event
async def on_message(message):
  # Only respond to messages from other users, not from the bot itself
  if message.author == client.user:
    return
  
  # Check if the bot is mentioned in the message
  if client.user in message.mentions:
 
    # Use the OpenAI API to generate a response to the message
    response = openai.Completion.create(
    engine="text-davinci-002",
    prompt=f"{message.content}",
    max_tokens=2048,
    temperature=0.5,
    )

  # Send the response as a message
  await message.channel.send(response.choices[0].text)

# Start the bot
client.run(TOKEN)
If you re-run the script, your bot should only respond if you mention them.

Host the Bot on Replit
You’re ready to take things from your local environment to the cloud. You are going to use Replit to host your bot. They have a great tutorial you can reference if you get stuck here.

Create a Repl
Log into Replit with a free account and create a new Python repl.


You can use the main.py default file and copy your code directly into the file. You will need to make a few minor adjustments.

First, delete the load_dotenv() method and all of the os.getenv() methods. Instead you will store your environment variables using Repl Secrets.


Create a new secret where the key is DISCORD_TOKEN and the value is your discord API token. Create another secret where your key is OPENAI_KEY and the value is your OpenAI API key.

Then insert those into the open TOKEN and OPENAI_KEY variables at the top of the Repl.


Now you’re ready to hit the Run button at the top and you can start sending messages to your bot in Discord. The beauty of Replit is when you run the code it will install the necessary modules that you are importing!


Keeping the Bot Alive
Everything should be working swimmingly at the moment. But imagine you’re out and about bragging to your friends and you fire up Discord on your phone so you can message your Chat bot. Drum roll…you get no response.

Your bot is only going to respond as long as your Repl is running. That means closing the browser or your computer going to sleep will interrupt the connection. The best option is to create a separate thread to keep your Repl alive using Flask.

Create a new project file called keep_alive.py and add the following:

from flask import Flask
from threading import Thread

app = Flask('')

@app.route('/')
def home():
  return "I'm alive"

def run():
  app.run(host='0.0.0.0',port=8080)

def keep_alive():
  t = Thread(target=run)
  t.start()
Then back in your main.py file make sure to import this server at the top of your file and start up the web server at the bottom of your file before you start the bot.


Hit the Run button at the top again and now you are finally ready to brag to all your friends.


There you have it. In the future you can modify it to give the bot a personality or replace it with ChatGPT when the API comes out.
