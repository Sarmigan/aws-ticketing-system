# Prerequisites
- [ ] Python ^3.12
- [ ] Poetry
- [ ] AWS access keys
- [ ] Premium Power Automate account

# Installation

Create a .env file in the root directory with the following
```
ACCESS_KEY=...
SECRET_KEY=...
```

# Hurdles

TEAMS WEBHOOK
- Needed ngrok to test teams webhook
- Webhook request to callback service is in html format which required parsing using bs4
- Webhook adds new lines in messages to seperate 'a' tags, which required further processing of the message.

POWER AUTOMATE FLOWS
- Used Power Automate to set up a flow to post an adaptive card in channel when keyword is used.
- Power Automate flow