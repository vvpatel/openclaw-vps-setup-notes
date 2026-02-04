*Last updated: 2/2/2026 by Vivek Patel*
# How to Setup OpenClaw on Virtual Private Server

- **What is OpenClaw?** It's an open source, personal AI assistant that can run on your own infrastructure. You can chat with your agent through popular messaging apps and connect it to various services to act on your behalf. This includes accessing the web through a browser, writing code, having access to local files, calendar, email, email, local places, etc. 
- **Benefits of a hosted server?** Runs 24/7, more secure than running on your laptop, no disconnect from WhatsApp/Telegram when your machine sleeps. For me, the top reason was wanting to try it in a completely separate environment from my personal data given the risks.
- Disclaimer: These are personal notes, not official docs. I'm more of a "PM" than an "engineer" so this not official documentation. Security is a big risk with all this, so proceed with caution!

# Set up your server and log in

- Set up a [DigitalOcean Account](https://cloud.digitalocean.com/registrations/new). The recommended setup will cost you ~$24/mo, here are some of the benefits of [OpenClaw on Digital Ocean Marketplace](https://marketplace.digitalocean.com/apps/openclaw). 
- Create a Droplet, choose a Region close to you, select Marketplace, search for OpenClaw, choose Basic Shared CPU with at least 4GB RAM
- For Authentication, select SSH. To set up an SSH key from your Mac terminal:
``ssh-keygen -t ed25519 -f ~/.ssh/id_digitalocean -C "digitalocean-key"``
- Save your passphrase
- Copy the public key from your Mac terminal:
``cat ~/.ssh/id_digitalocean.pub``
- Paste it into the SSH popup in Digital Ocean
- Set up billing and create your Droplet
- Wait for it to get set up (~1 minute), copy the IP address at the top 
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

- From your server terminal (you can SSH in from your Mac now)
``node /opt/clawdbot/dist/entry.js onboard`` 
- Go through the quick start setup, set your Anthropic key. (I switched from the Opus model to anthropic/claude-sonnet-4-20250514 to save some tokens. The default settings seem to consume a lot of tokens)
- You can now go to the web dashboard and chat. To get there, note that the url is the IP address from what you copied down earlier. The bot may sometimes tell you to access from localhost (e.g. 127.0.0.1) but you'll need to use the IP address from your desktop browser.
- In the chat, you can ask to get started and tell it your name, timezone, the vibe you want, etc.

# Set up a Communication Channel

- From your server terminal:
``/opt/clawdbot-cli.sh channels add`` 
- Install Telegram or whatever app you'd like to use on your phone. you can use your new Google Voice number for verification. I set this up since I wanted to have a separate app dedicated for this use case instead of connecting to something I use already on my phone like WhatsApp with all my contacts.
- In Telegram, message @BotFather and follow the prompts to get your tokens to paste in once you send this command
``/newbot``
- Open your bot in Telegram and it should respond

# Set up Web Browsing

- Get a [Brave API key](https://brave.com/search/api/)
- On your server terminal:
``node /opt/clawdbot/dist/entry.js configure --section web``
- Configure on localhost, enable web browsing, paste in your key, yes to keyless web fetch
- Clawdbot expects Chrome or Chromium. Install from the server terminal:
``sudo apt-get update``
``sudo apt-get install -y chromium-browser``
- Start the browser
``node /opt/clawdbot/dist/entry.js browser start``
- On the web dashboard go to the Config > Web, switch it to on, click save, and it will restart the gateway.
- Test it in chat by asking to use brave to browse the web to look up something, it should work!

# If you want to set up Moltbook (a strange social network for AI agents of questionable value)

- In chat tell your bot to join:
``Go to [moltbook.com](https://moltbook.com), read the skill/setup instructions, and register this agent on Moltbook.``
- After the agent signs up, it should send you a claim link (a URL that lets you claim this agent as yours).
- Set up a new X account from your new Google Account
- Follow the various steps to post the message on your X account and claim your bot
- You can let your bot know once it's been set up

# Connect to Github

- Set up a [Github account](https://github.com/) with your new Google Account 
- On your server, create an SSH key, press enter twice
``ssh-keygen -t ed25519 -C "your_github_email@example.com"``
- Copy your public key
``cat ~/.ssh/id_ed25519.pub``
- Go to Github and add your new SSH key to [settings](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fkeys)
- Verify on server, type yes 
``ssh -T git@github.com``
- Tell clawdbot by going to chat and letting it know what you set up.
- You may need to help it set up a repo locally and remotely. 

# Set up Google Connections 

- (I imagine there's an easier way to do this, but this sequence worked. The trouble was that you need to auth Google in a web browser and this server doesn't have a browser with UI so you have to use your desktop)
- Login with your new Google Account 
- Go to [Google Console](https://console.cloud.google.com/welcome/new?pli=1)
- Create a new project
- Go to Enabled APIs & Services: Enable Gmail API, Calendar API, Drive, Contacts, Sheets, Docs, Places, etc. I locked down my restrictions to my server's IP.
- Go to Keys and Credentials set up OAuth Desktop app
- Download the client_secret.json file
- To copy it to the server from your Mac:
``scp -i ~/.ssh/id_digitalocean ~/Downloads/client_secret_*.json root@YOUR_SERVER_IP:~/``
- From your server terminal, get the URL for gog (to connect to google services)
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
- Now, note the port it fails when it says site can't be reached. So in this example it would be 35699 since I see http://127.0.0.1:35699/...
- On your Mac terminal replace PORT with the port from the URL, e.g. 35699 instead of 8080, and YOUR_SERVER_IP with your server's IP:
``ssh -i ~/.ssh/id_digitalocean -L 8080:localhost:8080 root@YOUR_SERVER_IP``
(That tunnel sends "Mac's 127.0.0.1:8080" → "server's localhost:8080")
- Refresh that error page in your browser at the end of the google auth flow and it should now work
- In the server terminal you may have to enter a keyring passcode.
- To verify it worked on the server:
``gog auth list``
- Restart the daemon
``systemctl --user restart clawdbot-gateway``
- I looked up how to make my auth persist longer and learned that you can go to Google Console > credentials > audience and hit publish. This will avoid new Google Cloud projects are in "Testing" mode. In this mode, Google automatically expires your login tokens every 7 days. If you don't change this, your bot will stop working every week, and you'll have to redo the whole login process.
- Hook OpenClaw to gog. I was able to do this in the chat and passed it my token by exporting it on the command line and pasting it into chat 
``gog auth tokens export vatch2025@gmail.com --out token.txt``
- Test it out! From your Telegram, you can message your bot to create a doc and email it to someone. 

# General Debugging Tips

- If you're like me, and don't spend your days on a server command line, you can make progress by opening the log file, dropping the issues into Gemini/ChatGPT/etc, and asking it to help you debug problems. It got me through a good amount of issues. However, it's not as well informed as reading the reference docs so if you get stuck reference those: https://docs.openclaw.ai/
- To watch the log from the server:
``sudo journalctl -u clawdbot -f``
- If you keep seeing issues in the log with the bot not having access to do things you are asking it to in the sandbox, take a look at configuring the sandbox permissions using the web dashboard. Have a look under "Config > Agents". To relax some permissions you can let it "rw" read/write to the workspace, and read about how "Elevated Default works".
- If you get a gateway token error, you can find the token on the server then update in the gateway settings in the dashboard
``more /home/clawdbot/.clawdbot/gateway-token.txt`` 
