# 0 to 60 with Amazon Connect

# What are we going to build?
Create a contact center that has the following functionality:
1. If a phone number has called before say "welcome back", if not, greet the caller.
2. Ask the person if they would like to be called back later to respond to a survey after the call.
3. Using their voice to choose, give them the option of waiting on hold, being called back later, or connected to the emergency line.
4. If they choose to wait, check to see if it is in the hours of operation and if anyone is available. If no one is currently available, provide the option of being put in the callback queue.
5. If they select the callback queue, place the customer in a callback queue.
6. If they are connected to the emergency line, look up who is on call and connect the customer to that person.  If they're unable to connect, reach out to someone else.
7. If the person choose to answer the survey, call the person back to answer a satisfaction survey.

## You will need a computer, headset, and phone to test your implementation

# Creating a call center in about 5 minutes

## Create the Amazon Connect instance
- Log into the console.
- Navigate to the Amazon Connect service page.
- Select Add an instance
- For identity management, choose Store users within Amazon Connect and enter a unique url to serve as an access point for your Contact center. Click Next.
  - For production applications, it may make more sense to integrate with an existing directory, but for the purposes of today, let's use the built in user management capabilities.
- For Create an Administrator page, add an admin identity.  You can use your IAM user to authenticate as an admin into the instance if you do not create a separate entity. Click Next.
- For Telephony Options, make sure both incoming and outbound calls are selected. Click Next.
- For Data storage, leave the default settings. Click Next.
- Review and Create the instance.
- Once the instance is created, select Get Started.
  
## Getting your first call
- Once you've entered the Amazon Connect application, select "Let's go".
- Claim your phone number.  Select a US (+1) Direct Dial number and click Next.
  - You can port your own US phone number to Amazon Connect if you'd like to replace existing phone numbers.
- Wait a minute or so.
- Give yourself a call! Amazon Connect comes with a number of example contact flows to demonstrate different functionalities.
- Test them out.


# Getting starting with Amazon Connect
## Let's build out a queue from scratch
### Create hours of operation
- Under Routing, select Hours of Operation.
- Select Add new hours.
- Add the name "Never Open" and description of your choosing.
- Select all of the days and remove them.  
  - Normally, we would want to set this, but we're going to demonstrate how to build logic around hours of operation.
- Save.
  
### Create your queue
- Under Routing, select Queues.
- Add new queue.
- Enter a name and description.
- Select the "Never Open" hours of operation you just created.
- Add an Outbound caller ID name.
- For the Outbound caller ID number, select the number you claimed from the drop down.
- Select Add new queue.

### Create a new Routing Profile
- Under Users, go to Routing Profiles
- Add a new routing profile
- Enter a name and description for the new Routing profile, potentially Admin Routing Profile since we'll be assigning this to your Admin user.
- Add both your new queue and the basic queue to the routing profile.  This will allow you to take calls from both queues.
- Associate the BasicQueue as the default outbound queue.
- Add new profile.

### Associate your admin user to the new Routing Profile
- Under Users, go to User Management.
- Select your Admin user and edit.
- Change the routing profile to your new profile.
- Save

### Create your contact flow
- Under Routing, select Contact Flows.
- Select create contact flow.
- Enter the name TransferToQueue.
- Under Interact, add a Play prompt module and link it to Entry point.
- Click into the module, select Text to speech and enter the text "I'm putting you in the queue".
- Under Set, add a Set working queue module and link it to the Play prompt module.
- Click into the module and select the queue you just created.
- Under Terminate/Transfer add a Transfer to Queue module and link it to the Success option of the Set working queue module.
- Add two more Play audio modules.  Make one say "Something went wrong.  Try again later" and the other "I'm sorry.  It seems our queue is at capacity.  Try again later".
- Link the error message to the Error options and the at capacity message to the At capacity options.
- Under Terminate/Transfer, add a Disconnect/Hang up module and link your final messages to it.
- Save and then publish.

### Test it out!
- Under phone numbers, select the number you've claimed.
- Under Contact flow / IVR, select the contact flow you just created and save.
- Wait a few moments and give yourself a call.

## Create a callback queue
- Under Routing, select Contact Flows.
- Select create contact flow.
- Enter the name TransferToCallbackQueue.
- Under Interact, add a Play prompt module and link it to Entry point.
- Click into the module, select Text to speech and enter the text "I'll put you in the callback queue".
- Under Set, add a Set working queue module and link it to the Play prompt module.  Click into the module and select the queue you just created.
- Under Interact, add a Store Customer Input module.  Add the text "What number should we use to call you back?".  Select Phone number under customer input and select the US Country code.  Link it to the Play prompt module.
- Under Set, add a Set callback number module and link it to the success output of Store customer input.  In the module, select System for Type and Stored customer input for Attribute.
- Under Interact, add a Play prompt module and enter the text "The number entered is invalid. Please try again."  Link this to the Invalid number and Not dialable options. Link the Okay option on the play prompt to Store customer input.
- Under Terminate/Transfer add a Transfer to Queue module.  In the module, select Transfer to callback queue, set the initial delay to 30 seconds, and click save.  Link it to the Success option of the Set callback number module.
- Add two more Play audio modules.  Make one say "Something went wrong.  Try again later" and the other "You will receive a call at your selected number shortly.  Thank you".
- Link the error message to the Error options and the call back message to the success option of Transfer to queue.
- Under Terminate/Transfer, add a Disconnect/Hang up module and link your final messages to it.
- Save and then publish.
- Test it out by changing the phone number contact flow to the one you've just created.

