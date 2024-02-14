# Prerequisites
- [ ] Python ^3.12
- [ ] Poetry
- [ ] AWS access keys
- [ ] Premium Power Automate account

# Installation

## Flask Server
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

### Install dependencies with poetry
Install [poetry](https://python-poetry.org/docs/).

Install all dependencies
```
poetry install
```

Run the Flask app
```
poetry run flask run
```

### Install dependencies with pip
Install all dependencies
```
pip install -r requirements.txt
```

Run the Flask app
```
flask run
```

## Configure Power Automate Flows

Visit [Power Automate](https://make.powerautomate.com/) and login.

> [!IMPORTANT]
> Make sure you are in the default environment.

Create a new solution.

Within the new solution create a new instant cloud flow.



# Hurdles

TEAMS WEBHOOK
- Needed ngrok to test teams webhook
- Webhook request to callback service is in html format which required parsing using bs4
- Webhook adds new lines in messages to seperate 'a' tags, which required further processing of the message.

POWER AUTOMATE FLOWS
- Used Power Automate to set up a flow to post an adaptive card in channel when keyword is used.
- Power Automate flow