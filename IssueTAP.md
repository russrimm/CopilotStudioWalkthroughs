# Setting up a Microsoft Copilot Studio Agent for Temporary Access Pass Issuance

## Overview
This document provides step-by-step instructions for creating a Copilot Studio agent that helps users who are locked out of their accounts due to MFA issues (typically after getting a new device) by issuing a Temporary Access Pass after identity verification.

## Prerequisites
- Administrative access to Microsoft Entra ID (formerly Azure AD)
- Access to Microsoft Copilot Studio
- Access to Power Automate
- Access to Dataverse
- Access to Azure Key Vault
- Access to create, grant permissions to, and consent to an Entra ID Application (Service Principal) - Application Permissions (UserAuthenticationMethod.ReadWrite.All) to Graph.

## Step 1: Create Microsoft Entra App Registration

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com)
2. Navigate to "App registrations" under "Applications"
3. Click "New registration"
4. Enter a name for your application (e.g., "Temporary Access Pass Issuer")
5. Select the appropriate supported account types (typically "Accounts in this organizational directory only")
6. Click "Register"
7. Record the following information for later use:
   - Application (Client) ID
   - Directory (Tenant) ID
8. Navigate to "Certificates & secrets" under the app's menu
9. Click "New client secret"
10. Enter a description and select an expiration period
11. Click "Add"
12. Record the client secret value immediately (it will not be visible again)

## Step 2: Grant API Permissions to the App

1. Under the app registration, navigate to "API permissions"
2. Click "Add a permission"
3. Select "Microsoft Graph"
4. Choose "Application permissions"
5. Search for and select "UserAuthenticationMethod.ReadWrite.All"
6. Click "Add permissions"
7. Click "Grant admin consent for [your tenant]" and confirm

## Step 3: Create an Azure Key Vault and Add Secrets

1. In the Azure portal, search for and select "Key vaults"
2. Click "Create"
3. Fill in the required details:
   - Select your subscription and resource group
   - Enter a unique Key vault name
   - Choose your region
4. Review and create the Key vault
5. Once deployed, navigate to the Key vault
6. Go to "Secrets" in the left menu
7. Click "Generate/Import"
8. Create the following secrets:
   - Name: "ClientSecret" with the value of your app's client secret
   - Name: "ClientID" with the value of your app's Application (Client) ID
   - Name: "TenantID" with the value of your Directory (Tenant) ID

## Step 4: Create a Dataverse Table for User Verification

1. Navigate to the Power Apps portal
2. Select your environment
3. Go to "Tables" in the left menu
4. Click "New table"
5. Configure the table:
   - Display name: "UserVerification" (the system name will be "UserVerifications")
   - Primary column: "Name" (default)
6. Click "Create"
7. Once created, add the following columns:
   - Name: "EmployeeNumber", Type: Single line of text
   - Name: "EntraObject", Type: Lookup, Related Table: Users
     - This will allow looking up the Entra Object ID and the user's name
   - Name: "Q1", Type: Single line of text (for verification question 1)
   - Name: "A1", Type: Single line of text (for verification answer 1)
   - Name: "Q2", Type: Single line of text (for verification question 2)
   - Name: "A2", Type: Single line of text (for verification answer 2)
   - Add as many Q/A pairs as needed for your verification process

## Step 5: Create a Copilot Studio Agent