## Take a look at the Default outbound and Default customer queue contact flows
- These are contact flows that happen when a customer enters a queue.
- They can be changed or new ones can be created and referenced via the Set customer queue flow module.

## Create a contact flow to route incoming calls 
- Under Routing, select Contact Flows.
- Select create contact flow.
- Enter the name InboundCallRouter.
- Under Branch, select Check hours of operation.  Select Basic Hours.
- Build out the Error flow with error message and termination.
- Under Branch, select Check staffing.  Under Status to check, select Available.  Optionally select a queue.  Link this module to Check hours of operation's In Hours module. Link the error option.
- Under Terminate/Transfer, select Transfer to flow.  Select your TransferToQueue contact flow and save.  Link this to the True output of Check Staffing and the error path.
- Under Terminate/Transfer, select Transfer to flow.  Select your TransferToCallbackQueue contact flow and save.  Link this to the False output of Check Staffing and the error path.
- Add a Play prompt module for the Out of Hours option and terminate.
- Save, Publish, and Update your phone number's Contact Flow.
- Now you can test how the caller is routed when you are Available or Unavailable in the CCP.  Similarly, if you changed the hours of operation to "Never Open" and republished the contact flow, you can see how users are routed.

## Integrate a chatbot
### (Optional) Import a Chat Bot 
- Open the Amazon Lex console.
- Cancel out of the default getting started page.
- Under Actions, select Import.
- Upload the ConnectBot.zip file.
- Click into the newly created bot and select Build.
- Publish the bot as ConnectBot.


### Create a Chat Bot using Amazon Lex
- Open the Amazon Lex console
- Select Create 
- Select Custom bot
  - Bot name: ConnectBot
  - Output voice: any
  - Session timeout: 5 minutes
  - IAM role: leave default
  - COPPA: No
- Create Intents:
  - Create a WaitOnHold Intent
    - Add sample utterances for what people would say to wait on hold
      - I'll wait on hold
      - I'll wait
      - I want to wait on hold
      - I'll hold
    - Save Intent
  - Create a Callback Intent
    - Add sample utterances 
      -  Call me back
      -  Leave a callback number
      -  Call me back later
      -  call back
   -  Save Intent
-  Create an Emergency Intent
   -  Add sample utterances
      -  It's an emergency
      -  I need help
      -  Something's wrong
      -  There's an issue
      -  Help
   -  Save Intent
-  Create a YesNo Intent
   -  Create a custom Slot Type
      -  Name: YesNoSlot
      -  Slot Resolution: Restrict to Slot values and Synonyms
      -  Add the no value with synonyms: nope, no thanks, no thank you, I don't think so, I don't know
      -  Add the yes value with synonyms: yeah, sure, yes please, sure thing, why not
      -  Save and add to Intent
   -  Rename the Slot to Response
   -  Add a sample utterance {Response}
-  Create a SatisfactionSurvey Intent
   -  Create a custom Slot Type
      -  Name: SatisfactionSlot
      -  Slot Resolution: Restrict to Slot values and Synonyms
      -  Add the 5 value with synonyms: great, amazing, fantastic, wonderful, five stars
      -  Add the 4 value with synonyms: good, nice, pretty good, alright, four stars  
      -  Add the 3 value with synonyms: okay, decent, three stars
      -  Add the 2 value with synonyms: not great, pretty bad, two stars
      -  Add the 1 value with synonyms: terrible, horrible, miserable, one star
      -  Save and add to Intent
   -  Rename the Slot to Satisfaction
   -  Add a sample utterance {Satisfaction}
   -  (Optional) Add a new slot named Location with slot type AMAZON.Country with prompt "Which country do you live in?".  Make both required
-  Build the Lex Bot
-  Publish the chatbot after testing.

### Give Connect the ability to access the chatbot
- Go to the Amazon Connect console and select your instance.
- Under Contact flows, add the Lex bot you just built and published.

### Use your chatbot to let users route themselves
- Create a new contact flow called InboundLexRouter.
- Under Interact, add a Get customer input module.  Add a Text to speech prompt. "Would you like to wait on hold or be called back later when we are ready to serve you?".  Select Amazon Lex and select the bot you just created.  Add the WaitOnHold, CallBack, and Emergency Intents.
- Create an error flow.
- Under Terminate/Transfer, select Transfer to flow.  Select your InboundCallRouter contact flow and save.  Link this to the WaitOnHold output and the error path.  If you tested the "Never Open" hours of operation, maybe change that back to the basic hours.
- Under Terminate/Transfer, select Transfer to flow.  Select your TransferToCallbackQueue contact flow and save.  Link this to the Callback output and the error path.
- For now, have the Emergency and Default outputs also go to the error path.
- Save, Publish, and Test

