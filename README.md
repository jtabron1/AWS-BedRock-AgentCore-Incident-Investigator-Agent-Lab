

**AWS AgentCore Hands-On Lab**

CloudTrail Security Event Investigator

*Build an AI agent that watches your AWS account for suspicious activity, investigates what happened, and explains it in plain English.*

| Skill Level Beginner-Friendly | Estimated Time 2-3 Hours | Prior Experience None Required |
| :---- | :---- | :---- |

| AWS Services Used AgentCore, Bedrock (Claude), Lambda, EventBridge, CloudTrail, SES, Security Hub, IAM, Secrets Manager | What You Will Build An AI agent that detects security group changes in your AWS account, investigates the full context, and emails you a plain-English summary. |
| :---- | :---- |

# **0\. Before You Begin: Terminal Basics**

This lab involves typing commands into a program called the terminal (also called the command line or command prompt). If you have never used a terminal before, this section will get you comfortable with the basics. Even if you have some experience, skim this section — it covers everything you will need.

## **0.1 What Is the Terminal?**

The terminal is a text-based way to give your computer instructions. Instead of clicking buttons and icons, you type commands and press Enter. It might look intimidating at first, but you will only need a small handful of commands for this lab and every command is explained as you go.

## **0.2 How to Open a Terminal**

### **On a Mac:**

1. Press Command (⌘) \+ Space on your keyboard to open Spotlight Search.

2. Type Terminal and press Enter.

3. A window with a dark or white background and a blinking cursor will appear. That is your terminal.

### **On Windows:**

4. Press the Windows key on your keyboard.

5. Type PowerShell and click Windows PowerShell.

6. A blue or dark window will appear. That is your terminal.

| ⚠️   NOTE: This lab was written with Mac users in mind. If you are on Windows, most commands will work the same way, but some installation steps differ. Notes for Windows users are included throughout. |
| :---- |

## **0.3 Understanding the Terminal Prompt**

When your terminal is ready to accept a command, you will see a line ending in $ or % followed by a blinking cursor. This is called the prompt. It means the terminal is waiting for you to type something.

| yourname@MacBook-Pro \~ % |
| :---- |

You do not need to type the prompt — it is already there. Just type your command after it and press Enter to run it.

## **0.4 How to Run a Command**

Every command in this lab follows the same pattern:

7. Click on your terminal window to make sure it is selected.

8. Type the command exactly as shown in the lab. Capitalization and spacing matter.

9. Press Enter to run the command.

10. Wait for the command to finish. You will know it is done when the prompt reappears.

| 💡  TIP: If a command seems to be taking a long time, it is probably still running. Do not close the terminal — just wait for the prompt to come back. |
| :---- |

## **0.5 Common Terminal Commands Reference**

Here is a quick reference for the basic terminal commands you will use in this lab. You do not need to memorize these — come back to this table whenever you need a reminder.

| Command | What It Does | Example |
| :---- | :---- | :---- |
| pwd | Shows which folder you are currently in | pwd |
| ls | Lists all files and folders in your current location | ls |
| cd \[folder\] | Moves you into a folder | cd Desktop |
| cd .. | Moves you back one folder level | cd .. |
| mkdir \[name\] | Creates a new folder | mkdir my-lab |
| cat \[file\] | Displays the contents of a file | cat agent.py |
| clear | Clears all the text off the terminal screen | clear |
| Up arrow key (↑) | Recalls your previous command so you can re-run or edit it | (press ↑) |
| Tab key | Auto-completes a file or folder name you are typing | cd Desk\[Tab\] |
| Ctrl \+ C | Cancels the current running command | (press Ctrl+C) |

## **0.6 What Does a Backslash (\\) Mean in Commands?**

Some commands in this lab are long and span multiple lines. The backslash character \\ at the end of a line is simply a way of saying "this command is not finished yet — continue reading on the next line." It is purely for readability.

For example, these two commands do exactly the same thing:

| aws iam create-role \--role-name MyRole \--assume-role-policy-document file://policy.json |
| :---- |

And the easier-to-read version using backslashes:

| aws iam create-role \\     \--role-name MyRole \\     \--assume-role-policy-document file://policy.json |
| :---- |
| **🔴  IMPORTANT:** When copying a command that uses backslashes, make sure you copy the entire block including every line. If you miss a line, the command will be incomplete. |

## **0.7 Installing a Code Editor**

Several steps in this lab require you to create Python code files. The easiest way to do this is with Visual Studio Code (VS Code) — a free, beginner-friendly code editor.

| 💻  TERMINAL ACTION: To install VS Code: 1\. Go to code.visualstudio.com in your web browser. 2\. Click the Download button for your operating system (Mac or Windows). 3\. Open the downloaded file and follow the installation instructions. 4\. Once installed, open VS Code. You will use it to create and edit the code files in this lab. |
| :---- |

| 🎓  WHY ARE WE DOING THIS? VS Code gives you a clean, visual way to write and save code files. It highlights different parts of the code in different colors, making it easier to read and spot mistakes. |
| :---- |

# **1\. What You Will Build**

## **1.1 The Big Picture**

Imagine you are a security manager and someone makes a change to your company's AWS account — maybe they accidentally open up a firewall rule that lets anyone on the internet access your servers. In a normal setup, you might not find out about that for hours or days, if ever.

In this lab, you will build a system where an AI agent watches your AWS account at all times. The moment a firewall rule changes, the agent wakes up, reads the details of what happened, pulls related activity from the last hour to see the full picture, reasons about whether it looks suspicious, and then sends you a plain-English email summary explaining exactly what happened and what the risk is.

No rules to configure. No dashboards to check. Just an intelligent agent that does the investigation and explains it to you.

## **1.2 How the System Works Step by Step**

| Step | Component | What Happens |
| :---- | :---- | :---- |
| 1 | CloudTrail | AWS automatically records every action taken in your account. This is always running — you don't need to set it up. |
| 2 | EventBridge | A rule watches the CloudTrail log for specific firewall-related events. When it sees one, it triggers your Lambda function. |
| 3 | Lambda Function | A small piece of code that wakes up when triggered and passes the event details to your AI agent. |
| 4 | AgentCore Agent | Your AI agent (powered by Claude on Amazon Bedrock) receives the event, uses tools to investigate, and reasons about what happened. |
| 5 | Email via SES | The agent sends you a plain-English email with a summary of what happened and an assessment of the risk. |
| 6 | Security Hub | The agent logs an official security finding for your records. This is useful for audits and compliance reporting. |

## **1.3 What You Will Learn**

* How to deploy an AI agent to AWS AgentCore using the command line

* How to connect AWS services together using EventBridge and Lambda

* How to give your agent tools so it can take actions (query logs, send emails, create findings)

* How AWS CloudTrail logs every action in your account and how to query those logs

* How to store sensitive configuration values securely using Secrets Manager

* How to verify everything is working end-to-end

## **1.4 What Will Trigger the Agent During Testing**

To test your system, you will intentionally make a firewall rule change in the AWS console — specifically, you will add an inbound rule to a security group that allows traffic from anywhere on the internet. This is a common mistake that organizations want to catch immediately.

When you make that change, you will then watch as your agent detects it, investigates it, and sends you an email — all automatically, within a few minutes.

| 🎓  WHY ARE WE DOING THIS? Security groups are AWS's version of a firewall. They control what network traffic is allowed to reach your cloud resources. A rule that allows traffic from 0.0.0.0/0 (which means any IP address on the internet) on a sensitive port is a common security misconfiguration that attackers look for. |
| :---- |

# **2\. Setting Up Your Computer**

This section walks through installing everything you need before starting the lab. If something is already installed on your computer, you can skip that step.

## **2.1 Install Homebrew (Mac Only)**

Homebrew is a package manager for Mac — think of it as an app store for developer tools that you install from the terminal. It makes installing Python, the AWS CLI, and other tools much easier.

| 💻  TERMINAL ACTION: Open your terminal and run the following command. This will download and install Homebrew. It may take a few minutes. |
| :---- |
| /bin/bash \-c "$(curl \-fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" |

When the installation finishes, verify it worked by running:

| brew \--version |
| :---- |

You should see a version number printed, such as Homebrew 4.2.0. If you do, Homebrew is installed successfully.

