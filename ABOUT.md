# The Project

## Teams

Initially, it was decided that the user would use a Team's outgoing webhook to send ticket information. This would send a POST request to the Flask server containing the ticket information. However, after configuring the webhook, there were a few issues that arose.

### Problems with Teams Outgoing Webhooks

#### Formatting and Parsing
The webhook formats the user's Teams message in HTML and sends the formatted message in the body of the request. Parsing this on the back-end was relatively simple using [BeautifulSoup4](). However, if a user uses a line break, Teams renders the new line in a seperate tag which requires further processing before parsing. Although it may have been feasible to combine the seperate tags and then parse the HTML, this would have required more effort than necessary. [Power Automate]() and [Adaptive Cards]() can be used to enforce input validation on the front-end.

#### Extra Steps
Mentioning the webhook provides an additional step in the process of a user submitting a ticket. Power Automate can be used to automate the process of creating a new submission form.

#### Teams Channel Congestion
Creating a new message for each submission can congest the Teams channel with old ticket submissions. Power Automate and Adaptive Cards can be used to provide a single form for all users to submit tickets from.

### Switching to Power Automate and Adaptive Cards

#### Eliminating Input Validation and Parsing
Adaptive Cards were used in conjuction with Power Automate to enforce input validation eliminating the need for processing and parsing the ticket submission on the Flask server.

Initially, the plan was to create a [flow]() which would send an adaptive card, from which users can submit tickets, when a keyword was tpyed in the appropriate channel. The flow would then wait for a response from the adaptive card, then send a new message in the same channel, mentioning the webhook and providing the ticket information in a standardised format, eliminating the [Formatting and Parsing](#Formatting-and-Parsing) issue. However, this was not possible, since webhooks cannot be mentioned through flow actions as of writing.

#### Using HTTP Actions
The solution was to use a HTTP action. Although this requires a Premium Power Automate account, using the HTTP action streamlines the ticket submission process. After waiting for the card response, a HTTP post request is sent from the flow to the Flask server with the user's ticket information. The flow waits for the response and replies with a message under the adaptive card with the response status message.

#### Streamlining Ticket Submission and Eliminating Congestion 
Using a keyword to create an adaptive card does not provide a solution to the [Extra Steps](#extra-steps) and [Teams Channel Congestion](#teams-channel-congestion) issues. Therefore, instead of trigerring the flow with a keyword, the flow was triggered when responding to an exsiting adaptive card. This required setting up another flow which can be triggered manually to create an adaptive card in the appropriate channel. With this solution, only one adaptive card exists for all users to submit with, allowing for a cleaner Teams channel devoid of old ticket submissions and one less step for the user in submitting a ticket.

#### Solutions and the Default Environment
It is important to note that an issue found during development was when the ```When someone responds to an adaptive card``` trigger was not activating when a user responded to the adaptive card. The ```Card Type Id``` parameter under the ```When someone responds to an adaptive card``` trigger and the action ```Post a card in a chat or channel``` (in the manually triggered flow) were identical. The solution was to make sure that the flows were created within the same [solution]() and existed under the [default environment]().

## Flask
