# Prerequisites
- [ ] Python ^3.12
- [ ] Poetry
- [ ] AWS access keys
- [ ] Premium Power Automate account

# Installation

## Flask Server
1. Clone the repository
```
git clone https://github.com/Sarmigan/aws-ticketing-system.git
```

2. Retrieve an AWS access key and secret key for a user with permissions to create and send messages in SQS.

3. Create a .env file in the root directory and populate with appropriate AWS access credentials
```
ACCESS_KEY=...
SECRET_KEY=...
```

### Install dependencies
#### ... with poetry
1. Install [poetry](https://python-poetry.org/docs/).

2. Install all dependencies
```
poetry install
```

3. Run the Flask app
```
poetry run flask run
```

#### ... with pip
1. Install all dependencies
```
pip install -r requirements.txt
```

2. Run the Flask app
```
flask run
```

## Configure Power Automate Flows

1. Visit [Power Automate](https://make.powerautomate.com/) and login.

> [!IMPORTANT]
> Make sure you are in the [default environment](https://learn.microsoft.com/en-us/power-platform/admin/environments-overview#the-default-environment).

2. Create a new [solution](https://learn.microsoft.com/en-us/power-automate/overview-solution-flows).

### Configure "Create Adaptive Card" Flow

1. Within the new solution create a new instant cloud flow.

2. Create a "Manually trigger a flow" trigger

3. Add a Teams "Post a card in a chat or channel" action.

|  Post As  |  Flow bot  |
|  Post In  |  Channel  |
|  Team  |  [Target team]  |
|  Channel  |  [Target channel]  |
|  Adaptive Card  |  [Copy template from `card_templates/ticket_card.json`]  |

# Hurdles

TEAMS WEBHOOK
- Needed ngrok to test teams webhook
- Webhook request to callback service is in html format which required parsing using bs4
- Webhook adds new lines in messages to seperate 'a' tags, which required further processing of the message.

POWER AUTOMATE FLOWS
- Used Power Automate to set up a flow to post an adaptive card in channel when keyword is used.
- Power Automate flow