| ⚠️   NOTE: Windows users: You do not need Homebrew. For Windows, the recommended approach is to install Windows Subsystem for Linux (WSL). Search for 'Install WSL Windows' on Microsoft's website for instructions, then proceed with the Mac/Linux commands in this lab inside your WSL terminal. |
| :---- |

## **2.2 Install Python 3.10**

Python is the programming language used to write the agent code in this lab. AWS AgentCore requires Python 3.10 specifically — newer versions such as 3.12 or 3.13 will cause installation errors later in the lab.

| 💻  TERMINAL ACTION: In your terminal, check which version of Python you have: |
| :---- |
| python3 \--version |

If you see Python 3.10.x, you are good — skip to Section 2.3.

If you see any other version or an error, install Python 3.10:

### **Mac:**

| brew install python@3.10 |
| :---- |

After installation, verify it installed correctly:

| python3.10 \--version |
| :---- |

You should see Python 3.10.x printed. This is the version you will use when creating your virtual environment in Section 8.1.

### **Windows:**

Go to python.org/downloads/release/python-31011, download the Windows installer for Python 3.10, and run it. During installation, check the box that says Add Python to PATH.

| 🔴  IMPORTANT: Do not install a newer Python version thinking it will work better. AgentCore's starter toolkit is pinned to Python 3.10 and will fail to install on 3.11, 3.12, or 3.13. |
| :---- |

## **2.3 Install the AWS CLI**

The AWS CLI (Command Line Interface) is the main tool you will use to interact with AWS from your terminal. It lets you create resources, configure services, and manage your account without needing to use the AWS website.

| 💻  TERMINAL ACTION: Check if the AWS CLI is already installed: |
| :---- |
| aws \--version |

If you see output like aws-cli/2.15.0, it is already installed — skip to Section 2.4.

If you see an error, install it:

### **Mac:**

| brew install awscli |
| :---- |

### **Windows:**

Go to docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html and download the Windows installer. Run it and follow the prompts.

After installation, verify it worked:

| aws \--version |
| :---- |
| **💡  TIP:** The AWS CLI version should be 2.x or higher. If it shows version 1.x, consider upgrading. |

## **2.4 Install the uv Package Manager**

uv is a fast Python package manager that AWS AgentCore requires for deploying your agent code. Think of it as a tool that helps bundle your Python code and its dependencies together.

| 💻  TERMINAL ACTION: In your terminal, run: |
| :---- |
| curl \-LsSf https://astral.sh/uv/install.sh | sh |

After the installation completes, apply the changes to your terminal session:

### **Mac / Linux:**

| source \~/.zshrc |
| :---- |

### **If that does not work, try:**

| source \~/.bashrc |
| :---- |

Verify uv is installed:

| uv \--version |
| :---- |

You should see a version number like uv 0.4.0.

# **3\. AWS Account Setup**

## **3.1 Create an AWS Account (If You Don't Have One)**

If you already have an AWS account, skip to Section 3.2.

To create a free AWS account:

11. Go to aws.amazon.com and click Create an AWS Account.

12. Follow the sign-up process. You will need a credit card, but this lab stays within the AWS Free Tier — you should not be charged.

13. Once your account is active, sign in to the AWS Management Console at console.aws.amazon.com.

## **3.2 Create a Dedicated IAM User for This Lab**

Rather than using your AWS account's root credentials (the main login), it is best practice to create a dedicated user for this lab. This keeps things organized and is safer.

| 🎓  WHY ARE WE DOING THIS? Your root account has unlimited access to everything in AWS. Using it for day-to-day work is like using the master key to your building for every errand. A dedicated IAM user lets you scope permissions appropriately and avoids accidental damage. |
| :---- |

### **Step 1: Create the user**

| 💻  TERMINAL ACTION: In your terminal, run the following command to create a new IAM user named cloudtrail-lab: |
| :---- |
| aws iam create-user \\     \--user-name cloudtrail-lab |

### **Step 2: Attach administrator permissions**

| 💻  TERMINAL ACTION: Run this command to give your lab user full AWS access: |
| :---- |
| aws iam attach-user-policy \\     \--user-name cloudtrail-lab \\     \--policy-arn arn:aws:iam::aws:policy/AdministratorAccess |

### **Step 3: Create access keys**

| 💻  TERMINAL ACTION: Run this command to generate credentials for your lab user: |
| :---- |
| aws iam create-access-key \\     \--user-name cloudtrail-lab |

You will see output like this:

| {     "AccessKey": {         "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",         "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"     } } |
| :---- |
| **🔴  IMPORTANT:** Copy both the AccessKeyId and SecretAccessKey values right now and save them somewhere safe. The SecretAccessKey is only shown once. If you lose it, you will need to create new keys. |

### **Step 4: Configure your AWS CLI profile**

| 💻  TERMINAL ACTION: Run this command. The CLI will ask you four questions — answer them as shown below: |
| :---- |
| aws configure \--profile cloudtrail-lab |

When prompted, enter the following:

| Prompt | What to Enter |
| :---- | :---- |
| AWS Access Key ID | Your AccessKeyId from Step 3 |
| AWS Secret Access Key | Your SecretAccessKey from Step 3 |
| Default region name | us-east-1 |
| Default output format | json |

### **Step 5: Verify your profile works**

| 💻  TERMINAL ACTION: Run this command to confirm everything is configured correctly: |
| :---- |
| aws sts get-caller-identity \\     \--profile cloudtrail-lab |

You should see output showing your AWS account ID and the cloudtrail-lab user. If you see this, your credentials are working correctly.

| 💡  TIP: The \--profile cloudtrail-lab part that appears in AWS CLI commands throughout this lab tells the CLI which set of credentials to use. Always include it. |
| :---- |

# **4\. Enable Required AWS Services**

Before building anything, you need to turn on the AWS services that your agent will use. This section walks through each one.

## **4.1 Enable Amazon Bedrock Model Access**

Amazon Bedrock is the AWS service that hosts AI models including Claude. Before your agent can use Claude, you need to explicitly request access.

| 🖥️  AWS CONSOLE ACTION: 1\. Go to console.aws.amazon.com and sign in. 2\. In the search bar at the top, type Bedrock and click Amazon Bedrock. 3\. In the left navigation panel, click Model access. 4\. Click the orange Modify model access button. 5\. Find Anthropic in the list and check the box next to Claude Sonnet 4\. 6\. Click Next, then click Submit. 7\. Wait for the Status column to change to Access granted. This usually takes 1-2 minutes. |
| :---- |

## **4.2 Enable AWS Security Hub**

Security Hub is where your agent will log its findings. Think of it as a central dashboard for security alerts across your AWS account.

| 💻  TERMINAL ACTION: In your terminal, run: |
| :---- |
| aws securityhub enable-security-hub \\     \--profile cloudtrail-lab \\     \--region us-east-1 \\     \--enable-default-standards |
| **⚠️   NOTE:** If you see an error that says Security Hub is already enabled, that is fine — it just means it was already turned on. Continue to the next step. |

## **4.3 Verify Your Email for SES**

Amazon SES (Simple Email Service) is how your agent will send you email alerts. AWS requires you to verify email addresses before they can be used for sending or receiving.

| 💻  TERMINAL ACTION: Replace your-email@domain.com with your actual email address and run: |
| :---- |
| aws ses verify-email-identity \\     \--profile cloudtrail-lab \\     \--email-address your-email@domain.com \\     \--region us-east-1 |
| **🔴  IMPORTANT:** Check your inbox right now and click the verification link that AWS just sent you. Your agent cannot send emails until this step is done. The email comes from no-reply-aws@amazon.com with the subject 'Amazon Web Services – Email Address Verification Request'. |

# **5\. Create Permissions for Your Agent and Lambda**

In AWS, every service that needs to do things needs explicit permission to do those things. In this section you will create two permission sets: one for your AI agent and one for your Lambda function.

| 🎓  WHY ARE WE DOING THIS? Think of IAM roles like job descriptions at a company. A receptionist has permission to answer phones and schedule meetings, but not to access financial records. In the same way, your Lambda function has permission to invoke your agent, but not to directly access your emails. This principle of least privilege — only giving each component exactly the permissions it needs — is a security best practice. |
| :---- |

