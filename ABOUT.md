# The Project

## Teams

Initially, it was decided that the user would use a Team's outgoing webhook to send ticket information. This would send a POST request to the Flask server containing the ticket information. However, after configuring the webhook, there were a few issues that arose.

### Problems with Teams Outgoing Webhooks

##### Formatting and Parsing
The webhook formats the user's Teams message in HTML and sends the formatted message in the body of the request. Parsing this on the back-end was relatively simple using [BeautifulSoup4](). However, if a user uses a line break, Teams renders the new line in a seperate tag which requires further processing before parsing. Although it may have been feasible to combine the seperate tags and then parse the HTML, this would have required more effort than necessary. [Power Automate]() and [Adaptive Cards]() can be used to enforce input validation on the front-end.

##### Extra Steps
Mentioning the webhook provides an additional step in the process of a user submitting a ticket. Power Automate can be used to automate the process of creating a new submission form.

##### Teams Channel Congestion
Creating a new message for each submission can congest the Teams channel with old ticket submissions. Power Automate and Adaptive Cards can be used to provide a single form for all users to submit tickets from.

### Switching to Power Automate and Adaptive Cards

##### Eliminating Input Validation and Parsing
Adaptive Cards were used in conjuction with Power Automate to enforce input validation eliminating the need for processing and parsing the ticket submission on the Flask server.

Initially, the plan was to create a [flow]() which would send an Adaptive Card, from which users can submit tickets, when a keyword was tpyed in the appropriate channel. The flow would then wait for a response from the Adaptive Card, then send a new message in the same channel, mentioning the webhook and providing the ticket information in a standardised format, eliminating the [Formatting and Parsing](#Formatting-and-Parsing) issue. However, this was not possible, since webhooks cannot be mentioned through flow actions as of writing.

##### Using HTTP Actions
The solution was to use a HTTP action. Although this requires a Premium Power Automate account, using the HTTP action streamlines the ticket submission process. After waiting for the card response, a HTTP post request is sent from the flow to the Flask server with the user's ticket information. The flow waits for the response and replies with a message under the Adaptive Card with the response status message.

##### Streamlining Ticket Submission and Eliminating Congestion 
Using a keyword to create an Adaptive Card does not provide a solution to the [Extra Steps](#extra-steps) and [Teams Channel Congestion](#teams-channel-congestion) issues. Therefore, instead of trigerring the flow with a keyword, the flow was triggered when responding to an exsiting Adaptive Card. This required setting up another flow which can be triggered manually to create an Adaptive Card in the appropriate channel. With this solution, only one Adaptive Card exists for all users to submit with, allowing for a cleaner Teams channel devoid of old ticket submissions and one less step for the user in submitting a ticket.

##### Solutions and the Default Environment
It is important to note that an issue found during development was when the `When someone responds to an adaptive card` trigger was not activating when a user responded to the Adaptive Card. The `Card Type Id` parameter under the `When someone responds to an adaptive card` trigger and the action `Post a card in a chat or channel` (in the manually triggered flow) were identical. The solution was to make sure that the flows were created within the same [solution]() and existed under the [default environment]().

## Flask

### Creating the Route
The Flask app contains one route (`/`) which handles the submission of the ticket to the appropriate AWS SQS queue based on priority. 

##### Sorting by Priority
The logic for sorting tickets by priority was relatively simple thanks to the Adaptive Card's input standardisation; users can  donly pick between aefined list of strings (`High`, `Low`, `Medium`) which is used by the Flask server in a switch statement to send the ticket to the correct SQS queue.

##### Creating and Sending to AWS SQS Queues
The Flask server uses `boto3` to handle the API calls to AWS SQS. Initially, once the route determined the queue to submit the ticket to, the app would use the `get_queue_by_name` method to find the relevant queue and if not found it will use the `create_queue` method to create the queue before sending the message using the `send_message` method. However, the `create_queue` method will return the SQS queue if it exists already, therefore the `get_queue_by_name` was removed since it was obsolete.

### AWS Credentials
The server uses the access credentials of an AWS user with permission to read and modify SQS queues. Initially, the credentials were configured and used through the AWS CLI using `aws configure`. However, in order to simplify the process of setting up the Flask server, the credentials are retrieved as environment variables using [python-dotenv]() and a boto3 client is initialised to handle API calls.

Since the AWS CLI was configured with a default region, the boto3 client was also configured with a default region retrieved as an environment variable.

### Ngrok
The Flask server during development needed to be publicly accessible for Power Automate to send HTTP requests, therefore [ngrok]() was used as a reverse proxy.