### Build a survey contact flow using Lex
- Create a new Contact Flow called Caller Survey.
- Use the Get customer input with Lex to call your SatisfactionSurvey Intent.  Prompt the user with "How was your service today?"
- Normally you would call a Lambda function to capture this feedback, but we can simply play a prompt thanking the caller and close the call.
- Save, Publish, Test


# Building Advanced Functionality by Integrating with other AWS Services

## Launch a CloudFormation Template
- Open up the CallerSurvey contact flow and take a look at the URL.  It should look like this:
  - https://<CONNECT_INSTANCE_NAME>.awsapps.com/connect/contact-flows/edit?id=arn:aws:connect:<REGION>:<ACCOUNT_NUMBER>:instance/<INSTANCE_ID>/contact-flow/<CONTACT_FLOW_ID>
- Go to the CloudFormation service and launch a stack
  - Select Template is ready and upload the full-template.yml file in this repo
  - Name the stack and aadd the following information from your Connect instance into the parameters:
    - ConnectInstanceId = the INSTANCE_ID from the URL
    - ConnectPhoneNumber = the phone number provisioned for your contact center
    - OutboundContactFlowId = The CONTACT_FLOW_ID from the CallerSurvey contact flow
    - UserPhoneNumber = your phone number 
  - Keep the other settings as default
  - Accept the Capabilities and transforms conditions and Creat Stack
- What does this create for you?
  - ContactHistoryTable: A DynamoDB table that will let you track who calls and the last time they called.
  - ContactLookupLambda: A Lambda function that when given CustomerNumber as a parameter (when invoked from Connect) will look up the number in your table and return whether they are a new_caller or return_caller via attribute ContactStatus.
  - ContactRouterLambda: A Lambda function that will return a primary contact and escalation contact (your phone number) via attributes TargetContact and EscalationContact.
  - OutboundDialQueue: An SQS Queue that stores people queued for an automated outbound contact.
  - PutContactinQueueLambda: A Lambda function that puts a contact in the outbound dial queue when given CustomerNumber as a parameter from Connect.
  - InitiateOutboundDialLambda: A Lambda function that uses the SQS Queue to trigger an automated outbound dial.
  - Relevant IAM roles and policies

## Give Connect Access to trigger Lambda functions
- In the Amazon Connect Console, select your instance, and open Contact flows.
- Under AWS Lambda, select and add the ContactLookupLambda, ContactRouterLambda, and PutContactinQueueLambda Functions

## Create an entry point contact flow
- Return to the Amazon Connect application and create an EntryPoint contact flow.
- Under Set, add a Set logging behavior module.
- Under Integrate, add an Invoke AWS Lambda function module.  Select the ContactLookupLambda.  Add a parameter using an attribute.  Set the Destination Key to CustomerNumber, Type to System, and Attribute to Customer Number. 
- Under Branch, add a module for Check contact attributes. Set Type to External and Attribute to Contact Status.  Add two conditions.  One for when value equals "return_caller" and another for "new_caller".
- Add audio prompts to greet a return caller and another for new callers.  Route no match callers to the new caller prompt.
- Add an error flow.
- Transfer the callers to your InboundLexRouter.
- Save, Publish, Test (twice)

## Create an on call Contact Flow
- Create an OnCall contact flow.
- Add an error flow.
- Add a Lambda invocation to call the ContactRouterLambda.
- Under Set, add a Set contact attributes block.  Use attribute to create TargetContact from the External attribute TargetContact.  Add another attribute for EscalationContact.
- Under Transfer/Terminate, add a Transfer to phone number block.  Transfer to the User Defined Attribute TargetContact.
- Add prompt asking if the user was able to reach someone.  Then add a Get User Input block that uses Lex to trigger the YesNo intent.
- Use the Check contact attributes to look at the Lex Slot for the Response slot and determine whether the slot equals Yes or No.  If No, no match, call failed, or the initial call times out, play a prompt announcing a call escalation.  If yes, prompt "thank you for calling" and hang up.
- Use Transfer to phone number again, but use the EscalationContact attribute to reach the secondary person.  Change the Resume contact flow after disconnect to no.
- Connect the remaining open errors to the error flow.
- Save and publish.
- Update the InboundLexRouter's emergency option to use the new on contact flow.
- Save, Publish, and Test

## Create a Request Callback Survey Contact Flow
- Create an RequestCallbackSurvey contact flow.
- Get customer input by using the YesNo intent in the Lex bot and asking "Can we call you back later to answer a satisfaction survey?"
- Branch based on the customer input (like above) except if yes, trigger the PutContactInQueueLambda function. Add a parameter using an attribute.  Set the Destination Key to CustomerNumber, Type to System, and Attribute to Customer Number. 
- Play a prompt thanking the customer or saying okay.
- Transfer to the InboundLexRouter contact flow.
- Wire up the error flow.
- Save and Publish
- Modify the EntryPoint contact flow to transfer to this contact flow.
- Save, Publish, and Test.

## (Optional) Update the InboundCallRouter
Optionally update the inbound call router to use the Lex YesNo intent to ask if they'd rather wait on hold if no agent is available instead of automatically being placed in the callback queue.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