## **5.1 Create the AgentCore Execution Role**

This role defines what your AI agent is allowed to do. Your agent needs to be able to read CloudTrail logs, send emails, and log findings to Security Hub.

### **Step 1: Create a trust policy file**

First, you need to create a file on your computer that defines which AWS service is allowed to use this role. Open VS Code and create a new file with the following content, then save it as agent-trust-policy.json in a folder called cloudtrail-lab on your Desktop.

To create the folder, run this in your terminal:

| 💻  TERMINAL ACTION: In your terminal, run: |
| :---- |
| mkdir \~/Desktop/cloudtrail-lab cd \~/Desktop/cloudtrail-lab |

Now open VS Code, create a new file, paste the following content, and save it as agent-trust-policy.json inside your cloudtrail-lab folder:

| {     "Version": "2012-10-17",     "Statement": \[         {             "Effect": "Allow",             "Principal": {                 "Service": "bedrock.amazonaws.com"             },             "Action": "sts:AssumeRole"         },         {             "Effect": "Allow",             "Principal": {                 "Service": "bedrock-agentcore.amazonaws.com"             },             "Action": "sts:AssumeRole"         }     \] } |
| :---- |
| **🎓  WHY ARE WE DOING THIS?** Two service principals are listed here because Bedrock and AgentCore are distinct services. bedrock.amazonaws.com covers general Bedrock model invocations. bedrock-agentcore.amazonaws.com is the service principal used when AgentCore deploys and manages your agent at runtime. Without both entries, the deployment will fail with a role validation error. |

### **Step 2: Create the role**

| 💻  TERMINAL ACTION: Make sure you are in your cloudtrail-lab folder (run cd \~/Desktop/cloudtrail-lab if not), then run: |
| :---- |
| aws iam create-role \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorAgentRole \\     \--assume-role-policy-document file://agent-trust-policy.json |

### **Step 3: Create a permissions policy file**

Create a new file in VS Code, paste the following content, and save it as agent-policy.json in your cloudtrail-lab folder:

| {     "Version": "2012-10-17",     "Statement": \[         {             "Sid": "CloudTrailReadAccess",             "Effect": "Allow",             "Action": \[                 "cloudtrail:LookupEvents",                 "cloudtrail:GetTrailStatus",                 "cloudtrail:DescribeTrails"             \],             "Resource": "\*"         },         {             "Sid": "EC2ReadAccess",             "Effect": "Allow",             "Action": \[                 "ec2:DescribeSecurityGroups",                 "ec2:DescribeSecurityGroupRules"             \],             "Resource": "\*"         },         {             "Sid": "SecretsAccess",             "Effect": "Allow",             "Action": \["secretsmanager:GetSecretValue"\],             "Resource": "arn:aws:secretsmanager:\*:\*:secret:cloudtrail-investigator/\*"         },         {             "Sid": "SecurityHubAccess",             "Effect": "Allow",             "Action": \["securityhub:BatchImportFindings"\],             "Resource": "\*"         },         {             "Sid": "SESAccess",             "Effect": "Allow",             "Action": \["ses:SendEmail", "ses:SendRawEmail"\],             "Resource": "\*"         },         {             "Sid": "BedrockAccess",             "Effect": "Allow",             "Action": \[                 "bedrock:InvokeModel",                 "bedrock:InvokeModelWithResponseStream"             \],             "Resource": "\*"         },         {             "Sid": "STSAccess",             "Effect": "Allow",             "Action": \["sts:GetCallerIdentity"\],             "Resource": "\*"         }     \] } |
| :---- |

### **Step 4: Create the policy in AWS and attach it to your role**

First, get your AWS account ID — you will need it for the next command:

| 💻  TERMINAL ACTION: Run this command and note the Account value in the output: |
| :---- |
| aws sts get-caller-identity \\     \--profile cloudtrail-lab \\     \--query Account \\     \--output text |
| **💻  TERMINAL ACTION:** Now create the policy (replace YOUR\_ACCOUNT\_ID with the number from the previous command): |
| aws iam create-policy \\     \--profile cloudtrail-lab \\     \--policy-name CloudTrailInvestigatorAgentPolicy \\     \--policy-document file://agent-policy.json |
| aws iam attach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorAgentRole \\     \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorAgentPolicy |

## **5.2 Create the Lambda Execution Role**

This role defines what your Lambda function is allowed to do. It needs permission to invoke your agent and to write logs so you can see what is happening.

### **Step 1: Create a trust policy file for Lambda**

In VS Code, create a new file with the following content and save it as lambda-trust-policy.json in your cloudtrail-lab folder:

| {     "Version": "2012-10-17",     "Statement": \[         {             "Effect": "Allow",             "Principal": {                 "Service": "lambda.amazonaws.com"             },             "Action": "sts:AssumeRole"         }     \] } |
| :---- |

### **Step 2: Create the Lambda role**

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws iam create-role \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorLambdaRole \\     \--assume-role-policy-document file://lambda-trust-policy.json |

### **Step 3: Create a permissions policy for Lambda**

In VS Code, create a new file with the following content and save it as lambda-policy.json:

| {     "Version": "2012-10-17",     "Statement": \[         {             "Sid": "AgentCoreInvoke",             "Effect": "Allow",             "Action": \["bedrock:InvokeAgent"\],             "Resource": "\*"         },         {             "Sid": "CloudWatchLogs",             "Effect": "Allow",             "Action": \[                 "logs:CreateLogGroup",                 "logs:CreateLogStream",                 "logs:PutLogEvents"             \],             "Resource": "\*"         }     \] } |
| :---- |
| **💻  TERMINAL ACTION:** Create and attach the Lambda policy (replace YOUR\_ACCOUNT\_ID): |
| aws iam create-policy \\     \--profile cloudtrail-lab \\     \--policy-name CloudTrailInvestigatorLambdaPolicy \\     \--policy-document file://lambda-policy.json |
| aws iam attach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorLambdaRole \\     \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorLambdaPolicy |
| aws iam attach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorLambdaRole \\     \--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole |

# **6\. Store Your Email Configuration Securely**

Your agent needs to know which email address to send alerts from and to. Rather than putting this information directly in your code (which is a security risk), you will store it in AWS Secrets Manager — a secure vault for sensitive configuration values.

| 🎓  WHY ARE WE DOING THIS? Think of Secrets Manager like a locked filing cabinet. Your agent knows where the cabinet is and has a key to open it, but the sensitive values are never sitting out in the open in your code. If someone ever looks at your code, they cannot find your credentials. |
| :---- |
| **💻  TERMINAL ACTION:** Run the following command, replacing both email addresses with your own verified SES email address from Section 4.3: |
| aws secretsmanager create-secret \\     \--profile cloudtrail-lab \\     \--name cloudtrail-investigator/notifications \\     \--secret-string '{"sender\_email": "your-email@domain.com", "recipient\_email": "your-email@domain.com"}' \\     \--region us-east-1 |
| **💡  TIP:** For this lab, the sender and recipient are the same address — your email. In a real production setup, the sender might be a no-reply company address and the recipient might be a security team distribution list. |

# **7\. Create the Agent Code**

Now you will create the Python files that make up your AI agent. Each file has a specific purpose and they work together as a team.

Create a new folder inside your cloudtrail-lab folder called agent:

| 💻  TERMINAL ACTION: In your terminal, run: |
| :---- |
| mkdir \~/Desktop/cloudtrail-lab/agent mkdir \~/Desktop/cloudtrail-lab/agent/tools cd \~/Desktop/cloudtrail-lab/agent |

## **7.1 Create requirements.txt**

This file tells Python which external libraries your agent needs. Think of it like a shopping list that Python will use to install everything required.

In VS Code, create a new file with the following content and save it as requirements.txt inside your cloudtrail-lab/agent folder:

| bedrock-agentcore\>=0.1.0 bedrock-agentcore-starter-toolkit\>=0.1.21 strands-agents\>=0.1.0 strands-agents-tools\>=0.1.0 boto3\>=1.35.0 |
| :---- |

## **7.2 Create agent.py**

This is the brain of your agent. It defines the agent's personality (via its system prompt), registers all the tools the agent can use, and handles incoming events.