1. Navigate to [Microsoft Copilot Studio](https://copilotstudio.microsoft.com/)
2. Click "New" and select "Copilot"
3. Enter a name for your copilot (e.g., "Password Reset Assistant")
4. Select your language and click "Create"
5. In the agent configuration:
   - Go to "Settings" > "Security"
   - Set authentication to "None" (as we'll handle authentication within our flows)

## Step 6: Create a "Reset my Password" Topic

1. In your Copilot Studio agent, go to "Topics"
2. Click "New topic"
3. Add trigger phrase: "Reset my Password"
4. Click "Create"

## Step 7: Configure the Topic Flow

### Initial Collection of Employee Number

1. In the topic builder, add an "Ask a question" node
2. Enter: "What is your employee number?"
3. In the "Identify" section, select "User's entire response"
4. Set "Save user response as" to `varEmployeeNum` (Type: String)

### Create the "Get Employee Info" Flow

1. Add an "Action" node in your topic
2. Select "Call an action" and then "Create a flow"
3. Name the flow "Get Employee Info"
4. In Power Automate, configure the flow:

   a. Start with the trigger "When an agent calls the flow"
   
   b. Add an input to the trigger:
      - Name: "varEmployee"
      - Type: Text
   
   c. Add a Dataverse action "List rows":
      - Table: UserVerifications
      - Filter rows: `publishername_employeenumber eq '@{triggerBody()['varEmployee']}'`
      - Expand Query: `publishername_EntraObject($select=azureactivedirectoryobjectid,firstname,lastname)`
   
   d. Add a "Respond to Copilot" action with the following outputs:
      - FirstName: `first(outputs('List_rows')?['body/value'])?['publishername_EntraObject/firstname']`
      - LastName: `first(outputs('List_rows')?['body/value'])?['publishername_EntraObject/lastname']`
      - Q1: `first(outputs('List_rows')?['body/value'])?['publishername_q1']`
      - A1: `first(outputs('List_rows')?['body/value'])?['publishername_a1']`
      - Q2: `first(outputs('List_rows')?['body/value'])?['publishername_q2']`
      - A2: `first(outputs('List_rows')?['body/value'])?['publishername_a2']`
      - Add additional Q/A pairs as needed
   
   e. Save and publish the flow

### Validate Employee Number and Verify Identity

1. Back in Copilot Studio, add a "Condition" node to check if the EntraObject was found:
   - Condition: EntraObject is not blank
   
2. For the "No" branch (employee number not found):
   - Add a "Send a message" node: "Wrong employee number. Please try again later."
   - Add an "End topic" node

3. For the "Yes" branch (employee number found):
   - Add an "Ask a question" node with the text:
     "I have a few questions for you. If you answer them all correctly, I will issue a Temporary Access Pass valid for 10 minutes. Here's the first question: {Q1}"
   - Set "Save user response as" to `varQ1Response`

4. Add a "Condition" node to check the answer:
   - Condition: A1 is equal to varQ1Response
   
5. For the "No" branch (incorrect answer):
   - Add a "Send a message" node: "Wrong answer. For security reasons, this session will end."
   - Add an "End topic" node

6. For the "Yes" branch (correct answer):
   - Add an "Ask a question" node with text: "{Q2}"
   - Set "Save user response as" to `varQ2Response`

7. Add another "Condition" node:
   - Condition: A2 is equal to varQ2Response
   
8. Continue this pattern for as many questions as needed

### Create the "Issue TAP" Flow

1. After all verification questions are answered correctly, add an "Action" node
2. Select "Call an action" and then "Create a flow"
3. Name the flow "Issue TAP"
4. In Power Automate, configure the flow:

   a. Start with the trigger "When an agent calls the flow"
   
   b. Add an input to the trigger:
      - Name: "EntraObject"
      - Type: Text
   
   c. Add an "Azure Key Vault - Get Secret" action:
      - Key vault name: [Your key vault name]
      - Secret name: "ClientSecret"
   
   d. Add an "HTTP" action:
      - Method: POST
      - URI: `https://graph.microsoft.com/v1.0/users/@{triggerBody()['EntraObject']}/authentication/temporaryAccessPassMethods`
      - Headers: `Content-Type: application/json`
      - Body:
        ```json
        {
          "lifetimeInMinutes": 10,
          "isUsableOnce": true
        }
        ```
      - Authentication:
        - Authentication type: Active Directory OAuth
        - Authority: `https://login.windows.net`
        - Tenant: [Use TenantID from Key Vault or enter directly]
        - Audience: `https://graph.microsoft.com`
        - Client ID: [Use ClientID from Key Vault or enter directly]
        - Credential type: Secret
        - Secret: [Use the output from "Get Secret" action]
   
   e. Add a "Parse JSON" action:
      - Content: Body output from the HTTP action
      - Schema: Use sample payload from the [Microsoft documentation](https://learn.microsoft.com/en-us/graph/api/authentication-post-temporaryaccesspassmethods?view=graph-rest-1.0&tabs=http)
   
   f. Add a "Respond to Copilot" action:
      - Parameter name: "TAP"
      - Value: Output of Parse JSON.temporaryAccessPass
   
   g. Save and publish the flow

### Complete the Copilot Studio Topic

1. Back in Copilot Studio, add a "Send a message" node:
   - Message: "Your temporary access password which is valid for the next 10 minutes is: {TAP}"
   
2. Add an "End topic" node

## Step 8: Test the Copilot

1. In Copilot Studio, click "Test your copilot"
2. Type "Reset my password"
3. Enter a test employee number
4. Answer the verification questions
5. Verify that a Temporary Access Pass is generated

## Step 9: Publish and Deploy

1. In Copilot Studio, go to "Publish"
2. Review and publish your changes
3. Navigate to "Channels" to configure how users will access the copilot (e.g., Teams, Web, etc.)

## Maintenance Considerations

- Regularly rotate the client secret in Entra ID and update it in the Key Vault
- Periodically review and update the verification questions in the Dataverse table
- Monitor usage logs to detect potential abuse
- Consider implementing rate limiting or lockout policies for failed verification attempts
