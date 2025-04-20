## Overview

The Temporary Access Pass helpdesk solution includes an exported, unmanaged solution, and step-by-step instructions for creating a Microsoft Copilot Studio agent. The agent addresses a critical identity and access management challenge: helping users who are locked out of their accounts due to Multi-Factor Authentication (MFA) issues, particularly after getting a new device. 

### Use Cases
- Employees who have replaced their mobile device and cannot access their authenticator app
- Remote workers who need immediate access outside of IT support hours
- Users experiencing technical issues with their existing authentication methods
- Onboarding scenarios where temporary access is needed before permanent credentials are established

### Benefits
- Reduces helpdesk call volume by providing self-service password reset capabilities
- Minimizes productivity loss due to account lockouts
- Decreases IT support costs while maintaining strong security controls
- Provides 24/7 access recovery without requiring IT staff intervention
- Creates an auditable trail of access recovery attempts

### Security Considerations
As the scenario this solution addresses users who are not able to authenticate, this agent must accept unauthenticated chat requests.  It is important to realize that this means anyone and everyone can chat with the agent. If a user attempts to brute force answers, the agent will end the conversation with the user. This does not prevent the user from initiating another chat session repeatedly, so it is crucial to set up monitoring to alert and potentially even auto-blacklist any IP addresses which are repeatedly invoking the agent and providing incorrect answers in a short period of time. 

The solution incorporates several critical security measures:

1. **User Verification**: Before issuing a Temporary Access Pass, the agent conducts a thorough identity verification process using information that only the legitimate user should know. This multi-question approach significantly reduces the risk of unauthorized access.

2. **Limited Lifetime**: The Temporary Access Pass is configured with a short lifetime (60 minutes) and set as single-use only, minimizing the window of opportunity for potential misuse. It is important to note that the minimum Temporary Access Pass lifetime is 10 minutes and it is recommended to use the minimum value for security reasons. The TAP issued is also set to only allow a single use.

3. **Secure Credential Storage**: As a security best practice, the solution stores client IDs, tenant IDs, and secret keys in Azure Key Vault rather than embedding them directly in the flows. For proof-of-concept environments, these can optionally be stored in environment variables, though this is not recommended for production deployments.

### How the Agent Works
The agent operates through a conversational interface that follows a strict verification workflow:

1. The user initiates the conversation with the trigger phrase "Reset my Password"
2. The agent requests the user's employee number to identify them in the system
3. Upon finding a match, the agent begins the verification process by asking a series of security questions
4. If the user answers ANY question incorrectly, the agent immediately terminates the conversation with a generic message, preventing attackers from learning which specific answer was incorrect
5. Only after correctly answering ALL verification questions will the agent issue a Temporary Access Pass
6. The pass is displayed to the user along with its expiration time

The solution is highly customizable, allowing for:
- Any number of verification questions to balance security with user experience
- Optional integration with ticketing systems to automatically create and close support tickets for audit purposes
- Optional analytics enablement through Azure Application Insights
- Customization of messages and verification logic to suit organizational policies

### [Installation Instructions](https://github.com/russrimm/CopilotStudioWalkthroughs/blob/main/IssueTAP.md)
```

