# Prerequisites
- [ ] Python ^3.12
- [ ] Poetry
- [ ] AWS access keys
- [ ] Premium Power Automate account

# Installation
Clone the repository
```
git clone https://github.com/Sarmigan/aws-ticketing-system.git
```

Retrieve an AWS access key and secret key for a user with permissions to create and send messages in SQS.

Create a .env file in the root directory and populate with appropriate AWS access credentials
```
ACCESS_KEY=...
SECRET_KEY=...
```

Install [poetry](https://python-poetry.org/docs/).

Install all dependencies
```
poetry install
```

Run the Flask app
```
poetry run flask run
```

Create a new MS Teams Team for the ticketing system.
Click the + button on the Teams tab of MS Teams and select Create team.
Select From scratch > Public, then give your new team a name and description, and click Create.
Create a new channel in the team by clicking the ellipsis button, then Add channel.
Give the channel a name and select Standard access.


# Hurdles

TEAMS WEBHOOK
- Needed ngrok to test teams webhook
- Webhook request to callback service is in html format which required parsing using bs4
- Webhook adds new lines in messages to seperate 'a' tags, which required further processing of the message.

POWER AUTOMATE FLOWS
- Used Power Automate to set up a flow to post an adaptive card in channel when keyword is used.
- Power Automate flow