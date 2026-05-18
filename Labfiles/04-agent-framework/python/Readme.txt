Develop an Azure AI chat agent with the Microsoft Agent Framework SDK
In this exercise, you'll use Azure AI Agent Service and Microsoft Agent Framework to create an AI agent that processes expense claims.

This exercise should take approximately 30 minutes to complete.

Note: Some of the technologies used in this exercise are in preview or in active development. You may experience some unexpected behavior, warnings, or errors.

Deploy a model in a Microsoft Foundry project
Let's start by creating a Foundry project.

In a web browser, open the Foundry portal at https://ai.azure.com and sign in using the username User1-61898375@LODSPRODMCA.onmicrosoft.com and the TAP xPyMY3WD. Close any tips or quick start panes that are opened the first time you sign in, and if necessary use the Foundry logo at the top left to navigate to the home page, which looks similar to the following image (close the Help pane if it's open):

Screenshot of Foundry portal.

Important: For this lab, you're using the New Foundry experience.

In the top banner, select Start building to try the new Microsoft Foundry Experience.

When prompted, create a new project, and enter Project61898375.

Expand Advanced options and specify the following settings:

Foundry resource: Project61898375-resource
Subscription: Your Azure subscription
Resource group: ResourceGroup1
Region: Select any AI Foundry recommended
* Some Azure AI resources are constrained by regional model quotas. In the event of a quota limit being exceeded later in the exercise, there's a possibility you may need to create another resource in a different region.

Select Create and wait for your project to be created.

After your project is created, select Build from the navigation bar.

Select Models from the left-hand menu, and then select Deploy a base model.

Enter gpt-4.1 in the search box, and then select the gpt-4.1 model from the search results.

Select Deploy with the default settings to create a deployment of the model.

After the model is deployed, the playground for the model is displayed.

In the navigation bar on the left, select Microsoft Foundry to return to the Foundry home page.

Copy the Project endpoint value to a notepad, as you'll use them to connect to your project in a client application.

Create an agent client app
Now you're ready to create a client app that defines an agent and a custom function. Some code has been provided for you in a GitHub repository.

Prepare the environment
Open a new browser tab (keeping the Foundry portal open in the existing tab). Then in the new tab, browse to the Azure portal at https://portal.azure.com; signing in with the username User1-61898375@LODSPRODMCA.onmicrosoft.com and the TAP xPyMY3WD if prompted.

Close any welcome notifications to see the Azure portal home page.

Use the [>_] button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a PowerShell environment with no storage in your subscription.

The cloud shell provides a command-line interface in a pane at the bottom of the Azure portal. You can resize or maximize this pane to make it easier to work in.

Note: If you have previously created a cloud shell that uses a Bash environment, switch it to PowerShell.

In the cloud shell toolbar, in the Settings menu, select Go to Classic version (this is required to use the code editor).

Ensure you've switched to the classic version of the cloud shell before continuing.

In the cloud shell pane, enter the following commands to clone the GitHub repo containing the code files for this exercise (type the command, or copy it to the clipboard and then right-click in the command line and paste as plain text):

TypeCopy
rm -r ai-agents -f
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
Tip: As you enter commands into the cloudshell, the output may take up a large amount of the screen buffer and the cursor on the current line may be obscured. You can clear the screen by entering the cls command to make it easier to focus on each task.

When the repo has been cloned, enter the following command to change the working directory to the folder containing the code files and list them all.

TypeCopy
cd ai-agents/Labfiles/04-agent-framework/python
ls -a -l
The provided files include application code a file for configuration settings, and a file containing expenses data.

Configure the application settings
In the cloud shell command-line pane, enter the following command to install the libraries you'll use:

TypeCopy
python -m venv labenv
./labenv/bin/Activate.ps1
pip install agent-framework==1.0.0b260212 opentelemetry-semantic-conventions-ai==0.4.13 --pre
Enter the following command to edit the configuration file that has been provided:

TypeCopy
code .env
The file is opened in a code editor.

In the code file, replace the your_project_endpoint placeholder with the endpoint for your project (copied from the project Overview page in the Foundry portal) and ensure that the AZURE_AI_MODEL_DEPLOYMENT_NAME variable is set to your model deployment name (which should be gpt-4.1).

After you've replaced the placeholders, use the CTRL+S command to save your changes and then use the CTRL+Q command to close the code editor while keeping the cloud shell command line open.

Write code for an agent app
Tip: As you add code, be sure to maintain the correct indentation. Use the existing comments as a guide, entering the new code at the same level of indentation.

Enter the following command to edit the agent code file that has been provided:

TypeCopy
code agent-framework.py
Review the code in the file. It contains:

