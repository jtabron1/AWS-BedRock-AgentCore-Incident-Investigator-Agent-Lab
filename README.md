# AWS AgentCore Hands-On Lab
## CloudTrail Security Event Investigator

> Build an AI agent that watches your AWS account for suspicious activity, investigates what happened, and explains it in plain English.

| | |
|---|---|
| **Skill Level** | Beginner-Friendly |
| **Estimated Time** | 2-3 Hours |
| **Prior Experience** | None Required |
| **AWS Services** | AgentCore, Bedrock (Claude), Lambda, EventBridge, CloudTrail, SES, Security Hub, IAM, Secrets Manager |

---

## Table of Contents

- [Section 0 — Before You Begin: Terminal Basics](#0-before-you-begin-terminal-basics)
- [Section 1 — What You Will Build](#1-what-you-will-build)
- [Section 2 — Setting Up Your Computer](#2-setting-up-your-computer)
- [Section 3 — AWS Account Setup](#3-aws-account-setup)
- [Section 4 — Enable Required AWS Services](#4-enable-required-aws-services)
- [Section 5 — Create Permissions for Your Agent and Lambda](#5-create-permissions-for-your-agent-and-lambda)
- [Section 6 — Store Your Email Configuration Securely](#6-store-your-email-configuration-securely)
- [Section 7 — Create the Agent Code](#7-create-the-agent-code)
- [Section 8 — Deploy the Agent](#8-deploy-the-agent)
- [Section 9 — Create the Lambda Function](#9-create-the-lambda-function)
- [Section 10 — Set Up the EventBridge Rule](#10-set-up-the-eventbridge-rule)
- [Section 11 — Testing Your System](#11-testing-your-system)
- [Section 12 — Troubleshooting](#12-troubleshooting)
- [Section 13 — Clean Up](#13-clean-up-important)
- [Section 14 — Deployment Checklist](#14-deployment-checklist)

---

## 0. Before You Begin: Terminal Basics

This lab involves typing commands into a program called the terminal (also called the command line or command prompt). If you have never used a terminal before, this section will get you comfortable with the basics.

### 0.1 What Is the Terminal?

The terminal is a text-based way to give your computer instructions. Instead of clicking buttons and icons, you type commands and press Enter. You will only need a small handful of commands for this lab and every command is explained as you go.

### 0.2 How to Open a Terminal

**On a Mac:**
1. Press **Command (⌘) + Space** to open Spotlight Search.
2. Type `Terminal` and press Enter.
3. A window with a blinking cursor will appear. That is your terminal.

**On Windows:**
1. Press the **Windows key**.
2. Type `PowerShell` and click **Windows PowerShell**.
3. A blue or dark window will appear. That is your terminal.

> ⚠️ **NOTE:** This lab was written with Mac users in mind. If you are on Windows, most commands will work the same way, but some installation steps differ. Notes for Windows users are included throughout.

### 0.3 Understanding the Terminal Prompt

When your terminal is ready to accept a command, you will see a line ending in `$` or `%` followed by a blinking cursor. This is called the **prompt**. You do not need to type the prompt — just type your command after it and press **Enter** to run it.

```
yourname@MacBook-Pro ~ %
```

### 0.4 How to Run a Command

1. Click on your terminal window to make sure it is selected.
2. Type the command exactly as shown in the lab. Capitalization and spacing matter.
3. Press **Enter** to run the command.
4. Wait for the command to finish. You will know it is done when the prompt reappears.

> 💡 **TIP:** If a command seems to be taking a long time, it is probably still running. Do not close the terminal — just wait for the prompt to come back.

### 0.5 Common Terminal Commands Reference

| Command | What It Does | Example |
|---|---|---|
| `pwd` | Shows which folder you are currently in | `pwd` |
| `ls` | Lists all files and folders in your current location | `ls` |
| `cd [folder]` | Moves you into a folder | `cd Desktop` |
| `cd ..` | Moves you back one folder level | `cd ..` |
| `mkdir [name]` | Creates a new folder | `mkdir my-lab` |
| `cat [file]` | Displays the contents of a file | `cat agent.py` |
| `clear` | Clears all the text off the terminal screen | `clear` |
| `↑ (Up arrow)` | Recalls your previous command | press ↑ |
| `Tab` | Auto-completes a file or folder name | `cd Desk[Tab]` |
| `Ctrl + C` | Cancels the current running command | press Ctrl+C |

### 0.6 What Does a Backslash (`\`) Mean in Commands?

The backslash `\` at the end of a line means "this command is not finished yet — continue reading on the next line." It is purely for readability. These two commands do exactly the same thing:

```bash
aws iam create-role --role-name MyRole --assume-role-policy-document file://policy.json
```

And the easier-to-read version:

```bash
aws iam create-role \
    --role-name MyRole \
    --assume-role-policy-document file://policy.json
```

> 🔴 **IMPORTANT:** When copying a command that uses backslashes, make sure you copy the entire block including every line. If you miss a line, the command will be incomplete.

### 0.7 Installing a Code Editor

Several steps in this lab require you to create Python code files. Install **Visual Studio Code (VS Code)** — a free, beginner-friendly code editor.

1. Go to [code.visualstudio.com](https://code.visualstudio.com) in your web browser.
2. Click the **Download** button for your operating system.
3. Open the downloaded file and follow the installation instructions.

---

## 1. What You Will Build

### 1.1 The Big Picture

You will build a system where an AI agent watches your AWS account at all times. The moment a firewall rule changes, the agent wakes up, reads the details of what happened, pulls related activity from the last hour to see the full picture, reasons about whether it looks suspicious, and then sends you a plain-English email summary explaining exactly what happened and what the risk is.

No rules to configure. No dashboards to check. Just an intelligent agent that does the investigation and explains it to you.

### 1.2 How the System Works Step by Step

| Step | Component | What Happens |
|---|---|---|
| 1 | CloudTrail | AWS automatically records every action taken in your account. |
| 2 | EventBridge | A rule watches CloudTrail for firewall-related events. When it sees one, it triggers your Lambda function. |
| 3 | Lambda Function | A small piece of code that wakes up when triggered and passes the event details to your AI agent. |
| 4 | AgentCore Agent | Your AI agent (powered by Claude on Amazon Bedrock) receives the event, uses tools to investigate, and reasons about what happened. |
| 5 | Email via SES | The agent sends you a plain-English email with a summary of what happened and an assessment of the risk. |
| 6 | Security Hub | The agent logs an official security finding for your records. |

### 1.3 What You Will Learn

- How to deploy an AI agent to AWS AgentCore using the command line
- How to connect AWS services together using EventBridge and Lambda
- How to give your agent tools so it can take actions (query logs, send emails, create findings)
- How AWS CloudTrail logs every action in your account and how to query those logs
- How to store sensitive configuration values securely using Secrets Manager
- How to verify everything is working end-to-end

### 1.4 What Will Trigger the Agent During Testing

To test your system, you will intentionally make a firewall rule change — adding a rule that allows traffic from anywhere on the internet. You will then watch as your agent detects it, investigates it, and sends you an email — all automatically, within a few minutes.

> 🎓 **WHY ARE WE DOING THIS?** Security groups are AWS's version of a firewall. A rule that allows traffic from `0.0.0.0/0` (any IP address on the internet) on a sensitive port is a common security misconfiguration that attackers look for.

---

## 2. Setting Up Your Computer

### 2.1 Install Homebrew (Mac Only)

Homebrew is a package manager for Mac — think of it as an app store for developer tools that you install from the terminal.

> 💻 **TERMINAL ACTION:** Open your terminal and run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify it worked:

```bash
brew --version
```

> ⚠️ **NOTE:** Windows users: You do not need Homebrew. The recommended approach is to install Windows Subsystem for Linux (WSL). Search for "Install WSL Windows" on Microsoft's website, then proceed with the Mac/Linux commands inside your WSL terminal.

### 2.2 Install Python 3.10

Python is the programming language used to write the agent code in this lab. AWS AgentCore requires **Python 3.10 specifically** — newer versions such as 3.12 or 3.13 will cause installation errors later in the lab.

> 💻 **TERMINAL ACTION:** Check which version of Python you have:

```bash
python3 --version
```

If you see `Python 3.10.x`, you are good — skip to Section 2.3.

If you see any other version or an error, install Python 3.10:

**Mac:**
```bash
brew install python@3.10
```

Verify it installed:
```bash
python3.10 --version
```

**Windows:** Go to [python.org/downloads/release/python-31011](https://python.org/downloads/release/python-31011), download the Windows installer for Python 3.10, and run it. Check the box that says **Add Python to PATH**.

> 🔴 **IMPORTANT:** Do not install a newer Python version thinking it will work better. AgentCore's starter toolkit is pinned to Python 3.10 and will fail to install on 3.11, 3.12, or 3.13.

### 2.3 Install the AWS CLI

The AWS CLI lets you interact with AWS from your terminal.

> 💻 **TERMINAL ACTION:** Check if it is already installed:

```bash
aws --version
```

If you see `aws-cli/2.x.x`, it is already installed — skip to Section 2.4.

**Mac:**
```bash
brew install awscli
```

**Windows:** Go to [docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and download the Windows installer.

### 2.4 Install the uv Package Manager

uv is a fast Python package manager that AWS AgentCore requires for deploying your agent code.

> 💻 **TERMINAL ACTION:** Run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Apply the changes to your terminal session:

**Mac / Linux:**
```bash
source ~/.zshrc
```

If that does not work, try:
```bash
source ~/.bashrc
```

Verify uv is installed:
```bash
uv --version
```

---

## 3. AWS Account Setup

### 3.1 Create an AWS Account (If You Don't Have One)

If you already have an AWS account, skip to Section 3.2.

1. Go to [aws.amazon.com](https://aws.amazon.com) and click **Create an AWS Account**.
2. Follow the sign-up process. You will need a credit card, but this lab stays within the AWS Free Tier.
3. Once your account is active, sign in to the AWS Management Console at [console.aws.amazon.com](https://console.aws.amazon.com).

### 3.2 Create a Dedicated IAM User for This Lab

> 🎓 **WHY ARE WE DOING THIS?** Your root account has unlimited access to everything in AWS. Using it for day-to-day work is like using the master key to your building for every errand. A dedicated IAM user lets you scope permissions appropriately.

**Step 1: Create the user**

> 💻 **TERMINAL ACTION:** Run:

```bash
aws iam create-user \
    --user-name cloudtrail-lab
```

**Step 2: Attach administrator permissions**

```bash
aws iam attach-user-policy \
    --user-name cloudtrail-lab \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

**Step 3: Create access keys**

```bash
aws iam create-access-key \
    --user-name cloudtrail-lab
```

> 🔴 **IMPORTANT:** Copy both the `AccessKeyId` and `SecretAccessKey` values right now and save them somewhere safe. The SecretAccessKey is only shown once.

**Step 4: Configure your AWS CLI profile**

```bash
aws configure --profile cloudtrail-lab
```

When prompted, enter:

| Prompt | What to Enter |
|---|---|
| AWS Access Key ID | Your AccessKeyId from Step 3 |
| AWS Secret Access Key | Your SecretAccessKey from Step 3 |
| Default region name | `us-east-1` |
| Default output format | `json` |

**Step 5: Verify your profile works**

```bash
aws sts get-caller-identity \
    --profile cloudtrail-lab
```

> 💡 **TIP:** The `--profile cloudtrail-lab` part that appears in AWS CLI commands throughout this lab tells the CLI which set of credentials to use. Always include it.

---

## 4. Enable Required AWS Services

### 4.1 Enable Amazon Bedrock Model Access

> 🖥️ **AWS CONSOLE ACTION:**
> 1. Go to [console.aws.amazon.com](https://console.aws.amazon.com) and search for **Bedrock**.
> 2. In the left navigation panel, click **Model access**.
> 3. Click **Modify model access**.
> 4. Find **Anthropic** and check the box next to **Claude Sonnet 4.6**.
> 5. Click **Next**, then **Submit**.
> 6. Wait for the Status column to show **Access granted** (1-2 minutes).

### 4.2 Enable AWS Security Hub

> 💻 **TERMINAL ACTION:** Run:

```bash
aws securityhub enable-security-hub \
    --profile cloudtrail-lab \
    --region us-east-1 \
    --enable-default-standards
```

> ⚠️ **NOTE:** If you see an error that Security Hub is already enabled, that is fine — continue to the next step.

### 4.3 Verify Your Email for SES

> 💻 **TERMINAL ACTION:** Replace `your-email@domain.com` with your actual email address and run:

```bash
aws ses verify-email-identity \
    --profile cloudtrail-lab \
    --email-address your-email@domain.com \
    --region us-east-1
```

> 🔴 **IMPORTANT:** Check your inbox right now and click the verification link AWS just sent you. Your agent cannot send emails until this step is done. The email comes from `no-reply-aws@amazon.com`.

---

## 5. Create Permissions for Your Agent and Lambda

> 🎓 **WHY ARE WE DOING THIS?** Think of IAM roles like job descriptions. A receptionist has permission to answer phones but not access financial records. Your Lambda function has permission to invoke your agent, but not to directly access your emails. This principle of least privilege is a security best practice.

### 5.1 Create the AgentCore Execution Role

**Step 1: Create your project folder**

> 💻 **TERMINAL ACTION:** Run:

```bash
mkdir ~/Desktop/cloudtrail-lab
cd ~/Desktop/cloudtrail-lab
```

**Step 2: Create a trust policy file**

In VS Code, create a new file with the following content and save it as `agent-trust-policy.json` inside your `cloudtrail-lab` folder:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "bedrock.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "bedrock-agentcore.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

> 🎓 **WHY ARE WE DOING THIS?** Two service principals are listed here because Bedrock and AgentCore are distinct services. `bedrock.amazonaws.com` covers general Bedrock model invocations. `bedrock-agentcore.amazonaws.com` is the service principal used when AgentCore deploys and manages your agent at runtime. Without both entries, the deployment will fail with a role validation error.

**Step 3: Create the role**

> 💻 **TERMINAL ACTION:** Make sure you are in your `cloudtrail-lab` folder, then run:

```bash
aws iam create-role \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorAgentRole \
    --assume-role-policy-document file://agent-trust-policy.json
```

**Step 4: Create a permissions policy file**

In VS Code, create a new file with the following content and save it as `agent-policy.json`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "CloudTrailReadAccess",
            "Effect": "Allow",
            "Action": [
                "cloudtrail:LookupEvents",
                "cloudtrail:GetTrailStatus",
                "cloudtrail:DescribeTrails"
            ],
            "Resource": "*"
        },
        {
            "Sid": "EC2ReadAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSecurityGroupRules"
            ],
            "Resource": "*"
        },
        {
            "Sid": "SecretsAccess",
            "Effect": "Allow",
            "Action": ["secretsmanager:GetSecretValue"],
            "Resource": "arn:aws:secretsmanager:*:*:secret:cloudtrail-investigator/*"
        },
        {
            "Sid": "SecurityHubAccess",
            "Effect": "Allow",
            "Action": ["securityhub:BatchImportFindings"],
            "Resource": "*"
        },
        {
            "Sid": "SESAccess",
            "Effect": "Allow",
            "Action": ["ses:SendEmail", "ses:SendRawEmail"],
            "Resource": "*"
        },
        {
            "Sid": "BedrockAccess",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:InvokeModelWithResponseStream"
            ],
            "Resource": "*"
        },
        {
            "Sid": "STSAccess",
            "Effect": "Allow",
            "Action": ["sts:GetCallerIdentity"],
            "Resource": "*"
        }
    ]
}
```

**Step 5: Create the policy in AWS and attach it to your role**

First, get your AWS account ID:

> 💻 **TERMINAL ACTION:** Run:

```bash
aws sts get-caller-identity \
    --profile cloudtrail-lab \
    --query Account \
    --output text
```

Then create and attach the policy (replace `YOUR_ACCOUNT_ID`):

```bash
aws iam create-policy \
    --profile cloudtrail-lab \
    --policy-name CloudTrailInvestigatorAgentPolicy \
    --policy-document file://agent-policy.json
```

```bash
aws iam attach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorAgentRole \
    --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorAgentPolicy
```

### 5.2 Create the Lambda Execution Role

**Step 1: Create a trust policy file for Lambda**

In VS Code, create a new file with the following content and save it as `lambda-trust-policy.json`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

**Step 2: Create the Lambda role**

> 💻 **TERMINAL ACTION:** Run:

```bash
aws iam create-role \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorLambdaRole \
    --assume-role-policy-document file://lambda-trust-policy.json
```

**Step 3: Create a permissions policy for Lambda**

In VS Code, create a new file with the following content and save it as `lambda-policy.json`:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AgentCoreInvoke",
            "Effect": "Allow",
            "Action": ["bedrock:InvokeAgent"],
            "Resource": "*"
        },
        {
            "Sid": "CloudWatchLogs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

> 💻 **TERMINAL ACTION:** Create and attach the Lambda policy (replace `YOUR_ACCOUNT_ID`):

```bash
aws iam create-policy \
    --profile cloudtrail-lab \
    --policy-name CloudTrailInvestigatorLambdaPolicy \
    --policy-document file://lambda-policy.json
```

```bash
aws iam attach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorLambdaRole \
    --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorLambdaPolicy
```

```bash
aws iam attach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

## 6. Store Your Email Configuration Securely

> 🎓 **WHY ARE WE DOING THIS?** Think of Secrets Manager like a locked filing cabinet. Your agent knows where the cabinet is and has a key to open it, but the sensitive values are never sitting out in the open in your code.

> 💻 **TERMINAL ACTION:** Replace both email addresses with your own verified SES email address from Section 4.3:

```bash
aws secretsmanager create-secret \
    --profile cloudtrail-lab \
    --name cloudtrail-investigator/notifications \
    --secret-string '{"sender_email": "your-email@domain.com", "recipient_email": "your-email@domain.com"}' \
    --region us-east-1
```

---

## 7. Create the Agent Code

Create your agent folder structure:

> 💻 **TERMINAL ACTION:** Run:

```bash
mkdir ~/Desktop/cloudtrail-lab/agent
mkdir ~/Desktop/cloudtrail-lab/agent/tools
cd ~/Desktop/cloudtrail-lab/agent
```

### 7.1 Create requirements.txt

In VS Code, create a new file with the following content and save it as `requirements.txt` inside your `cloudtrail-lab/agent` folder:

```
bedrock-agentcore>=0.1.0
bedrock-agentcore-starter-toolkit>=0.1.21
strands-agents>=0.1.0
strands-agents-tools>=0.1.0
boto3>=1.35.0
```

### 7.2 Create agent.py

In VS Code, create a new file with the following content and save it as `agent.py` inside your `cloudtrail-lab/agent` folder:

```python
"""
CloudTrail Security Event Investigator Agent
"""
from bedrock_agentcore import BedrockAgentCoreApp
from strands import Agent
from strands.models import BedrockModel

app = BedrockAgentCoreApp()


def get_agent():
    """
    Initialize the agent with lazy loading.
    We do this inside a function rather than at the top of the file
    to avoid slow startup times when the agent first wakes up.
    """
    from tools import (
        get_cloudtrail_events,
        get_security_group_details,
        send_investigation_email,
        create_security_finding
    )

    model = BedrockModel(
        model_id="us.anthropic.claude-sonnet-4-6"
    )

    return Agent(
        model=model,
        tools=[
            get_cloudtrail_events,
            get_security_group_details,
            send_investigation_email,
            create_security_finding
        ],
        system_prompt="""
        You are a cloud security investigator agent. Your job is to analyze
        AWS security events, understand what happened, and communicate it
        clearly to non-technical stakeholders.

        When you receive a security event:

        1. Use get_cloudtrail_events to look up what else the same user
           did in the 60 minutes before and after this event. Look for
           any patterns that suggest malicious or accidental behavior.

        2. If the event involves a security group change, use
           get_security_group_details to get more context about the
           affected firewall rule.

        3. Assess the severity:
           - CRITICAL: Changes that expose sensitive ports (22, 3389, 3306,
             5432) to the entire internet (0.0.0.0/0)
           - HIGH: Changes that expose any port to the entire internet
           - MEDIUM: Changes that open access to a broad but not universal
             IP range
           - LOW: Changes to egress rules or non-sensitive configurations
           - INFORMATIONAL: Rule removals that improve security posture

        4. Use send_investigation_email to send a clear, plain-English
           email. Write as if explaining to someone who is not technical.
           Include: what changed, who made the change, when, what the
           risk is, and what action (if any) to take.

        5. Use create_security_finding to log an official record in
           Security Hub.

        Be concise, clear, and avoid unnecessary jargon.
        """
    )


@app.entrypoint
def handler(payload, context):
    """Handle incoming security events from EventBridge via Lambda"""
    agent = get_agent()

    event_name    = payload.get("event_name", "UnknownEvent")
    event_time    = payload.get("event_time", "")
    user_identity = payload.get("user_identity", {})
    source_ip     = payload.get("source_ip", "unknown")
    resources     = payload.get("resources", [])
    raw_event     = payload.get("raw_event", {})

    username = (
        user_identity.get("userName")
        or user_identity.get("sessionContext", {}).get("sessionIssuer", {}).get("userName")
        or user_identity.get("arn", "unknown")
    )

    prompt = f"""
    A security event has occurred in this AWS account. Please investigate
    and report on it.

    Event Name: {event_name}
    Time: {event_time}
    Performed By: {username}
    Source IP Address: {source_ip}
    Affected Resources: {resources}

    Raw Event Details:
    {raw_event}

    Please investigate this event, check for related activity in the
    past hour from this user, assess the risk, send an email summary,
    and create a Security Hub finding.
    """

    result = agent(prompt)
    return {"status": "completed", "result": str(result)}


if __name__ == "__main__":
    app.run()
```

### 7.3 Create tools/\_\_init\_\_.py

In VS Code, create a new file with the following content and save it as `__init__.py` inside your `cloudtrail-lab/agent/tools` folder:

```python
"""
Agent Tools Package
"""
from tools.cloudtrail_tools import get_cloudtrail_events
from tools.ec2_tools import get_security_group_details
from tools.notification_tools import send_investigation_email
from tools.security_hub_tools import create_security_finding
```

### 7.4 Create tools/cloudtrail\_tools.py

Save this as `cloudtrail_tools.py` inside your `cloudtrail-lab/agent/tools` folder:

```python
"""
CloudTrail Tools - Query related security events
"""
import boto3
from strands import tool
from datetime import datetime, timedelta, timezone


@tool
def get_cloudtrail_events(username: str, minutes_back: int = 60) -> dict:
    """
    Look up recent CloudTrail events for a specific user.

    Args:
        username: The IAM username or ARN to look up
        minutes_back: How many minutes of history to retrieve (default 60)

    Returns:
        A dictionary containing recent events for that user
    """
    try:
        cloudtrail = boto3.client("cloudtrail", region_name="us-east-1")

        end_time   = datetime.now(timezone.utc)
        start_time = end_time - timedelta(minutes=minutes_back)

        response = cloudtrail.lookup_events(
            LookupAttributes=[
                {
                    "AttributeKey": "Username",
                    "AttributeValue": username
                }
            ],
            StartTime=start_time,
            EndTime=end_time,
            MaxResults=20
        )

        events = []
        for event in response.get("Events", []):
            events.append({
                "event_name":   event.get("EventName"),
                "event_time":   event.get("EventTime").isoformat() if event.get("EventTime") else None,
                "event_source": event.get("EventSource"),
                "resources":    [r.get("ResourceName") for r in event.get("Resources", [])]
            })

        return {
            "success":     True,
            "username":    username,
            "event_count": len(events),
            "time_window": f"Last {minutes_back} minutes",
            "events":      events
        }
    except Exception as e:
        return {"success": False, "error": str(e)}
```

### 7.5 Create tools/ec2\_tools.py

Save this as `ec2_tools.py` inside your `cloudtrail-lab/agent/tools` folder:

```python
"""
EC2 Tools - Get security group details
"""
import boto3
from strands import tool


@tool
def get_security_group_details(security_group_id: str) -> dict:
    """
    Retrieve details about a specific security group.

    Args:
        security_group_id: The security group ID (e.g. sg-12345abc)

    Returns:
        A dictionary with security group name, description, and rules
    """
    try:
        ec2 = boto3.client("ec2", region_name="us-east-1")

        response = ec2.describe_security_groups(
            GroupIds=[security_group_id]
        )

        groups = response.get("SecurityGroups", [])
        if not groups:
            return {"success": False, "error": f"Security group {security_group_id} not found"}

        group = groups[0]
        inbound_rules = []
        for rule in group.get("IpPermissions", []):
            for ip_range in rule.get("IpRanges", []):
                inbound_rules.append({
                    "protocol":    rule.get("IpProtocol"),
                    "from_port":   rule.get("FromPort"),
                    "to_port":     rule.get("ToPort"),
                    "cidr":        ip_range.get("CidrIp"),
                    "description": ip_range.get("Description", "No description")
                })

        return {
            "success":             True,
            "group_id":            group.get("GroupId"),
            "group_name":          group.get("GroupName"),
            "description":         group.get("Description"),
            "vpc_id":              group.get("VpcId"),
            "inbound_rules":       inbound_rules,
            "outbound_rule_count": len(group.get("IpPermissionsEgress", []))
        }
    except Exception as e:
        return {"success": False, "error": str(e)}
```

### 7.6 Create tools/notification\_tools.py

Save this as `notification_tools.py` inside your `cloudtrail-lab/agent/tools` folder:

```python
"""
Notification Tools - Send investigation emails via SES
"""
import boto3
import json
from strands import tool
from datetime import datetime


def get_notification_config():
    """Retrieve email configuration from Secrets Manager"""
    client = boto3.client("secretsmanager", region_name="us-east-1")
    response = client.get_secret_value(
        SecretId="cloudtrail-investigator/notifications"
    )
    return json.loads(response["SecretString"])


SEVERITY_COLORS = {
    "CRITICAL":      "#e53e3e",
    "HIGH":          "#dd6b20",
    "MEDIUM":        "#d69e2e",
    "LOW":           "#3182ce",
    "INFORMATIONAL": "#38a169"
}


@tool
def send_investigation_email(
    subject: str,
    what_happened: str,
    who_did_it: str,
    when_it_happened: str,
    risk_assessment: str,
    recommended_action: str,
    severity: str
) -> dict:
    """
    Send a plain-English security investigation summary email.

    Args:
        subject: Email subject line
        what_happened: Clear description of the event in plain English
        who_did_it: Who performed the action (username, IP address)
        when_it_happened: Timestamp of the event
        risk_assessment: What the risk is and why it matters
        recommended_action: What the recipient should do
        severity: CRITICAL, HIGH, MEDIUM, LOW, or INFORMATIONAL

    Returns:
        Dictionary with success status and email message ID
    """
    try:
        config = get_notification_config()
        ses    = boto3.client("ses", region_name="us-east-1")

        color = SEVERITY_COLORS.get(severity.upper(), "#718096")

        html = f"""
        <html>
        <body style="font-family: Arial, sans-serif; max-width: 640px;
                     margin: 0 auto; color: #2d3748;">
          <div style="background: {color}; padding: 24px; border-radius: 8px 8px 0 0;">
            <h1 style="color: white; margin: 0; font-size: 22px;">Security Event Detected</h1>
            <span style="background: rgba(255,255,255,0.25); color: white;
                         padding: 4px 12px; border-radius: 20px;
                         font-size: 14px; font-weight: bold;">
              {severity.upper()}
            </span>
          </div>
          <div style="background: #f7fafc; padding: 24px;">
            <div style="background: white; border-radius: 8px; padding: 20px;
                        margin-bottom: 16px; border-left: 4px solid {color};">
              <h2 style="margin: 0 0 12px 0; font-size: 16px;">What Happened</h2>
              <p style="margin: 0; line-height: 1.6;">{what_happened}</p>
            </div>
            <div style="background: white; border-radius: 8px; padding: 20px; margin-bottom: 16px;">
              <table style="width: 100%; border-collapse: collapse;">
                <tr>
                  <td style="padding: 8px 0; border-bottom: 1px solid #e2e8f0;
                             font-weight: bold; width: 140px; color: #718096; font-size: 13px;">
                    PERFORMED BY
                  </td>
                  <td style="padding: 8px 0; border-bottom: 1px solid #e2e8f0;">{who_did_it}</td>
                </tr>
                <tr>
                  <td style="padding: 8px 0; font-weight: bold; color: #718096; font-size: 13px;">
                    TIME
                  </td>
                  <td style="padding: 8px 0;">{when_it_happened}</td>
                </tr>
              </table>
            </div>
            <div style="background: white; border-radius: 8px; padding: 20px; margin-bottom: 16px;">
              <h2 style="margin: 0 0 12px 0; font-size: 16px;">Risk Assessment</h2>
              <p style="margin: 0; line-height: 1.6;">{risk_assessment}</p>
            </div>
            <div style="background: #f0fff4; border-radius: 8px; padding: 20px;">
              <h2 style="margin: 0 0 12px 0; font-size: 16px;">Recommended Action</h2>
              <p style="margin: 0; line-height: 1.6;">{recommended_action}</p>
            </div>
          </div>
          <div style="padding: 16px 24px; background: #edf2f7; font-size: 12px; color: #718096;">
            This alert was generated automatically by the CloudTrail Security Event Investigator Agent.
          </div>
        </body>
        </html>
        """

        response = ses.send_email(
            Source=config["sender_email"],
            Destination={"ToAddresses": [config["recipient_email"]]},
            Message={
                "Subject": {"Data": f"[{severity.upper()}] {subject}"},
                "Body": {
                    "Html": {"Data": html},
                    "Text": {"Data": (
                        f"Security Event: {subject}\n\n"
                        f"What Happened: {what_happened}\n"
                        f"Performed By: {who_did_it}\n"
                        f"Time: {when_it_happened}\n"
                        f"Risk: {risk_assessment}\n"
                        f"Action: {recommended_action}"
                    )}
                }
            }
        )

        return {"success": True, "message_id": response["MessageId"]}
    except Exception as e:
        return {"success": False, "error": str(e)}
```

### 7.7 Create tools/security\_hub\_tools.py

Save this as `security_hub_tools.py` inside your `cloudtrail-lab/agent/tools` folder:

```python
"""
Security Hub Tools - Create official security findings
"""
import boto3
from strands import tool
from datetime import datetime, timezone


@tool
def create_security_finding(
    title: str,
    description: str,
    severity: str,
    event_name: str,
    username: str,
    resource_id: str = "aws:cloudtrail:event"
) -> dict:
    """
    Create an official security finding in AWS Security Hub.

    Args:
        title: Short descriptive title for the finding
        description: Full description of what happened and the risk
        severity: CRITICAL, HIGH, MEDIUM, LOW, or INFORMATIONAL
        event_name: The CloudTrail event name
        username: The user who performed the action
        resource_id: The affected AWS resource identifier

    Returns:
        Dictionary with success status and finding ID
    """
    try:
        sts        = boto3.client("sts")
        account_id = sts.get_caller_identity()["Account"]
        region     = "us-east-1"
        now        = datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")
        finding_id = f"cloudtrail-investigator-{event_name}-{datetime.now(timezone.utc).strftime('%Y%m%d%H%M%S')}"

        severity_map = {
            "CRITICAL":      {"Label": "CRITICAL",      "Normalized": 90},
            "HIGH":          {"Label": "HIGH",          "Normalized": 70},
            "MEDIUM":        {"Label": "MEDIUM",        "Normalized": 40},
            "LOW":           {"Label": "LOW",           "Normalized": 20},
            "INFORMATIONAL": {"Label": "INFORMATIONAL", "Normalized": 0}
        }

        finding = {
            "SchemaVersion": "2018-10-08",
            "Id":            finding_id,
            "ProductArn":    f"arn:aws:securityhub:{region}:{account_id}:product/{account_id}/default",
            "GeneratorId":   "cloudtrail-investigator-agent",
            "AwsAccountId":  account_id,
            "Types":         ["TTPs/Initial Access/Unusual Activity Detected"],
            "CreatedAt":     now,
            "UpdatedAt":     now,
            "Severity":      severity_map.get(severity.upper(), severity_map["MEDIUM"]),
            "Title":         title,
            "Description":   description,
            "Remediation": {
                "Recommendation": {
                    "Text": "Review the CloudTrail event and take corrective action if needed.",
                    "Url":  f"https://console.aws.amazon.com/cloudtrail/home?region={region}#/events"
                }
            },
            "Resources": [{
                "Type":   "Other",
                "Id":     resource_id,
                "Region": region
            }],
            "Compliance":    {"Status": "FAILED"},
            "WorkflowState": "NEW",
            "RecordState":   "ACTIVE",
            "Note": {
                "Text":      f"Investigated by CloudTrail Investigator Agent | User: {username} | Event: {event_name} | NIST 800-53: SI-4 | SOC 2: CC7.2",
                "UpdatedBy": "cloudtrail-investigator-agent",
                "UpdatedAt": now
            }
        }

        securityhub = boto3.client("securityhub", region_name=region)
        response    = securityhub.batch_import_findings(Findings=[finding])

        return {
            "success":      True,
            "finding_id":   finding_id,
            "failed_count": response.get("FailedCount", 0)
        }
    except Exception as e:
        return {"success": False, "error": str(e)}
```

---

## 8. Deploy the Agent

### 8.1 Set Up a Python Virtual Environment

> 💻 **TERMINAL ACTION:** Navigate to your agent folder and create the virtual environment using Python 3.10:

```bash
cd ~/Desktop/cloudtrail-lab/agent
uv venv --python 3.10
source .venv/bin/activate
```

After running the last command, you should see `(.venv)` at the beginning of your terminal prompt.

> 🎓 **WHY ARE WE DOING THIS?** Forcing Python 3.10 here ensures compatibility with the AgentCore starter toolkit. Using a newer Python version will cause the installation in the next step to fail.

### 8.2 Install the AgentCore CLI and Dependencies

> 💻 **TERMINAL ACTION:** With your virtual environment active (you should see `(.venv)` in your prompt), run:

```bash
uv pip install bedrock-agentcore-starter-toolkit
uv pip install -r requirements.txt
```

> 🔴 **IMPORTANT:** Use `uv pip install` rather than regular `pip install` here. The AgentCore starter toolkit requires Python 3.10 and uv ensures the packages are installed into your Python 3.10 virtual environment correctly.

### 8.3 Get Your Agent Role ARN

> 💻 **TERMINAL ACTION:** Run:

```bash
aws iam get-role \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorAgentRole \
    --query Role.Arn \
    --output text
```

Copy the output — it looks like: `arn:aws:iam::123456789012:role/CloudTrailInvestigatorAgentRole`

### 8.4 Configure the Agent

> 💻 **TERMINAL ACTION:** With your virtual environment still active and inside the agent folder, run:

```bash
AWS_PROFILE=cloudtrail-lab agentcore configure -e agent.py
```

When prompted, answer as follows:

| Question | Your Answer |
|---|---|
| Agent name | `CloudTrailInvestigator` (no spaces or special characters) |
| Execution Role ARN | Paste the ARN you copied in Section 8.3 |
| ECR Repository | Press Enter to let AWS create one automatically |
| Memory - Short-term | Type `yes` and press Enter |
| Memory - Long-term | Type `no` and press Enter |

### 8.5 Test the Agent Locally

**In your current terminal window, start the local agent server:**

> 💻 **TERMINAL ACTION:** Run:

```bash
AWS_PROFILE=cloudtrail-lab agentcore deploy --local
```

Leave this terminal window open.

**Open a second terminal window and send a test event:**

On Mac, press **Command + T** to open a new terminal tab.

> 💻 **TERMINAL ACTION:** In the new terminal window, run:

```bash
curl -X POST http://localhost:8080/invocations \
    -H 'Content-Type: application/json' \
    -d '{
        "event_name": "AuthorizeSecurityGroupIngress",
        "event_time": "2026-03-01T14:32:00Z",
        "user_identity": {"userName": "test-user"},
        "source_ip": "203.0.113.42",
        "resources": ["sg-0abc12345"],
        "raw_event": {"groupId": "sg-0abc12345", "ipPermissions": [{"fromPort": 22, "toPort": 22, "cidrIp": "0.0.0.0/0"}]}
    }'
```

Watch your first terminal window — you should see the agent start reasoning through the event. Check your email for an investigation summary.

**Stop the local server:**

Go back to your first terminal window and press **Ctrl + C**.

### 8.6 Deploy to AWS

> 💻 **TERMINAL ACTION:** Run:

```bash
AWS_PROFILE=cloudtrail-lab agentcore deploy
```

This will take 3-5 minutes.

> 🔴 **IMPORTANT:** When the deployment finishes, it will print your **Agent ARN**. It looks like: `arn:aws:bedrock-agentcore:us-east-1:123456789012:runtime-endpoint/XXXXXXXXXX`. Copy this value and save it — you will need it in the next section.

---

## 9. Create the Lambda Function

> 🎓 **WHY ARE WE DOING THIS?** EventBridge is designed to trigger functions that respond quickly. Your agent needs time to think, query logs, and send emails, which can take 30-60 seconds. The Lambda function receives the event instantly, acknowledges it to EventBridge, and then invokes the agent asynchronously in the background.

### 9.1 Create the Lambda Code

> 💻 **TERMINAL ACTION:** Create a new folder for your Lambda function:

```bash
mkdir ~/Desktop/cloudtrail-lab/lambda
cd ~/Desktop/cloudtrail-lab/lambda
```

In VS Code, create a new file with the following content and save it as `lambda_function.py` inside your `cloudtrail-lab/lambda` folder:

```python
"""
CloudTrail EventBridge Lambda
Receives security events from EventBridge and invokes the AgentCore agent.
"""
import json
import boto3
import os
from datetime import datetime


AGENT_ARN = os.environ["AGENT_ARN"]


def lambda_handler(event, context):
    """
    Receives CloudTrail events from EventBridge and passes the
    relevant details to the AgentCore investigation agent.
    """
    print(f"Received event: {json.dumps(event, default=str)}")

    try:
        detail     = event.get("detail", {})
        event_name = detail.get("eventName", "UnknownEvent")
        event_time = detail.get("eventTime", datetime.utcnow().isoformat())
        user_ident = detail.get("userIdentity", {})
        source_ip  = detail.get("sourceIPAddress", "unknown")
        resources  = detail.get("resources", [])
        req_params = detail.get("requestParameters", {})

        resource_ids = []
        if req_params.get("groupId"):
            resource_ids.append(req_params["groupId"])
        for r in resources:
            if r.get("ARN"):
                resource_ids.append(r["ARN"])

        agent_payload = {
            "event_name":    event_name,
            "event_time":    event_time,
            "user_identity": user_ident,
            "source_ip":     source_ip,
            "resources":     resource_ids,
            "raw_event": {
                "eventName":         event_name,
                "requestParameters": req_params,
                "userIdentity":      user_ident,
                "sourceIPAddress":   source_ip
            }
        }

        session_id = f"event-{event_name}-{datetime.utcnow().strftime('%Y%m%d%H%M%S')}"

        agent_client = boto3.client(
            "bedrock-agent-runtime",
            region_name="us-east-1"
        )

        agent_client.invoke_agent_runtime_endpoint(
            agentRuntimeEndpointArn=AGENT_ARN,
            sessionId=session_id,
            request=json.dumps(agent_payload)
        )

        print(f"Successfully invoked agent for event: {event_name}")
        return {
            "statusCode": 200,
            "body": json.dumps({
                "status":     "Agent invoked",
                "event_name": event_name,
                "session_id": session_id
            })
        }

    except Exception as e:
        print(f"Error invoking agent: {str(e)}")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }
```

### 9.2 Package and Deploy the Lambda Function

> 💻 **TERMINAL ACTION:** Navigate to your lambda folder and create a zip file:

```bash
cd ~/Desktop/cloudtrail-lab/lambda
zip lambda_function.zip lambda_function.py
```

Deploy the Lambda function (replace `YOUR_ACCOUNT_ID` and `YOUR_AGENT_ARN`):

```bash
aws lambda create-function \
    --profile cloudtrail-lab \
    --function-name cloudtrail-event-investigator \
    --runtime python3.12 \
    --handler lambda_function.lambda_handler \
    --role arn:aws:iam::YOUR_ACCOUNT_ID:role/CloudTrailInvestigatorLambdaRole \
    --zip-file fileb://lambda_function.zip \
    --timeout 60 \
    --memory-size 256 \
    --environment Variables={AGENT_ARN=YOUR_AGENT_ARN} \
    --region us-east-1
```

Verify the Lambda was created:

```bash
aws lambda get-function \
    --profile cloudtrail-lab \
    --function-name cloudtrail-event-investigator \
    --region us-east-1 \
    --query Configuration.FunctionArn \
    --output text
```

---

## 10. Set Up the EventBridge Rule

> 🎓 **WHY ARE WE DOING THIS?** CloudTrail logs every single action in your AWS account — potentially thousands of events per day. EventBridge lets you write a precise filter that selects only the specific events you care about.

### 10.1 Create the Event Pattern File

In VS Code, create a new file with the following content and save it as `event-pattern.json` inside your `cloudtrail-lab` folder:

```json
{
    "source": ["aws.ec2"],
    "detail-type": ["AWS API Call via CloudTrail"],
    "detail": {
        "eventSource": ["ec2.amazonaws.com"],
        "eventName": [
            "AuthorizeSecurityGroupIngress",
            "AuthorizeSecurityGroupEgress",
            "RevokeSecurityGroupIngress",
            "RevokeSecurityGroupEgress",
            "CreateSecurityGroup",
            "DeleteSecurityGroup"
        ]
    }
}
```

### 10.2 Create the EventBridge Rule

> 💻 **TERMINAL ACTION:** Navigate back to your cloudtrail-lab folder and run:

```bash
cd ~/Desktop/cloudtrail-lab

aws events put-rule \
    --profile cloudtrail-lab \
    --name cloudtrail-security-group-changes \
    --event-pattern file://event-pattern.json \
    --state ENABLED \
    --description "Watches for security group changes via CloudTrail" \
    --region us-east-1
```

### 10.3 Grant EventBridge Permission to Invoke Lambda

Replace `YOUR_ACCOUNT_ID`:

```bash
aws lambda add-permission \
    --profile cloudtrail-lab \
    --function-name cloudtrail-event-investigator \
    --statement-id allow-eventbridge \
    --action lambda:InvokeFunction \
    --principal events.amazonaws.com \
    --source-arn arn:aws:events:us-east-1:YOUR_ACCOUNT_ID:rule/cloudtrail-security-group-changes
```

### 10.4 Connect the Rule to Your Lambda

Replace `YOUR_ACCOUNT_ID`:

```bash
aws events put-targets \
    --profile cloudtrail-lab \
    --rule cloudtrail-security-group-changes \
    --targets Id=lambda-target,Arn=arn:aws:lambda:us-east-1:YOUR_ACCOUNT_ID:function:cloudtrail-event-investigator
```

### 10.5 Verify the Rule Is Active

```bash
aws events describe-rule \
    --profile cloudtrail-lab \
    --name cloudtrail-security-group-changes \
    --region us-east-1 \
    --query '{Name:Name, State:State}'
```

You should see `State: ENABLED`. Your system is now live.

---

## 11. Testing Your System

### 11.1 Create a Test Security Group

> 💻 **TERMINAL ACTION:** Run:

```bash
aws ec2 create-security-group \
    --profile cloudtrail-lab \
    --group-name lab-test-firewall \
    --description "Test security group for CloudTrail lab" \
    --region us-east-1
```

Note the `GroupId` from the output (looks like `sg-0abc12345`).

### 11.2 Trigger the Agent

> 💻 **TERMINAL ACTION:** Replace `sg-YOUR-GROUP-ID` with your GroupId and run:

```bash
aws ec2 authorize-security-group-ingress \
    --profile cloudtrail-lab \
    --group-id sg-YOUR-GROUP-ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0 \
    --region us-east-1
```

### 11.3 Immediately Revoke the Test Rule

As soon as you have confirmed the command ran successfully, remove the rule you just created.

> 💻 **TERMINAL ACTION:** Run right away (replace `sg-YOUR-GROUP-ID`):

```bash
aws ec2 revoke-security-group-ingress \
    --profile cloudtrail-lab \
    --group-id sg-YOUR-GROUP-ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0 \
    --region us-east-1
```

> 💡 **TIP:** Your test security group has no EC2 instances attached to it, so nothing was actually exposed to the internet. However, removing the rule immediately builds good security habits. The revocation will also trigger your agent a second time — this time classifying the event as INFORMATIONAL since removing a permissive rule is a security improvement.

### 11.4 Wait for the Agent to Respond

CloudTrail events can take up to 5-15 minutes to arrive via EventBridge. While you wait, watch your Lambda logs in real time:

> 💻 **TERMINAL ACTION:** Run (press Ctrl+C to stop):

```bash
aws logs tail \
    --profile cloudtrail-lab \
    /aws/lambda/cloudtrail-event-investigator \
    --follow
```

> ⚠️ **NOTE:** If you do not see an email within 15 minutes, use the troubleshooting steps in Section 12.

### 11.5 Verify the Security Hub Finding

> 💻 **TERMINAL ACTION:** Run:

```bash
aws securityhub get-findings \
    --profile cloudtrail-lab \
    --filters '{"GeneratorId": [{"Value": "cloudtrail-investigator-agent", "Comparison": "EQUALS"}]}' \
    --region us-east-1 \
    --query 'Findings[0].{Title:Title,Severity:Severity.Label,Time:CreatedAt}'
```

### 11.6 View the Finding in the Console

> 🖥️ **AWS CONSOLE ACTION:**
> 1. Go to the AWS Console and search for **Security Hub**.
> 2. Click **Findings** in the left navigation.
> 3. Look for findings with Generator ID containing `cloudtrail-investigator-agent`.

### 11.7 Test 2: Confirm the Revocation Was Also Investigated

Check your inbox for a second email — this one should have a much lower severity (INFORMATIONAL) since removing an overly permissive rule is a positive security action. This demonstrates that the agent is not just pattern matching on event names — it is evaluating whether the change made your environment more or less secure.

---

## 12. Troubleshooting

| Problem | How to Fix It |
|---|---|
| No email received after 15 minutes | 1. Check Lambda logs (Section 11.4 command). 2. Verify your email is confirmed in SES. 3. Confirm the EventBridge rule is ENABLED. |
| Lambda logs show "Agent ARN not found" | Re-check the `AGENT_ARN` environment variable on your Lambda. Run `agentcore deploy` again to get the correct ARN. |
| `AccessDenied` error when deploying agent | Verify your IAM role has both service principals in the trust policy (`bedrock.amazonaws.com` and `bedrock-agentcore.amazonaws.com`). |
| `agentcore` command not found | Your virtual environment is not active. Run: `source ~/Desktop/cloudtrail-lab/agent/.venv/bin/activate` |
| Bedrock model access error | Go to the Bedrock console and verify Claude Sonnet 4.6 shows "Access granted" in Model access. |
| SES "Email address not verified" error | Open your inbox, find the AWS verification email, and click the link. |
| Large zip file error (135MB+) | Your virtual environment is being included. Make sure you only zip `lambda_function.py`. |
| EventBridge rule ENABLED but nothing fires | CloudTrail events can take up to 15 min. Also confirm `put-targets` ran successfully. |
| `No matching distribution found for bedrock-agentcore` | Your virtual environment is not using Python 3.10. Re-create it with: `uv venv --python 3.10` |
| Role validation failed on `agentcore deploy` | Your trust policy is missing `bedrock-agentcore.amazonaws.com`. Re-run the `update-assume-role-policy` command from Section 5.1. |

**How to check CloudWatch logs:**

```bash
aws logs tail \
    --profile cloudtrail-lab \
    /aws/lambda/cloudtrail-event-investigator \
    --since 1h
```

---

## 13. Clean Up (Important!)

> 🔴 **IMPORTANT:** Run all of the following commands to avoid unexpected AWS charges. Replace `YOUR_ACCOUNT_ID` throughout.

### 13.1 Delete the EventBridge Rule

```bash
aws events remove-targets \
    --profile cloudtrail-lab \
    --rule cloudtrail-security-group-changes \
    --ids lambda-target
```

```bash
aws events delete-rule \
    --profile cloudtrail-lab \
    --name cloudtrail-security-group-changes
```

### 13.2 Delete the Lambda Function

```bash
aws lambda delete-function \
    --profile cloudtrail-lab \
    --function-name cloudtrail-event-investigator
```

### 13.3 Delete the Test Security Group

```bash
aws ec2 delete-security-group \
    --profile cloudtrail-lab \
    --group-id sg-YOUR-GROUP-ID
```

### 13.4 Delete the AgentCore Agent

```bash
AWS_PROFILE=cloudtrail-lab agentcore delete
```

### 13.5 Delete Secrets Manager Secret

```bash
aws secretsmanager delete-secret \
    --profile cloudtrail-lab \
    --secret-id cloudtrail-investigator/notifications \
    --force-delete-without-recovery
```

### 13.6 Delete IAM Roles and Policies

```bash
aws iam detach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorAgentRole \
    --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorAgentPolicy
```

```bash
aws iam detach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorLambdaRole \
    --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorLambdaPolicy
```

```bash
aws iam detach-role-policy \
    --profile cloudtrail-lab \
    --role-name CloudTrailInvestigatorLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

```bash
aws iam delete-role --profile cloudtrail-lab --role-name CloudTrailInvestigatorAgentRole
aws iam delete-role --profile cloudtrail-lab --role-name CloudTrailInvestigatorLambdaRole
aws iam delete-policy --profile cloudtrail-lab --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorAgentPolicy
aws iam delete-policy --profile cloudtrail-lab --policy-arn arn:aws:iam::YOUR_ACCOUNT_ID:policy/CloudTrailInvestigatorLambdaPolicy
```

---

## 14. Deployment Checklist

- [ ] Homebrew installed (Mac) or WSL configured (Windows)
- [ ] Python 3.10 installed — verified with: `python3.10 --version`
- [ ] AWS CLI installed — verified with: `aws --version`
- [ ] uv package manager installed — verified with: `uv --version`
- [ ] VS Code installed
- [ ] AWS account created or confirmed accessible
- [ ] IAM user `cloudtrail-lab` created with AdministratorAccess
- [ ] Access key created and saved securely
- [ ] AWS CLI profile `cloudtrail-lab` configured — verified with: `aws sts get-caller-identity --profile cloudtrail-lab`
- [ ] Amazon Bedrock enabled with Claude Sonnet 4.6 model access granted
- [ ] AWS Security Hub enabled in us-east-1
- [ ] Email address verified in Amazon SES (verification link clicked)
- [ ] Folder `~/Desktop/cloudtrail-lab` created
- [ ] IAM role `CloudTrailInvestigatorAgentRole` created with both Bedrock service principals in trust policy
- [ ] IAM role `CloudTrailInvestigatorLambdaRole` created with policies attached
- [ ] Secrets Manager secret `cloudtrail-investigator/notifications` created
- [ ] Agent code files created: `agent.py`, `requirements.txt`, `tools/__init__.py`, `tools/cloudtrail_tools.py`, `tools/ec2_tools.py`, `tools/notification_tools.py`, `tools/security_hub_tools.py`
- [ ] Python virtual environment created using `uv venv --python 3.10` and activated
- [ ] `uv pip install -r requirements.txt` completed successfully
- [ ] `agentcore configure` completed — agent named `CloudTrailInvestigator`
- [ ] Local test passed — email received from `agentcore deploy --local`
- [ ] `agentcore deploy` completed — Agent ARN saved
- [ ] Lambda code file `lambda_function.py` created
- [ ] Lambda function `cloudtrail-event-investigator` deployed with `AGENT_ARN` environment variable
- [ ] Lambda function ARN saved
- [ ] EventBridge pattern file `event-pattern.json` created
- [ ] EventBridge rule `cloudtrail-security-group-changes` created and ENABLED
- [ ] Lambda permission granted to EventBridge
- [ ] EventBridge target connected to Lambda function
- [ ] Test security group created
- [ ] Test 1 passed: Added `0.0.0.0/0` port 22 rule → received HIGH/CRITICAL email alert
- [ ] Test rule revoked immediately after Test 1
- [ ] Test 2 passed: Revocation → received INFORMATIONAL email
- [ ] Security Hub finding visible in the AWS console
- [ ] All resources cleaned up per Section 13

---

*Lab Complete — You have built an AI-powered security event investigator using AWS AgentCore and Amazon Bedrock.*