In VS Code, create a new file with the following content and save it as agent.py inside your cloudtrail-lab/agent folder:

| """ CloudTrail Security Event Investigator Agent """ from bedrock\_agentcore import BedrockAgentCoreApp from strands import Agent from strands.models import BedrockModel app \= BedrockAgentCoreApp() def get\_agent():     """     Initialize the agent with lazy loading.     We do this inside a function rather than at the top of the file     to avoid slow startup times when the agent first wakes up.     """     from tools import (         get\_cloudtrail\_events,         get\_security\_group\_details,         send\_investigation\_email,         create\_security\_finding     )     model \= BedrockModel(         model\_id="us.anthropic.claude-sonnet-4-6"     )     return Agent(         model=model,         tools=\[             get\_cloudtrail\_events,             get\_security\_group\_details,             send\_investigation\_email,             create\_security\_finding         \],         system\_prompt="""         You are a cloud security investigator agent. Your job is to analyze         AWS security events, understand what happened, and communicate it         clearly to non-technical stakeholders.         When you receive a security event:         1\. Use get\_cloudtrail\_events to look up what else the same user            did in the 60 minutes before and after this event. Look for            any patterns that suggest malicious or accidental behavior.         2\. If the event involves a security group change, use            get\_security\_group\_details to get more context about the            affected firewall rule.         3\. Assess the severity:            \- CRITICAL: Changes that expose sensitive ports (22, 3389, 3306,              5432\) to the entire internet (0.0.0.0/0)            \- HIGH: Changes that expose any port to the entire internet            \- MEDIUM: Changes that open access to a broad but not universal              IP range            \- LOW: Changes to egress rules or non-sensitive configurations            \- INFORMATIONAL: Rule removals that improve security posture         4\. Use send\_investigation\_email to send a clear, plain-English            email. Write as if explaining to someone who is not technical.            Include: what changed, who made the change, when, what the            risk is, and what action (if any) to take.         5\. Use create\_security\_finding to log an official record in            Security Hub.         Be concise, clear, and avoid unnecessary jargon.         """     ) @app.entrypoint def handler(payload, context):     """Handle incoming security events from EventBridge via Lambda"""     agent \= get\_agent()     event\_name   \= payload.get("event\_name", "UnknownEvent")     event\_time   \= payload.get("event\_time", "")     user\_identity \= payload.get("user\_identity", {})     source\_ip    \= payload.get("source\_ip", "unknown")     resources    \= payload.get("resources", \[\])     raw\_event    \= payload.get("raw\_event", {})     username \= (         user\_identity.get("userName")         or user\_identity.get("sessionContext", {}).get("sessionIssuer", {}).get("userName")         or user\_identity.get("arn", "unknown")     )     prompt \= f"""     A security event has occurred in this AWS account. Please investigate     and report on it.     Event Name: {event\_name}     Time: {event\_time}     Performed By: {username}     Source IP Address: {source\_ip}     Affected Resources: {resources}     Raw Event Details:     {raw\_event}     Please investigate this event, check for related activity in the     past hour from this user, assess the risk, send an email summary,     and create a Security Hub finding.     """     result \= agent(prompt)     return {"status": "completed", "result": str(result)} if \_\_name\_\_ \== "\_\_main\_\_":     app.run() |
| :---- |

## **7.3 Create tools/\_\_init\_\_.py**

This file makes your tools folder recognizable to Python as a package. Create a new file with the following content and save it as \_\_init\_\_.py inside your cloudtrail-lab/agent/tools folder:

| """ Agent Tools Package """ from tools.cloudtrail\_tools import get\_cloudtrail\_events from tools.ec2\_tools import get\_security\_group\_details from tools.notification\_tools import send\_investigation\_email from tools.security\_hub\_tools import create\_security\_finding |
| :---- |

## **7.4 Create tools/cloudtrail\_tools.py**

This tool allows your agent to look up related CloudTrail events — essentially asking "what else did this user do recently?" Save this file as cloudtrail\_tools.py inside your cloudtrail-lab/agent/tools folder:

| """ CloudTrail Tools \- Query related security events """ import boto3 from strands import tool from datetime import datetime, timedelta, timezone @tool def get\_cloudtrail\_events(username: str, minutes\_back: int \= 60\) \-\> dict:     """     Look up recent CloudTrail events for a specific user.     This lets the agent understand the full context of what     a user was doing around the time of the security event.     Args:         username: The IAM username or ARN to look up         minutes\_back: How many minutes of history to retrieve (default 60\)     Returns:         A dictionary containing recent events for that user     """     try:         cloudtrail \= boto3.client("cloudtrail", region\_name="us-east-1")         end\_time   \= datetime.now(timezone.utc)         start\_time \= end\_time \- timedelta(minutes=minutes\_back)         response \= cloudtrail.lookup\_events(             LookupAttributes=\[                 {                     "AttributeKey": "Username",                     "AttributeValue": username                 }             \],             StartTime=start\_time,             EndTime=end\_time,             MaxResults=20         )         events \= \[\]         for event in response.get("Events", \[\]):             events.append({                 "event\_name":   event.get("EventName"),                 "event\_time":   event.get("EventTime").isoformat() if event.get("EventTime") else None,                 "event\_source": event.get("EventSource"),                 "resources":    \[r.get("ResourceName") for r in event.get("Resources", \[\])\]             })         return {             "success":     True,             "username":    username,             "event\_count": len(events),             "time\_window": f"Last {minutes\_back} minutes",             "events":      events         }     except Exception as e:         return {"success": False, "error": str(e)} |
| :---- |

## **7.5 Create tools/ec2\_tools.py**

This tool lets your agent get details about a specific security group (firewall rule set) that was modified. Save this as ec2\_tools.py inside your cloudtrail-lab/agent/tools folder:

| """ EC2 Tools \- Get security group details """ import boto3 from strands import tool @tool def get\_security\_group\_details(security\_group\_id: str) \-\> dict:     """     Retrieve details about a specific security group including     all current inbound and outbound rules.     Args:         security\_group\_id: The security group ID (e.g. sg-12345abc)     Returns:         A dictionary with security group name, description, and rules     """     try:         ec2 \= boto3.client("ec2", region\_name="us-east-1")         response \= ec2.describe\_security\_groups(             GroupIds=\[security\_group\_id\]         )         groups \= response.get("SecurityGroups", \[\])         if not groups:             return {"success": False, "error": f"Security group {security\_group\_id} not found"}         group \= groups\[0\]         inbound\_rules \= \[\]         for rule in group.get("IpPermissions", \[\]):             for ip\_range in rule.get("IpRanges", \[\]):                 inbound\_rules.append({                     "protocol":    rule.get("IpProtocol"),                     "from\_port":   rule.get("FromPort"),                     "to\_port":     rule.get("ToPort"),                     "cidr":        ip\_range.get("CidrIp"),                     "description": ip\_range.get("Description", "No description")                 })         return {             "success":          True,             "group\_id":         group.get("GroupId"),             "group\_name":       group.get("GroupName"),             "description":      group.get("Description"),             "vpc\_id":           group.get("VpcId"),             "inbound\_rules":    inbound\_rules,             "outbound\_rule\_count": len(group.get("IpPermissionsEgress", \[\]))         }     except Exception as e:         return {"success": False, "error": str(e)} |
| :---- |

## **7.6 Create tools/notification\_tools.py**

This tool sends the investigation summary email. It retrieves your email address from Secrets Manager and sends a formatted HTML email. Save this as notification\_tools.py inside your cloudtrail-lab/agent/tools folder:

