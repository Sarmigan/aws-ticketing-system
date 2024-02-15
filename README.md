
<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
# Teams Ticketing

</div>

This is a bug ticketing system designed for Teams. Users can choose a priority level and provide a bug description using an [Adaptive Card](https://learn.microsoft.com/en-us/adaptive-cards/) within a Teams channel. Using Power Automate, the ticket is then dispatched to a Flask app, which directs the message to the appropriate AWS SQS queue for further processing.

# Prerequisites
- [ ] Python ^3.12
- [ ] Poetry/pip
- [ ] [AWS access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
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
REGION=<AWS region>
ACCESS_KEY=<AWS access key>
SECRET_KEY=<AWS secret key>
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

> [!IMPORTANT]
> The Flask server must have a publicly accessible URI. Consider using [ngrok](https://ngrok.com/) if hosting locally.

## Configure Power Automate Flows

1. Visit [Power Automate](https://make.powerautomate.com/) and login.

> [!IMPORTANT]
> Make sure you are in the [default environment](https://learn.microsoft.com/en-us/power-platform/admin/environments-overview#the-default-environment).

2. Create a new [solution](https://learn.microsoft.com/en-us/power-automate/overview-solution-flows).

### Configure 'Create Adaptive Card' Flow

1. Within the new solution create a new instant cloud flow.

2. Add a 'Manually trigger a flow' trigger

3. Add a Teams 'Post a card in a chat or channel' action.

<table>
<tr>
<td> Post As </td><td> Flow bot </td>
</tr>
<tr>
<td> Post In </td><td> Channel </td>
</tr>
<tr>
<td> Team </td><td> &lt;Target team&gt; </td>
</tr>
<tr>
<td> Channel </td><td> &lt;Target channel&gt; </td>
</tr>
<tr>
<td> Channel </td><td> &lt;Target channel&gt; </td>
</tr>
<tr>
<td> Adaptive Card </td><td> &lt;Copy template from card_templates/ticket_card.json&gt; </td>
</tr>
<tr>
<td> Card Type Id </td><td> TicketCard </td>
</tr>
</table>

> [!NOTE]
> The field 'Card Type Id' can be found under 'Advanced parameters'. 'Card Type Id' can be any value but must stay consistent across flows

4. Save and exit flow.

5. Trigger and run the flow.

### Configure 'Submit Ticket' Flow

1. Within the new solution create a new instant cloud flow.

2. Add a Teams 'When someone responds to an adaptive card' trigger

<table>
<tr>
<td> Inputs Adaptive Card </td><td> &lt;Copy template from card_templates/ticket_card.json&gt; </td>
</tr>
<tr>
<td> Card Type Id </td><td> TicketCard </td>
</tr>
</table>

3. Add a Teams 'Post card in chat or channel' action

<table>
<tr>
<td> Post as </td><td> Flow bot </td>
</tr>
<tr>
<td> Post in </td><td> Chat with Flow bot </td>
</tr>
<tr>
<td> Recipient </td><td> &lt;Responder User ID&gt; </td>
</tr>
<tr>
<td> Adaptive Card </td><td> &lt;Copy template from card_templates/ticket_update.json&gt; </td>
</tr>
</table>

> [!IMPORTANT]
> Replace the 'text' values in the 'update-priority-message' and 'update-description-message' TextBlocks in the ticket_update.json with the 'input-priority' and 'input-description' dynamic content. Replace the 'text' value in the 'update-heading' TextBlock in the ticket_update.json with a message for the user i.e. "Your ticket is being processed!"

> [!NOTE]
> 'Responder User ID' can be found under dynamic content from the 'When someone responds to an adaptive card' trigger.

> [!NOTE]
> 'input-priority' and 'input-description' can be found under dynamic content from the 'When someone responds to an adaptive card' trigger.

4. Add a HTTP 'HTTP' action

<table>
<tr>
<td> Method </td><td> POST </td>
</tr>
<tr>
<td> URI </td><td> &lt;Flask server URI&gt; </td>
</tr>
<tr>
<td> Body </td><td>
<pre lang="json">
{
    "priority": "&lt;input-priority&gt;",
    "description": "&lt;input-description&gt;"
}
</pre>
</td>
</tr>
</table>

> [!IMPORTANT]
> You will need a premium Power Automate account to be able to use the HTTP actions!

5. Add a Teams 'Post card in chat or channel' action

<table>
<tr>
<td> Post as </td><td> Flow bot </td>
</tr>
<tr>
<td> Post in </td><td> Chat with Flow bot </td>
</tr>
<tr>
<td> Recipient </td><td> &lt;Responder User ID&gt; </td>
</tr>
<tr>
<td> Adaptive Card </td><td> &lt;Copy template from card_templates/ticket_update.json&gt; </td>
</tr>
</table>

> [!IMPORTANT]
> Replace the 'text' values in the 'update-heading', 'update-priority-message' and 'update-description-message' TextBlocks in the ticket_update.json with the 'Body', 'input-priority' and 'input-description' dynamic content. 

> [!NOTE]
> 'Body' can be found under dynamic content from the 'HTTP' action.

6. Save and exit flow.

# Usage

The Adaptive Card under &lt;Target team&gt; in &lt;Target channel&gt; can be used to submit a ticket. The user will receive a 'processing' Adaptive Card sent to them followed by a succesful/unsucessful 'response' Adaptive Card. These Adaptive Cards will contain a summary of the ticket information i.e. priority and description. If everything is configured correctly, the tickets can be found in the relevant SQS queues.