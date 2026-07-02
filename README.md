# Threat Hunting Engine

## What this is

This is a tool that lets a security analyst investigate potential threats by typing in plain English instead of writing a technical query by hand. Something like "check for suspicious logons in the last 24 hours" is enough. The tool figures out where to look, pulls the real security logs, and uses AI to review them and explain what it found.

<img width="1372" height="484" alt="Screenshot 2026-06-29 202209" src="https://github.com/user-attachments/assets/fed71c88-f309-429f-9ba1-0cae971b45e6" />


It was built for the kind of work a security operations team does. An analyst has suspicion, needs to check real data to confirm or rule it out, and then has to decide what to do next. This tool handles the repetitive middle part of that process so the analyst can spend more time on judgment and less time on typing out queries by hand.

## Tools and Services Used

- Python
- OpenAI API
- Azure Log Analytics
- Microsoft Defender for Endpoint

## Setup and what each piece does

### Python

This tool is built primarily in Python, making it ideal for this type of work because of how it can handle data and it makes API integrations seamless.

### The OpenAI API

This is what gives the tool its reasoning ability. It's used in two separate steps. First, to decide which logs are relevant to a request. Second, to review the logs once they come back and explain what looks suspicious.

### The Azure CLI

This is a separate command line tool from Microsoft, and it has to be installed and logged into once before the tool can pull real data. Without it, the tool has no way to prove to Azure that it's allowed to read the logs.

### Tiktoken

This is a small library that counts how many tokens a piece of text will use before it's sent to the AI. It matters here because logs can be large, and a request that's too big will either fail outright or cost more than expected. Counting ahead of time lets the tool warn about this before spending anything.

---

## How the tool actually works

When an analyst types a request, the AI doesn't just go run whatever it wants. It first has to propose a plan: which table to search, which fields to pull back, and how far back in time to look. It also has to explain its reasoning in plain language, which gets printed out so the choice can be checked before anything actually runs.

<img width="1725" height="246" alt="image" src="https://github.com/user-attachments/assets/283e565a-2615-49a2-8bc5-799ee349a514" />


That plan then gets checked against a fixed allow list before it's used. If the AI ever picks a table or a field that isn't permitted, the tool stops right there. It's allowed to suggest, but the code decides what's actually allowed to happen.

Once a plan passes that check, a real query runs against Azure and pulls back the matching log records. If nothing comes back, the tool stops there too, instead of spending money running an AI review on an empty result.

The logs that do come back get handed to the AI a second time, along with instructions specific to whatever table was searched. It reviews the actual data and returns something structured: a title, a description, a confidence level, and a mapping to a known attack technique.

<img width="2269" height="604" alt="Screenshot 2026-06-29 203030" src="https://github.com/user-attachments/assets/bdcd4369-f95c-4d6d-907d-dca7cecda4e9" />


---

## Cost and model control

Every hunt has a real cost, since it involves a paid API. Before running the analysis step, the tool estimates how many tokens the request will use and checks that against the limits of the model being used. If the logs are too large for the smaller, cheaper model, the tool flags this and lets the analyst choose whether to switch to a larger one. This is a deliberate checkpoint. Nothing runs automatically without the analyst knowing roughly what it will cost first.

---
## Flow Chart
<img width="823" height="999" alt="Screenshot 2026-06-30 211728" src="https://github.com/user-attachments/assets/3a4116bd-7103-4eb1-989f-15bfc6a0dcf6" />


---

## Also available as a dashboard

Everything described above was originally built to run in a terminal. It also connects to a web dashboard, which sits on top of this same engine and gives it a full interface: a page for running hunts, a running case history, analyst notes, and a summary of AI cost over time. The dashboard doesn't change how any of the above works. It calls into this same engine and displays what comes back.
 
**Dashboard repo:** https://github.com/Colby-hin/agentic-threat-hunter-dashboard
