*Last updated: 2/2/2026 by Vivek Patel*
# How to Setup OpenClaw on Virtual Private Server

- **What is OpenClaw?** A self-hosted personal AI agent that **acts** for you by being able to access a browser, files, calendar, email, and can communicate with you through a messaging app like WhatsApp, Telegram, Slack, Discord.
- **Why hosted?** Runs 24/7, more secure than running on your laptop, no disconnect from WhatsApp/Telegram when your machine sleeps. For my setup, I also started with setting up fresh accounts for services to explore how it works while keeping it isolated from my personal data. 

# Set up your server and log in

- Sign in or sign up for a [Digital Ocean Account](https://cloud.digitalocean.com/registrations/new). For the recommended setup, it will cost you ~$24/mo. To learn more about why hosting here made sense to me: [OpenClaw on Digital Ocean Marketplace](https://marketplace.digitalocean.com/apps/openclaw). 
- Create a Droplet, choose a Region close to you, select Marketplace, search for OpenClaw, choose Basic Shared CPU with at least 4GB RAM
- For Authentication, select SSH. To set up an SSH key from your Mac terminal:
``ssh-keygen -t ed25519 -f ~/.ssh/id_digitalocean -C "digitalocean-key"``
- Set a passphrase and remember it as you'll use it again
- Get the public key from your Mac terminal:
``cat ~/.ssh/id_digitalocean.pub``
- Copy the output and paste it into the SSH popup in Digital Ocean
- Set up billing and create your Droplet
- Wait for it to get set up (~1 minute), click it, and copy the IP address next to ipv4 
- SSH to your server from your Mac terminal:
``ssh -i ~/.ssh/id_digitalocean root@your_droplet_ip`` (paste in the IP address above for your_droplet_ip)
- You should now be set up with OpenClaw running on your server!
- When you log in, you can see your dashboard URL in the text from OpenClaw, copy down that URL so you can access it later.

# Set Up Fresh Accounts for Google/Anthropic 

- Create a new Google Account
- Create a new Google Voice account
- Create a new Anthropic Account with your new Google Account
- Copy your Anthropic API Key from [Anthropic Console](https://console.anthropic.com) and fund the account with some credit

# Getting Started with OpenClaw 

- From your server command line run (you can click console from your droplet page or SSH to your server):
``node /opt/clawdbot/dist/entry.js onboard`` 
- Go through the quick start setup, set your Anthropic key. (I switched from the Opus model to anthropic/claude-sonnet-4-20250514 to save some tokens as I got things going)
- You can now go to the web dashboard and chat. To get there, note that the url is the IP address from what you copied down earlier. The bot may sometimes tell you to access from localhost (e.g. 127.0.0.1) but you'll need to use the IP address from your desktop browser.
- In the chat, you can ask to get started and tell it your name, timezone, the vibe you want, etc.

# Set up a Communication Channel
- From your server terminal:
``/opt/clawdbot-cli.sh channels add`` 
- Install Telegram or whatever app you'd like to use on your phone, use your new Google Voice number for verification. I set this up since I wanted to have a separate app dedicated for this use case instead of connecting to something I use already on my phone like WhatsApp with all my contacts.
- In Telegram, message @BotFather and follow the prompts to get your tokens to paste in once you send this command
``/newbot``
- Open your bot in Telegram and it should respond

# Set up Web Browsing
- Get a [Brave API key](https://brave.com/search/api/)
- On your server terminal:
``node /opt/clawdbot/dist/entry.js configure --section web``
- Configure on local host, enable web browsing, paste in your key, yes to keyless web fetch
- Clawdbot expects Chrome or Chromium. Install from the server terminal:
``sudo apt-get update``
``sudo apt-get install -y chromium-browser``
- Start the browser via Clawbot
``node /opt/clawdbot/dist/entry.js browser start``
- On the web dashboard go to the Config > Web, switch it to on, click save, and it will restart the gateway. (I'm not sure why this wasn't set correctly from the earlier commands, but it did the trick)
- Test it in chat by asking to use brave to browse the web to look up something, it should work!

# Set up Google Connections (I got oAuth with Google working a few times, but it seemed to die. Just the regular Google API key was fine)
- Login with your new Google Account 
- Go to [Google Console](https://console.cloud.google.com/welcome/new?pli=1)
- Create a new project
- Go to Enabled APIs & Services: Enable Gmail API, Calendar API, Drive, Contacts, Sheets, Docs, Places, etc.   
- Go to Keys and Credentials set up OAuth Desktop app
- Download the client_secret.json file
- To copy it to the server from your Mac:
``scp -i ~/.ssh/id_digitalocean ~/Downloads/client_secret_*.json root@YOUR_SERVER_IP:~/``
- [ ] From your server console, get the URL for gog (to connect to google services)
``curl -s https://api.github.com/repos/steipete/gogcli/releases/latest | grep "browser_download_url"``
- Download gog from github onto your server using the url you just found and install it
``cd /tmp``
``curl -sL "https://github.com/steipete/gogcli/releases/download/v0.9.0/gog_Linux_x86_64.tar.gz" -o gog.tar.gz``
``tar -tzf gog.tar.gz``
``sudo tar -xzf gog.tar.gz -C /usr/local/bin``
``sudo chmod +x /usr/local/bin/gog``
- Point gog at the JSON "credentials":
``gog auth credentials ~/client_secret_.....``
- Stop background service so it doesn't interfere
``systemctl --user stop clawdbot-gateway``
- From your server:
``gog auth add your@gmail.com``
- If gog prints a URL like https://accounts.google.com/..., open that URL on your Mac in your browser, continue, give it permissions.
- [ ] Now, note the port it fails when it says site can't be reached. So in this example it would be 35699 since I see http://127.0.0.1:35699/...
- [ ] On your Mac terminal replace PORT with the port from the URL, e.g. 35699 instead of 8080, and YOUR_SERVER_IP with your server's IP:
``ssh -i ~/.ssh/id_digitalocean -L 8080:localhost:8080 root@YOUR_SERVER_IP``
(That tunnel sends "Mac's 127.0.0.1:8080" → "server's localhost:8080")
- Refresh that error page in your browser at the end of the google auth flow and it should now work
- In the server terminal you may have to enter a keyring passcode, don't create one
- To verify it worked on the server console:
``gog auth list``
- Restart the daemon
``systemctl --user restart clawdbot-gateway``
- To make the auth persist longer, go to google console, credentials audience and hit publish. This will avoid new Google Cloud projects are in "Testing" mode. In this mode, Google automatically expires your login tokens every 7 days. If you don't change this, your bot will stop working every Sunday, and you'll have to redo the whole login process.
- Hook OpenClaw to gog. I was able to do this in the chat and passed it my token by exporting it on the command line and pasting it into chat
``gog auth tokens export vatch2025@gmail.com --out token.txt``
- Test it out! From your Telegram, you can message your bot to create a doc and email it to someone, pretty cool!

# Set up Moltbook (a social network for AI agents)
- In chat tell your bot to join:
``Go to [moltbook.com](https://moltbook.com), read the skill/setup instructions, and register this agent on Moltbook.``
- After the agent signs up, it should send you a claim link (a URL that lets you claim this agent as yours).
- Set up a new X account from your new Google Account
- Follow the various steps to post the message on your X account and claim your bot
- You can let your bot know once it's been set up

# General debugging
- If you're like me, and don't spend your days on a server command line, you can make progress by opening the log file, dopping it into Gemini/whatever, and asking it to help you debug problems. It got me through a good amount of issues.
- To watch the log from the server terminal:
``sudo journalctl -u clawdbot -f``


# Some other commands

Doctor checkup
``/opt/clawdbot/extensions/memory-core/node_modules/.bin/clawdbot doctor``

Check status
``/opt/clawdbot/extensions/memory-core/node_modules/.bin/clawdbot status``

Restart gateway
``/opt/clawdbot/dist/entry.js gateway restart``

# Some errors I ran into
- If you get a gateway token error try to find the token on the server than update in the gateway settings in the dashboard
``more /home/clawdbot/.clawdbot/gateway-token.txt`` 