| """ Notification Tools \- Send investigation emails via SES """ import boto3 import json from strands import tool from datetime import datetime def get\_notification\_config():     """Retrieve email configuration from Secrets Manager"""     client \= boto3.client("secretsmanager", region\_name="us-east-1")     response \= client.get\_secret\_value(         SecretId="cloudtrail-investigator/notifications"     )     return json.loads(response\["SecretString"\]) SEVERITY\_COLORS \= {     "CRITICAL":      "\#e53e3e",     "HIGH":          "\#dd6b20",     "MEDIUM":        "\#d69e2e",     "LOW":           "\#3182ce",     "INFORMATIONAL": "\#38a169" } @tool def send\_investigation\_email(     subject: str,     what\_happened: str,     who\_did\_it: str,     when\_it\_happened: str,     risk\_assessment: str,     recommended\_action: str,     severity: str ) \-\> dict:     """     Send a plain-English security investigation summary email.     Args:         subject: Email subject line         what\_happened: Clear description of the event in plain English         who\_did\_it: Who performed the action (username, IP address)         when\_it\_happened: Timestamp of the event         risk\_assessment: What the risk is and why it matters         recommended\_action: What the recipient should do (or "No action needed")         severity: CRITICAL, HIGH, MEDIUM, LOW, or INFORMATIONAL     Returns:         Dictionary with success status and email message ID     """     try:         config \= get\_notification\_config()         ses    \= boto3.client("ses", region\_name="us-east-1")         color \= SEVERITY\_COLORS.get(severity.upper(), "\#718096")         html \= f"""         \<html\>         \<body style="font-family: Arial, sans-serif; max-width: 640px;                      margin: 0 auto; color: \#2d3748;"\>           \<div style="background: {color}; padding: 24px; border-radius: 8px 8px 0 0;"\>             \<h1 style="color: white; margin: 0; font-size: 22px;"\>               Security Event Detected             \</h1\>             \<span style="background: rgba(255,255,255,0.25); color: white;                          padding: 4px 12px; border-radius: 20px;                          font-size: 14px; font-weight: bold;"\>               {severity.upper()}             \</span\>           \</div\>           \<div style="background: \#f7fafc; padding: 24px;"\>             \<div style="background: white; border-radius: 8px; padding: 20px;                         margin-bottom: 16px; border-left: 4px solid {color};"\>               \<h2 style="margin: 0 0 12px 0; font-size: 16px; color: \#1a202c;"\>                 What Happened               \</h2\>               \<p style="margin: 0; line-height: 1.6;"\>{what\_happened}\</p\>             \</div\>             \<div style="background: white; border-radius: 8px; padding: 20px;                         margin-bottom: 16px;"\>               \<table style="width: 100%; border-collapse: collapse;"\>                 \<tr\>                   \<td style="padding: 8px 0; border-bottom: 1px solid \#e2e8f0;                              font-weight: bold; width: 140px; color: \#718096;                              font-size: 13px;"\>PERFORMED BY\</td\>                   \<td style="padding: 8px 0; border-bottom: 1px solid \#e2e8f0;"\>{who\_did\_it}\</td\>                 \</tr\>                 \<tr\>                   \<td style="padding: 8px 0; font-weight: bold; color: \#718096;                              font-size: 13px;"\>TIME\</td\>                   \<td style="padding: 8px 0;"\>{when\_it\_happened}\</td\>                 \</tr\>               \</table\>             \</div\>             \<div style="background: white; border-radius: 8px; padding: 20px;                         margin-bottom: 16px;"\>               \<h2 style="margin: 0 0 12px 0; font-size: 16px; color: \#1a202c;"\>                 Risk Assessment               \</h2\>               \<p style="margin: 0; line-height: 1.6;"\>{risk\_assessment}\</p\>             \</div\>             \<div style="background: {"\#fff5f5" if severity in \["CRITICAL","HIGH"\] else "\#f0fff4"};                         border-radius: 8px; padding: 20px;"\>               \<h2 style="margin: 0 0 12px 0; font-size: 16px; color: \#1a202c;"\>                 Recommended Action               \</h2\>               \<p style="margin: 0; line-height: 1.6;"\>{recommended\_action}\</p\>             \</div\>           \</div\>           \<div style="padding: 16px 24px; background: \#edf2f7; font-size: 12px;                       color: \#718096; border-radius: 0 0 8px 8px;"\>             This alert was generated automatically by the CloudTrail             Security Event Investigator Agent.           \</div\>         \</body\>         \</html\>         """         response \= ses.send\_email(             Source=config\["sender\_email"\],             Destination={"ToAddresses": \[config\["recipient\_email"\]\]},             Message={                 "Subject": {"Data": f"\[{severity.upper()}\] {subject}"},                 "Body": {                     "Html": {"Data": html},                     "Text": {"Data": (                         f"Security Event: {subject}\\n\\n"                         f"What Happened: {what\_happened}\\n"                         f"Performed By: {who\_did\_it}\\n"                         f"Time: {when\_it\_happened}\\n"                         f"Risk: {risk\_assessment}\\n"                         f"Action: {recommended\_action}"                     )}                 }             }         )         return {"success": True, "message\_id": response\["MessageId"\]}     except Exception as e:         return {"success": False, "error": str(e)} |
| :---- |

## **7.7 Create tools/security\_hub\_tools.py**

This tool creates an official security finding in Security Hub — a permanent, timestamped record of the event. Save this as security\_hub\_tools.py inside your cloudtrail-lab/agent/tools folder:

| """ Security Hub Tools \- Create official security findings """ import boto3 from strands import tool from datetime import datetime, timezone @tool def create\_security\_finding(     title: str,     description: str,     severity: str,     event\_name: str,     username: str,     resource\_id: str \= "aws:cloudtrail:event" ) \-\> dict:     """     Create an official security finding in AWS Security Hub.     This provides a permanent, auditable record of the security event.     Args:         title: Short descriptive title for the finding         description: Full description of what happened and the risk         severity: CRITICAL, HIGH, MEDIUM, LOW, or INFORMATIONAL         event\_name: The CloudTrail event name (e.g. AuthorizeSecurityGroupIngress)         username: The user who performed the action         resource\_id: The affected AWS resource identifier     Returns:         Dictionary with success status and finding ID     """     try:         sts        \= boto3.client("sts")         account\_id \= sts.get\_caller\_identity()\["Account"\]         region     \= "us-east-1"         now        \= datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")         finding\_id \= f"cloudtrail-investigator-{event\_name}-{datetime.now(timezone.utc).strftime('%Y%m%d%H%M%S')}"         severity\_map \= {             "CRITICAL":      {"Label": "CRITICAL",      "Normalized": 90},             "HIGH":          {"Label": "HIGH",          "Normalized": 70},             "MEDIUM":        {"Label": "MEDIUM",        "Normalized": 40},             "LOW":           {"Label": "LOW",           "Normalized": 20},             "INFORMATIONAL": {"Label": "INFORMATIONAL", "Normalized": 0}         }         finding \= {             "SchemaVersion": "2018-10-08",             "Id":            finding\_id,             "ProductArn":    f"arn:aws:securityhub:{region}:{account\_id}:product/{account\_id}/default",             "GeneratorId":   "cloudtrail-investigator-agent",             "AwsAccountId":  account\_id,             "Types":         \["TTPs/Initial Access/Unusual Activity Detected"\],             "CreatedAt":     now,             "UpdatedAt":     now,             "Severity":      severity\_map.get(severity.upper(), severity\_map\["MEDIUM"\]),             "Title":         title,             "Description":   description,             "Remediation": {                 "Recommendation": {                     "Text": "Review the CloudTrail event and take corrective action if needed.",                     "Url":  f"https://console.aws.amazon.com/cloudtrail/home?region={region}\#/events"                 }             },             "Resources": \[{                 "Type":   "Other",                 "Id":     resource\_id,                 "Region": region             }\],             "Compliance":    {"Status": "FAILED"},             "WorkflowState": "NEW",             "RecordState":   "ACTIVE",             "Note": {                 "Text":      f"Investigated by CloudTrail Investigator Agent | User: {username} | Event: {event\_name} | NIST 800-53: SI-4 | SOC 2: CC7.2",                 "UpdatedBy": "cloudtrail-investigator-agent",                 "UpdatedAt": now             }         }         securityhub \= boto3.client("securityhub", region\_name=region)         response    \= securityhub.batch\_import\_findings(Findings=\[finding\])         return {             "success":      True,             "finding\_id":   finding\_id,             "failed\_count": response.get("FailedCount", 0\)         }     except Exception as e:         return {"success": False, "error": str(e)} |
| :---- |

# **8\. Deploy the Agent**

With all your code written, it is time to deploy your agent to AWS AgentCore — Amazon's managed platform for hosting AI agents.

## **8.1 Set Up a Python Virtual Environment**

