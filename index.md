# How to Setup OpenClaw on Virtual Private Server 

- **What is OpenClaw?** A personal AI agent that **acts** for you by being able to access a browser, files, messages (WhatsApp, Telegram, Slack, Discord).
- **Why hosted?** Runs 24/7, more secure than running on your laptop, no disconnect from WhatsApp/Telegram when your machine sleeps. For my setup, I also started with setting up fresh accounts for services to explore how it works while keeping it isolated from my personal data. 

# Set up your server and log in:

- [ ] Sign in or sign up for a Digital Ocean account (https://cloud.digitalocean.com/registrations/new). For the recommended setup, it will cost you ~$24/mo FYI. To learn more about some of the benefits: https://marketplace.digitalocean.com/apps/openclaw. 
- [ ] Create a Droplet, choose a Region close to you, select Marketplace, search for OpenClaw, choose Basic Shared CPU with at least 4GB RAM
- [ ] For Authentication, select SSH. To set up an SSH key from your Mac terminal:
``bash ssh-keygen -t ed25519 -f ~/.ssh/id_digitalocean -C "digitalocean-key"```
- [ ] Set a passphrase and remember it as you'll use it again
- [ ] Get the public key from your Mac terminal:
``cat ~/.ssh/id_digitalocean.pub```
- [ ] Copy the output and paste it into the SSH popup in Digital Ocean
- [ ] Set up billing and create your Droplet
- [ ] Wait for it to get set up (~1 minute), click it, and copy the IP address next to ipv4 
- [ ] SSH to your server from your Mac terminal:
ssh -i ~/.ssh/id_digitalocean root@your_droplet_ip (paste in the IP address above for your_droplet_ip)

You should now be set up with OpenClaw running on your server!
- [ ] When you log in, you should see your dashboard URL in the text from OpenClaw, copy down that URL so you can access it later.

# Set Up Fresh Accounts for Google/Anthropic 

- [ ] Create a new Google Account
- [ ] Create a new Google Voice account
- [ ] Create a new Anthropic Account with your new Google Account
- [ ] Copy your Anthropic API Key (https://console.anthropic.com) and fund the account with some credit

# Getting Started with OpenClaw 

- [ ] From your server run (you can click console from your droplet page or SSH from your Mac):
node /opt/clawdbot/dist/entry.js onboard 
(If you're in clawd in the console, you can always Ctrl+C once or twice to get out of Clawdbot and back to the server terminal)
- [ ] Go through the quick start setup, set your Anthropic key. (I switched from the claude opus model to anthropic/claude-sonnet-4-20250514 to save some tokens)
- [ ] You can now go to the web dashboard and chat. To get there, note that the url is the IP address from what you copied down earlier. The bot may tell you to access from localhost (e.g. 127.0.0.1) because it doesn't realize it's running on a server.
- [ ] In the chat, you can ask to get started and tell it your name, timezone, the vibe you want, etc.

# Part C: Setting up services

# To set up a Channel
- [ ] From your server console:
/opt/clawdbot-cli.sh channels add 
- [ ] Install Telegram app on your phone, use your new Google Voice number for verification. (I wanted to have something separate than my personal WhatsApp account with all my contacts for more control than just connecting to my existing WhatsApp)
- [ ] In Telegram message @BotFather and follow the prompts to get your tokens to paste in
/newbot
- [ ] Open your bot in Telegram and see if it responds

# Set up Web Browsing
- [ ] Get a Brave API key (https://brave.com/search/api/)
- [ ] On your server console:
node /opt/clawdbot/dist/entry.js configure --section web
- [ ] Configure on local host, enable web browsing, paste in your key, yes to keyless web fetch
- [ ] Clawdbot expects Chrome or Chromium. Install from the server:
sudo apt-get update
sudo apt-get install -y chromium-browser
- [ ] Start the browser via Clawbot
node /opt/clawdbot/dist/entry.js browser start
- [ ] On the web dashboard go to Config then Web, switch it to on, click save, and it will restart the gateway
- [ ] Test it in chat, it should work!

# Set up Google Connection
- [ ] Go to Google Console (https://console.cloud.google.com/welcome/new?pli=1)
- [ ] Create a new project                                                                                                  
- [ ] Go to Enabled APIs & Services: Enable Gmail API, Calendar API, Drive, Contacts, Sheets, Docs, Places, etc.   
- [ ] Go to Keys and Credentials set up OAuth Desktop app
- [ ] Download the client_secret.json file
- [ ] To copy it to the server from your Mac:
scp -i ~/.ssh/id_digitalocean ~/Downloads/client_secret_*.json root@YOUR_SERVER_IP:~/
(This next part seems overly complicated but I couldn't get it to work from the web dashboard so here's the command line way...)
- [ ] From your server console, get the URL for gog (to connect to google services)
curl -s https://api.github.com/repos/steipete/gogcli/releases/latest | grep "browser_download_url"
- [ ] Download gog from github onto your server using the url you just found and install it
cd /tmp
curl -sL "https://github.com/steipete/gogcli/releases/download/v0.9.0/gog_Linux_x86_64.tar.gz" -o gog.tar.gz
tar -tzf gog.tar.gz
sudo tar -xzf gog.tar.gz -C /usr/local/bin
sudo chmod +x /usr/local/bin/gog
- [ ] Point gog at the JSON and test "credentials":
gog auth credentials ~/client_secret_.....
- [ ] From your Mac, set up port forwarding, replace your server IP:
ssh -i ~/.ssh/id_digitalocean -L 8080:localhost:8080 root@YOUR_SERVER_IP
- [ ] From your server:
gog auth add your@gmail.com
- [ ] If gog prints a URL like https://accounts.google.com/..., open that URL on your Mac in your browser
- [ ] Note the port it fails on (e.g. http://127.0.0.1:35699/)
- [ ] On your Mac terminal replace PORT with the port from the URL, e.g. 8080, and YOUR_SERVER_IP with your server's IP:
ssh -i ~/.ssh/id_digitalocean -L 8080:localhost:8080 root@YOUR_SERVER_IP
(That tunnel sends "Mac's 127.0.0.1:8080" → "server's localhost:8080")
- [ ] Refresh that error page in your browser at the end of the google auth flow and it should now work
- [ ] In the terminal you may have to enter a keyring passcode, create one and save it for reference
- [ ] To verify it worked on the server console:
gog auth list
gog calendar calendars --json
- [ ] Hook OpenClaw to gog. I was able to do this in the chat and passed it my token by exporting it on the command line and pasting it into chat
gog auth tokens export vatch2025@gmail.com --out token.txt
(Like I said, feel free to comment and let me know if you found a cleaner way to do this, but this worked :)
- [ ] Test it out! From your Telegram, you can message your bot to create a doc and email it to someone, pretty cool!

# Set up Moltbook (a social network for AI agents)
- [ ] In chat tell your bot to join:
Go to moltbook.com, read the skill/setup instructions, and register this agent on Moltbook.
- [ ] After the agent signs up, it should send you a claim link (a URL that lets you claim this agent as yours).
- [ ] Set up a new X account
- [ ] Follow the various steps to post the message on your X account and claim your bot
- [ ] You can let your bot know once it's been set up

# Some generally useful commands
Restart gateway: node /opt/clawdbot/dist/entry.js gateway restart

# Dead-ends to avoid