Some import statements to add references to commonly used namespaces
A main function that loads a file containing expenses data, asks the user for instructions, and then calls…
A process_expenses_data function in which the code to create and use your agent must be added
At the top of the file, after the existing import statement, find the comment Add references, and add the following code to reference the namespaces in the libraries you'll need to implement your agent:

python
TypeCopy
# Add references
from agent_framework import tool, Agent
from agent_framework.azure import AzureOpenAIResponsesClient
from azure.identity.aio import AzureCliCredential
from pydantic import Field
from typing import Annotated
Near the bottom of the file, find the comment Create a tool function for the email functionality, and add the following code to define a function that your agent will use to send email (tools are a way to add custom functionality to agents)

python
TypeCopy
# Create a tool function for the email functionality
@tool(approval_mode="never_require")
def submit_claim(
to: Annotated[str, Field(description="Who to send the email to")],
subject: Annotated[str, Field(description="The subject of the email.")],
body: Annotated[str, Field(description="The text body of the email.")]):
   print("\nTo:", to)
   print("Subject:", subject)
   print(body, "\n")
Note: The function simulates sending an email by printing it to the console. In a real application, you'd use an SMTP service or similar to actually send the email!

Back up above the send_email code, in the process_expenses_data function, find the comment Create a client and initialize an agent with the tool and instructions, and add the following code:

(Be sure to maintain the indentation level)

python
TypeCopy
# Create a client and initialize an agent with the tool and instructions
async with (
    AzureCliCredential() as credential,
    Agent(
        client=AzureOpenAIResponsesClient(
            credential=credential,
            deployment_name=os.getenv("MODEL_DEPLOYMENT_NAME"),
            project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        ),
        instructions="""You are an AI assistant for expense claim submission.
                    At the user's request, create an expense claim and use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                    Then confirm to the user that you've done so. Don't ask for any more information from the user, just use the data provided to create the email.""",
        tools=[submit_claim],
    ) as agent,
):
Note that the AzureCliCredential object will allow your code to authenticate to your Azure account. The AzureOpenAIResponsesClient object includes the Foundry project settings from the .env configuration. The Agent object is initialized with the client, instructions for the agent, and the tool function you defined to send emails.

Find the comment Use the agent to process the expenses data, and add the following code to create a thread for your agent to run on, and then invoke it with a chat message.

(Be sure to maintain the indentation level):

python
TypeCopy
# Use the agent to process the expenses data
try:
   # Add the input prompt to a list of messages to be submitted
   prompt_messages = [f"{prompt}: {expenses_data}"]
   # Invoke the agent for the specified thread with the messages
   response = await agent.run(prompt_messages)
   # Display the response
   print(f"\n# Agent:\n{response}")
except Exception as e:
   # Something went wrong
   print (e)
Review that the completed code for your agent, using the comments to help you understand what each block of code does, and then save your code changes (CTRL+S).

Keep the code editor open in case you need to correct any typo's in the code, but resize the panes so you can see more of the command line console.

Sign into Azure and run the app
In the cloud shell command-line pane beneath the code editor, enter the following command to sign into Azure.

TypeCopy
az login
You must sign into Azure - even though the cloud shell session is already authenticated.

Note: In most scenarios, just using az login will be sufficient. However, if you have subscriptions in multiple tenants, you may need to specify the tenant by using the --tenant parameter. See Sign into Azure interactively using the Azure CLI for details.

When prompted, follow the instructions to open the sign-in page in a new tab and enter the authentication code provided and the username User1-61898375@LODSPRODMCA.onmicrosoft.com and the TAP xPyMY3WD. Then complete the sign in process in the command line, selecting the subscription containing your Foundry hub if prompted.

After you have signed in, enter the following command to run the application:

TypeCopy
python agent-framework.py
The application runs using the credentials for your authenticated Azure session to connect to your project and create and run the agent.

When asked what to do with the expenses data, enter the following prompt:

TypeCopy
Submit an expense claim
When the application has finished, review the output. The agent should have composed an email for an expenses claim based on the data that was provided.

Tip: If the app fails because the rate limit is exceeded. Wait a few seconds and try again. If there is insufficient quota available in your subscription, the model may not be able to respond.

Summary
In this exercise, you used the Microsoft Agent Framework SDK to create an agent with a custom tool.

Clean up
If you've finished exploring Azure AI Agent Service, you should delete the resources you have created in this exercise to avoid incurring unnecessary Azure costs.

Return to the browser tab containing the Azure portal (or re-open the Azure portal at https://portal.azure.com in a new browser tab) and view the contents of the resource group where you deployed the resources used in this exercise.
On the toolbar, select Delete resource group.
Enter the resource group name and confirm that you want to delete it.
Congratulations
You have successfully completed this lab. Click End to mark the lab as Complete.