A virtual environment is an isolated space for your Python project. It keeps the libraries for this project separate from other Python projects on your computer. Because AgentCore requires Python 3.10, you will use uv to create the environment with that specific version.

| 💻  TERMINAL ACTION: In your terminal, navigate to your agent folder and create the virtual environment: |
| :---- |
| cd \~/Desktop/cloudtrail-lab/agent uv venv \--python 3.10 source .venv/bin/activate |

After running the last command, you should see (.venv) at the beginning of your terminal prompt. This confirms the virtual environment is active and using Python 3.10.

| 🎓  WHY ARE WE DOING THIS? Without a virtual environment, installing libraries for one project can interfere with other projects on your computer. The virtual environment creates a clean, isolated container. Forcing Python 3.10 here ensures compatibility with the AgentCore starter toolkit. |
| :---- |

## **8.2 Install the AgentCore CLI and Dependencies**

| 💻  TERMINAL ACTION: With your virtual environment active (you should see (.venv) in your prompt), run: |
| :---- |
| uv pip install bedrock-agentcore-starter-toolkit uv pip install \-r requirements.txt |

This may take a minute or two. You will see a lot of text scroll by — that is normal.

| 🔴  IMPORTANT: Use uv pip install rather than regular pip install here. The AgentCore starter toolkit requires Python 3.10 and uv ensures the packages are installed into your Python 3.10 virtual environment correctly. Using plain pip can pull in incompatible versions. |
| :---- |

## **8.3 Get Your Agent Role ARN**

Your agent needs to know which IAM role to use. Run the following to get the ARN (Amazon Resource Name — a unique identifier) for the role you created earlier:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws iam get-role \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorAgentRole \\     \--query Role.Arn \\     \--output text |

Copy the output — it looks like: arn:aws:iam::123456789012:role/CloudTrailInvestigatorAgentRole

## **8.4 Configure the Agent**

| 💻  TERMINAL ACTION: With your virtual environment still active and inside the agent folder, run: |
| :---- |
| AWS\_PROFILE=cloudtrail-lab agentcore configure \-e agent.py |

The command will ask you several questions. Answer them as follows:

| Question | Your Answer |
| :---- | :---- |
| Agent name | CloudTrailInvestigator (no spaces or special characters) |
| Execution Role ARN | Paste the ARN you copied in Section 8.3 |
| ECR Repository | Press Enter to let AWS create one automatically |
| Memory \- Short-term | Type yes and press Enter |
| Memory \- Long-term | Type no and press Enter |

## **8.5 Test the Agent Locally**

Before deploying to AWS, test that the agent works on your computer.

### **In your current terminal window, start the local agent server:**

| 💻  TERMINAL ACTION: Run: |
| :---- |
| AWS\_PROFILE=cloudtrail-lab agentcore deploy \--local |

You should see a message that the server is running on localhost:8080. Leave this terminal window open.

### **Open a second terminal window and send a test event:**

On Mac, press Command \+ T to open a new terminal tab, or open a fresh Terminal window.

| 💻  TERMINAL ACTION: In the new terminal window, run: |
| :---- |
| curl \-X POST http://localhost:8080/invocations \\     \-H 'Content-Type: application/json' \\     \-d '{         "event\_name": "AuthorizeSecurityGroupIngress",         "event\_time": "2026-03-01T14:32:00Z",         "user\_identity": {"userName": "test-user"},         "source\_ip": "203.0.113.42",         "resources": \["sg-0abc12345"\],         "raw\_event": {"groupId": "sg-0abc12345", "ipPermissions": \[{"fromPort": 22, "toPort": 22, "cidrIp": "0.0.0.0/0"}\]}     }' |

Watch your first terminal window — you should see the agent start reasoning through the event. Check your email for an investigation summary.

| 💡  TIP: If the local test works but you do not receive an email, double-check that you verified your email in Section 4.3. You can also check the terminal output to see what the agent did. |
| :---- |

### **Stop the local server:**

Go back to your first terminal window and press Ctrl \+ C to stop the local server.

## **8.6 Deploy to AWS**

| 💻  TERMINAL ACTION: In the same terminal window where you stopped the local server (make sure your virtual environment is still active), run: |
| :---- |
| AWS\_PROFILE=cloudtrail-lab agentcore deploy |

This will take 3-5 minutes. AWS is packaging your code, building a container, and deploying it to the AgentCore runtime.

| 🔴  IMPORTANT: When the deployment finishes, it will print your Agent ARN. It looks like: arn:aws:bedrock-agentcore:us-east-1:123456789012:runtime-endpoint/XXXXXXXXXX. Copy this value and save it — you will need it in the next section. |
| :---- |

# **9\. Create the Lambda Function**

Your Lambda function is the bridge between EventBridge (which detects the security group change) and your AgentCore agent (which investigates it). When EventBridge detects a relevant CloudTrail event, it triggers this Lambda function, which passes the event details to your agent.

| 🎓  WHY ARE WE DOING THIS? You might wonder why you need a Lambda function at all — why not have EventBridge call the agent directly? EventBridge is designed to trigger functions that respond quickly. Your agent needs time to think, query logs, and send emails, which can take 30-60 seconds. The Lambda function receives the event instantly, acknowledges it to EventBridge, and then invokes the agent asynchronously in the background. |
| :---- |

## **9.1 Create the Lambda Code**

Create a new folder for your Lambda function:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| mkdir \~/Desktop/cloudtrail-lab/lambda cd \~/Desktop/cloudtrail-lab/lambda |

In VS Code, create a new file with the following content and save it as lambda\_function.py inside your cloudtrail-lab/lambda folder:

| """ CloudTrail Event Bridge Lambda Receives security events from EventBridge and invokes the AgentCore agent. """ import json import boto3 import os from datetime import datetime AGENT\_ARN \= os.environ\["AGENT\_ARN"\] def lambda\_handler(event, context):     """     This function receives CloudTrail events from EventBridge and     passes the relevant details to the AgentCore investigation agent.     """     print(f"Received event: {json.dumps(event, default=str)}")     try:         detail     \= event.get("detail", {})         event\_name \= detail.get("eventName", "UnknownEvent")         event\_time \= detail.get("eventTime", datetime.utcnow().isoformat())         user\_ident \= detail.get("userIdentity", {})         source\_ip  \= detail.get("sourceIPAddress", "unknown")         resources  \= detail.get("resources", \[\])         req\_params \= detail.get("requestParameters", {})         resource\_ids \= \[\]         if req\_params.get("groupId"):             resource\_ids.append(req\_params\["groupId"\])         for r in resources:             if r.get("ARN"):                 resource\_ids.append(r\["ARN"\])         agent\_payload \= {             "event\_name":    event\_name,             "event\_time":    event\_time,             "user\_identity": user\_ident,             "source\_ip":     source\_ip,             "resources":     resource\_ids,             "raw\_event":     {                 "eventName":          event\_name,                 "requestParameters":  req\_params,                 "userIdentity":       user\_ident,                 "sourceIPAddress":    source\_ip             }         }         session\_id \= f"event-{event\_name}-{datetime.utcnow().strftime('%Y%m%d%H%M%S')}"         agent\_client \= boto3.client(             "bedrock-agent-runtime",             region\_name="us-east-1"         )         \# Extract the agent endpoint ID from the ARN         \# ARN format: arn:aws:bedrock-agentcore:region:account:runtime-endpoint/ENDPOINT\_ID         endpoint\_id \= AGENT\_ARN.split("/")\[-1\]         agent\_client.invoke\_agent\_runtime\_endpoint(             agentRuntimeEndpointArn=AGENT\_ARN,             sessionId=session\_id,             request=json.dumps(agent\_payload)         )         print(f"Successfully invoked agent for event: {event\_name}")         return {             "statusCode": 200,             "body": json.dumps({                 "status":     "Agent invoked",                 "event\_name": event\_name,                 "session\_id": session\_id             })         }     except Exception as e:         print(f"Error invoking agent: {str(e)}")         return {             "statusCode": 500,             "body": json.dumps({"error": str(e)})         } |
| :---- |

## **9.2 Package and Deploy the Lambda Function**

| 💻  TERMINAL ACTION: Navigate to your lambda folder and create a zip file of your code: |
| :---- |
| cd \~/Desktop/cloudtrail-lab/lambda zip lambda\_function.zip lambda\_function.py |

Now deploy the Lambda function to AWS. Replace YOUR\_ACCOUNT\_ID with your account ID and YOUR\_AGENT\_ARN with the Agent ARN you saved in Section 8.6:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws lambda create-function \\     \--profile cloudtrail-lab \\     \--function-name cloudtrail-event-investigator \\     \--runtime python3.12 \\     \--handler lambda\_function.lambda\_handler \\     \--role arn:aws:iam::YOUR\_ACCOUNT\_ID:role/CloudTrailInvestigatorLambdaRole \\     \--zip-file fileb://lambda\_function.zip \\     \--timeout 60 \\     \--memory-size 256 \\     \--environment Variables={AGENT\_ARN=YOUR\_AGENT\_ARN} \\     \--region us-east-1 |

Verify the Lambda was created:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws lambda get-function \\     \--profile cloudtrail-lab \\     \--function-name cloudtrail-event-investigator \\     \--region us-east-1 \\     \--query Configuration.FunctionArn \\     \--output text |

You should see the Lambda function ARN printed. Save this — you will need it in the next section.

# **10\. Set Up the EventBridge Rule**

EventBridge is the service that watches your CloudTrail logs and fires a trigger when it detects a security group change. Think of it like a tripwire — when someone touches a specific type of action, it alerts your Lambda function.

| 🎓  WHY ARE WE DOING THIS? CloudTrail logs every single action in your AWS account — potentially thousands of events per day. You don't want your agent investigating every one of them. EventBridge lets you write a precise filter pattern that selects only the specific types of events you care about. In this case, you care about security group changes. |
| :---- |

## **10.1 Create the EventBridge Rule**

In VS Code, create a new file with the following content and save it as event-pattern.json inside your cloudtrail-lab folder:

| {     "source": \["aws.ec2"\],     "detail-type": \["AWS API Call via CloudTrail"\],     "detail": {         "eventSource": \["ec2.amazonaws.com"\],         "eventName": \[             "AuthorizeSecurityGroupIngress",             "AuthorizeSecurityGroupEgress",             "RevokeSecurityGroupIngress",             "RevokeSecurityGroupEgress",             "CreateSecurityGroup",             "DeleteSecurityGroup"         \]     } } |
| :---- |

This pattern tells EventBridge: watch for any CloudTrail event that comes from the EC2 service AND is one of the security group action types listed.

| 💻  TERMINAL ACTION: Now navigate back to your cloudtrail-lab folder and create the rule: |
| :---- |
| cd \~/Desktop/cloudtrail-lab aws events put-rule \\     \--profile cloudtrail-lab \\     \--name cloudtrail-security-group-changes \\     \--event-pattern file://event-pattern.json \\     \--state ENABLED \\     \--description "Watches for security group changes via CloudTrail" \\     \--region us-east-1 |

## **10.2 Grant EventBridge Permission to Invoke Lambda**

EventBridge needs explicit permission to trigger your Lambda function. Replace YOUR\_ACCOUNT\_ID with your account ID:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws lambda add-permission \\     \--profile cloudtrail-lab \\     \--function-name cloudtrail-event-investigator \\     \--statement-id allow-eventbridge \\     \--action lambda:InvokeFunction \\     \--principal events.amazonaws.com \\     \--source-arn arn:aws:events:us-east-1:YOUR\_ACCOUNT\_ID:rule/cloudtrail-security-group-changes |

## **10.3 Connect the EventBridge Rule to Your Lambda Function**

Replace YOUR\_ACCOUNT\_ID with your account ID:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws events put-targets \\     \--profile cloudtrail-lab \\     \--rule cloudtrail-security-group-changes \\     \--targets Id=lambda-target,Arn=arn:aws:lambda:us-east-1:YOUR\_ACCOUNT\_ID:function:cloudtrail-event-investigator |

## **10.4 Verify the Rule Is Active**

| 💻  TERMINAL ACTION: Run this to confirm the rule is active and connected: |
| :---- |
| aws events describe-rule \\     \--profile cloudtrail-lab \\     \--name cloudtrail-security-group-changes \\     \--region us-east-1 \\     \--query '{Name:Name, State:State, EventPattern:EventPattern}' |

You should see State: ENABLED in the output. Your system is now live and watching for security group changes.

# **11\. Testing Your System**

Your system is now fully deployed. Time to test it by making a deliberate security group change and watching the agent spring into action.

## **11.1 Create a Test Security Group**

First, create a security group specifically for testing. This keeps your test changes separate from anything important in your account.

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws ec2 create-security-group \\     \--profile cloudtrail-lab \\     \--group-name lab-test-firewall \\     \--description "Test security group for CloudTrail lab" \\     \--region us-east-1 |

Note the GroupId from the output (it looks like sg-0abc12345). You will use this in the next step.

## **11.2 Trigger the Agent (The Fun Part)**

Now make a deliberate firewall rule change — adding a rule that allows SSH access from anywhere on the internet. This is the type of change your agent is watching for.

| 💻  TERMINAL ACTION: Replace sg-YOUR-GROUP-ID with the GroupId from the previous step, then run: |
| :---- |
| aws ec2 authorize-security-group-ingress \\     \--profile cloudtrail-lab \\     \--group-id sg-YOUR-GROUP-ID \\     \--protocol tcp \\     \--port 22 \\     \--cidr 0.0.0.0/0 \\     \--region us-east-1 |

This command adds a rule allowing any IP address to connect to port 22 (SSH — the way you remotely log into a server). This is a classic security misconfiguration.

## **11.3 Immediately Revoke the Test Rule**

As soon as you have confirmed the command ran successfully, remove the rule you just created. Although your test security group is not attached to any EC2 instances (so nothing is actually exposed right now), it is good practice to clean up overly permissive rules immediately rather than leaving them sitting in your account.

| 💻  TERMINAL ACTION: Run this command right away, replacing sg-YOUR-GROUP-ID with your security group ID: |
| :---- |
| aws ec2 revoke-security-group-ingress \\     \--profile cloudtrail-lab \\     \--group-id sg-YOUR-GROUP-ID \\     \--protocol tcp \\     \--port 22 \\     \--cidr 0.0.0.0/0 \\     \--region us-east-1 |
| **💡  TIP:** Even though your test security group has no EC2 instances attached to it — meaning nothing was actually exposed to the internet — removing the rule immediately builds good security habits. In a real environment, you would want to catch and remediate these changes in seconds, not minutes. |

The revocation of this rule will also trigger your agent a second time, this time classifying the event as INFORMATIONAL since removing an overly permissive rule is a security improvement, not a risk. You will receive a second email with a different severity assessment.

## **11.4 Wait for the Agent to Respond**

CloudTrail logs are delivered to EventBridge within a few seconds, but there can sometimes be a delay of up to 5-15 minutes before the event arrives. This is normal behavior for how AWS delivers CloudTrail events.

| ⚠️   NOTE: If you do not see an email within 15 minutes, use the troubleshooting steps in Section 12 to investigate. |
| :---- |

While you wait, you can watch the Lambda function logs to see when it fires:

| 💻  TERMINAL ACTION: Run this command to watch your Lambda logs in real time. Press Ctrl+C to stop watching when you are done. |
| :---- |
| aws logs tail \\     \--profile cloudtrail-lab \\     /aws/lambda/cloudtrail-event-investigator \\     \--follow |

## **11.5 Verify the Security Hub Finding**

Once the agent runs, check that a Security Hub finding was created:

| 💻  TERMINAL ACTION: Run: |
| :---- |
| aws securityhub get-findings \\     \--profile cloudtrail-lab \\     \--filters '{"GeneratorId": \[{"Value": "cloudtrail-investigator-agent", "Comparison": "EQUALS"}\]}' \\     \--region us-east-1 \\     \--query 'Findings\[0\].{Title:Title,Severity:Severity.Label,Time:CreatedAt}' |

You should see the title, severity level, and timestamp of the finding your agent created.

## **11.6 View the Finding in the Console**

| 🖥️  AWS CONSOLE ACTION: 1\. Go to console.aws.amazon.com and search for Security Hub. 2\. Click Findings in the left navigation. 3\. Look for findings with Generator ID containing 'cloudtrail-investigator-agent'. 4\. Click on a finding to see the full details including the agent's description. |
| :---- |

## **11.7 Test 2: Confirm the Revocation Was Also Investigated**

When you ran the revoke command in Section 11.3, that action was also a CloudTrail event and your EventBridge rule caught it too. Check your inbox for a second email — this one should have a much lower severity (INFORMATIONAL) since removing an overly permissive rule is a positive security action, not a risk.

This demonstrates an important aspect of your agent's reasoning capability — it is not just pattern matching on event names. It is evaluating the context of what changed and whether the change made your environment more or less secure. Two events with the same event name (a security group rule change) produce very different assessments depending on what the change actually did.

# **12\. Troubleshooting**

If something is not working as expected, use this section to diagnose and fix the issue.

| Problem | How to Fix It |
| :---- | :---- |
| No email received after 15 minutes | 1\. Check Lambda logs (Section 11.3 command). 2\. Verify your email is confirmed in SES (check for the verification email). 3\. Confirm the EventBridge rule is ENABLED. |
| Lambda logs show 'Agent ARN not found' | Re-check the AGENT\_ARN environment variable on your Lambda. Run the agentcore deploy command again if needed to get the correct ARN. |
| Error: AccessDenied when deploying agent | Verify your IAM role has the correct policies attached from Section 5\. Re-run the attach-role-policy commands. |
| 'agentcore' command not found | Your virtual environment is not active. Run: source \~/Desktop/cloudtrail-lab/agent/.venv/bin/activate |
| Bedrock model access error | Go to the Bedrock console and verify Claude Sonnet 4 shows 'Access granted' in Model access. It can take a few minutes after approval. |
| SES 'Email address not verified' error | Open your email inbox, find the AWS verification email, and click the link. Then re-run the Secrets Manager command with your verified address. |
| Large zip file error (135MB+) | Your virtual environment or build cache is being included. Make sure you only zip lambda\_function.py, not the whole folder. |
| EventBridge rule shows ENABLED but nothing fires | CloudTrail events can take up to 15 minutes. Also confirm the put-targets command ran successfully by checking: aws events list-targets-by-rule \--rule cloudtrail-security-group-changes \--profile cloudtrail-lab |
| Python 'ModuleNotFoundError' on deploy | Run pip install \-r requirements.txt inside your active virtual environment before deploying. |

## **How to Check CloudWatch Logs (The Most Useful Debugging Tool)**

When something goes wrong, CloudWatch logs are your best friend. They show exactly what your Lambda function and agent did step by step.

| 💻  TERMINAL ACTION: To view Lambda logs: |
| :---- |
| aws logs tail \\     \--profile cloudtrail-lab \\     /aws/lambda/cloudtrail-event-investigator \\     \--since 1h |
| **💡  TIP:** The \--since 1h flag shows logs from the past hour. You can change it to 30m for 30 minutes, or 2h for 2 hours. |

# **13\. Clean Up (Important\!)**

When you are finished with the lab, it is important to remove the resources you created. Some AWS services charge by the hour or by usage, and leaving resources running can result in unexpected charges.

| 🔴  IMPORTANT: Run all of the following commands to clean up your lab environment. Replace YOUR\_ACCOUNT\_ID with your account ID throughout. |
| :---- |

## **13.1 Delete the EventBridge Rule**

| 💻  TERMINAL ACTION: Remove the target first, then the rule: |
| :---- |
| aws events remove-targets \\     \--profile cloudtrail-lab \\     \--rule cloudtrail-security-group-changes \\     \--ids lambda-target |
| aws events delete-rule \\     \--profile cloudtrail-lab \\     \--name cloudtrail-security-group-changes |

## **13.2 Delete the Lambda Function**

| aws lambda delete-function \\     \--profile cloudtrail-lab \\     \--function-name cloudtrail-event-investigator |
| :---- |

## **13.3 Delete the Test Security Group**

| 💻  TERMINAL ACTION: Replace sg-YOUR-GROUP-ID with your test security group ID: |
| :---- |
| aws ec2 delete-security-group \\     \--profile cloudtrail-lab \\     \--group-id sg-YOUR-GROUP-ID |

## **13.4 Delete the AgentCore Agent**

| 💻  TERMINAL ACTION: Run: |
| :---- |
| AWS\_PROFILE=cloudtrail-lab agentcore delete |

## **13.5 Delete Secrets Manager Secrets**

| aws secretsmanager delete-secret \\     \--profile cloudtrail-lab \\     \--secret-id cloudtrail-investigator/notifications \\     \--force-delete-without-recovery |
| :---- |

## **13.6 Delete IAM Roles and Policies**

| aws iam detach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorAgentRole \\     \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorAgentPolicy |
| :---- |
| aws iam detach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorLambdaRole \\     \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorLambdaPolicy |
| aws iam detach-role-policy \\     \--profile cloudtrail-lab \\     \--role-name CloudTrailInvestigatorLambdaRole \\     \--policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole |
| aws iam delete-role \--profile cloudtrail-lab \--role-name CloudTrailInvestigatorAgentRole aws iam delete-role \--profile cloudtrail-lab \--role-name CloudTrailInvestigatorLambdaRole aws iam delete-policy \--profile cloudtrail-lab \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorAgentPolicy aws iam delete-policy \--profile cloudtrail-lab \--policy-arn arn:aws:iam::YOUR\_ACCOUNT\_ID:policy/CloudTrailInvestigatorLambdaPolicy |

# **14\. Deployment Checklist**

Use this checklist to confirm every step is complete before testing:

| Done | Step |
| :---- | :---- |
| ☐ | Homebrew installed (Mac) or WSL configured (Windows) |
| ☐ | Python 3.12 or higher installed — verified with: python3 \--version |
| ☐ | AWS CLI installed — verified with: aws \--version |
| ☐ | uv package manager installed — verified with: uv \--version |
| ☐ | VS Code installed |
| ☐ | AWS account created or confirmed accessible |
| ☐ | IAM user cloudtrail-lab created with AdministratorAccess |
| ☐ | Access key created and saved securely |
| ☐ | AWS CLI profile cloudtrail-lab configured — verified with: aws sts get-caller-identity \--profile cloudtrail-lab |
| ☐ | Amazon Bedrock enabled with Claude Sonnet 4 model access granted |
| ☐ | AWS Security Hub enabled in us-east-1 |
| ☐ | Email address verified in Amazon SES (verification link clicked) |
| ☐ | Folder \~/Desktop/cloudtrail-lab created |
| ☐ | IAM role CloudTrailInvestigatorAgentRole created with policy attached |
| ☐ | IAM role CloudTrailInvestigatorLambdaRole created with policies attached |
| ☐ | Secrets Manager secret cloudtrail-investigator/notifications created |
| ☐ | Agent code files created: agent.py, requirements.txt, tools/\_\_init\_\_.py, tools/cloudtrail\_tools.py, tools/ec2\_tools.py, tools/notification\_tools.py, tools/security\_hub\_tools.py |
| ☐ | Python virtual environment created and activated (see (.venv) in terminal prompt) |
| ☐ | pip install \-r requirements.txt completed successfully |
| ☐ | agentcore configure completed — agent named CloudTrailInvestigator |
| ☐ | Local test passed — email received from agentcore deploy \--local |
| ☐ | agentcore deploy completed — Agent ARN saved |
| ☐ | Lambda code file lambda\_function.py created |
| ☐ | Lambda function cloudtrail-event-investigator deployed with AGENT\_ARN environment variable |
| ☐ | Lambda function ARN saved |
| ☐ | EventBridge pattern file event-pattern.json created |
| ☐ | EventBridge rule cloudtrail-security-group-changes created and ENABLED |
| ☐ | Lambda permission granted to EventBridge |
| ☐ | EventBridge target connected to Lambda function |
| ☐ | Test security group created |
| ☐ | Test 1 passed: Added 0.0.0.0/0 port 22 rule → received HIGH/CRITICAL email alert |
| ☐ | Test 2 passed: Removed the rule → received INFORMATIONAL email |
| ☐ | Security Hub finding visible in the AWS console |

**Lab Complete**  
*You have built an AI-powered security event investigator using AWS AgentCore and Amazon Bedrock.*
