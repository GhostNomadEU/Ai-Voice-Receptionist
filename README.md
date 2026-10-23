![GhostReception - AI Voice Receptionist](https://raw.githubusercontent.com/GhostNomadEU/GhostReception/main/Header.png)

# 👻 GhostReception

> An exportable Make.com scenario scaffold for a Vapi-powered AI phone receptionist.

Build notes, architecture, and setup guidance for an AI receptionist workflow that can check Google Calendar availability, book a confirmed appointment, and alert a team through Slack.

**Current status:** 🧱 Blueprint scaffold only — not a live or tested receptionist. Connections, hook setup, mappings, agent instructions, and runtime testing are still required.

---

## ✨ What it is

GhostNomad Voice Receptionist connects a Vapi phone assistant to a Make AI Agent. Vapi handles the live voice conversation; Make receives structured tool-call requests, selects an appropriate connected tool, and returns a result for Vapi to communicate to the caller.

When fully configured, the workflow is designed to:

- 📞 Receive a function/tool request from a Vapi assistant during a call.
- 🗓️ Check Google Calendar free/busy availability for a requested time window.
- ✅ Create a calendar event only after the caller explicitly confirms an exact available date and time.
- 🚨 Send an urgent-call alert to a configured Slack destination for human follow-up.
- 🔁 Return the agent result to Vapi.

**Important:** Urgent calls are alerted to a human through Slack; this workflow does **not** transfer a live call. Call records and transcripts remain in Vapi. This scenario does not create a separate transcript store.

## ⚙️ GhostReception Workflow

![GhostReception Workflow](https://raw.githubusercontent.com/GhostNomadEU/GhostReception/main/Workflow.png)

## 🧭 Architecture

```text
Caller
  │
  ▼
Vapi phone assistant
  │  Structured tool/function request
  ▼
Vapi — Watch Tool Call Request
  │
  ▼
Make AI Agent — Receptionist Agent
  ├── Google Calendar — Check Availability
  ├── Google Calendar — Book Confirmed Appointment
  └── Slack — Alert Team About Urgent Call
  │
  ▼
Vapi — Respond to a Tool Call
  │
  ▼
Vapi communicates the result to the caller
```

Vapi handles phone audio and the live conversation. Make's AI Agent does **not** process raw phone audio; it receives structured requests from Vapi, decides which connected tool to use, and returns a result.

The Calendar and Slack modules are attached as agent tools. They are **not** a fixed sequence that runs on every request. The agent should choose a tool based on the incoming request and its instructions.

## 🧩 Modules in the blueprint

| Module | Purpose | Scaffold status |
| --- | --- | --- |
| Vapi — Watch Tool Call Request | Instant entry point for Vapi tool requests | Present; connection and hook need configuration |
| Make AI Agent — Receptionist Agent | Interprets structured requests and selects tools | Present; provider connection, instructions, and input mapping need configuration |
| Google Calendar — Check Availability | Checks calendar free/busy information | Present as an agent tool; connection and fields need configuration |
| Google Calendar — Book Appointment | Creates a calendar event after explicit confirmation | Present as an agent tool; connection and fields need configuration |
| Slack — Urgent Alert | Sends an urgent-call alert for human follow-up | Present as an agent tool; connection, destination, and message need configuration |
| Vapi — Respond to a Tool Call | Returns the agent response to Vapi | Present; verify the tool-call ID and response mappings against the actual payload |

## 🛠️ Prerequisites

Before deployment, prepare:

- A Vapi account with an inbound-call assistant and phone number configured.
- A Vapi connection in Make.
- A supported model-provider connection for the Make AI Agent.
- A Google Calendar connection with access to the appointment calendar.
- A Slack connection with permission to post to the selected destination.

Decide these business settings before going live:

- Business name and timezone.
- Target calendar and standard appointment duration.
- Booking title and event details.
- Required caller details and appointment policies.
- What qualifies as urgent and who receives Slack alerts.
- Caller-data handling, retention, and disclosure requirements.

These values are not hard-coded in this scaffold.

## 🚀 Setup after importing

1. **Import the blueprint.** Import the JSON blueprint into Make and review the scenario and module chain.
2. **Configure Vapi.** Connect Vapi and set up **Watch Tool Call Request** and its hook. Complete the tool setup created by the Vapi module, then configure the Vapi assistant to invoke that tool during calls.
3. **Connect the agent model provider.** Select a supported provider connection for the Make AI Agent.
4. **Map agent input.** Map the Vapi function-call arguments and relevant call context into the agent's Input field. Confirm the real payload shape during setup; this scaffold has no captured test payload.
5. **Configure availability checks.** Connect Google Calendar, choose the target calendar, and map the requested availability window and business timezone.
6. **Configure event creation.** Use the same calendar for availability and booking. Map explicit start and end times, title, details, and timezone. Set the standard appointment duration.
7. **Configure Slack.** Connect Slack, select the destination, and compose a concise urgent-call alert. Include caller details only when provided and needed for follow-up.
8. **Set agent instructions.** Start with [`agent-prompt.md`](./agent-prompt.md), replace placeholders, and adapt its output contract to the actual Vapi tool schema.
9. **Verify Vapi response mapping.** The blueprint currently maps the first item in `toolCallList` to the tool-call ID and the agent response to Vapi's result. Confirm those paths match the actual request/response schema created during Vapi setup.
10. **Test before activation.** Test availability lookup, explicit-confirmation booking, urgent Slack alerts, tool failures, and the result returned to Vapi. Activate only after the end-to-end behavior is verified.

The Vapi trigger is instant, not schedule-based. Continuous availability depends on the Vapi tool being configured correctly and the Make scenario being active.

## 🧪 Suggested test checklist

- [ ] A Vapi tool request reaches the Make scenario.
- [ ] Missing date/time details are clarified instead of guessed.
- [ ] Availability is checked before a time is offered.
- [ ] Only calendar-confirmed available times are offered.
- [ ] No event is created before the caller confirms the exact slot.
- [ ] A booking result is reported only after the calendar tool confirms success.
- [ ] Urgent requests create a Slack alert with only necessary details.
- [ ] Routine inquiries do not trigger urgent alerts.
- [ ] Tool failures return an honest follow-up message.
- [ ] The result is valid for the configured Vapi tool response schema.
- [ ] Timezone, duration, and calendar mappings are correct.

## 🔐 Privacy and security

- Never commit API keys, OAuth credentials, live hook URLs, caller data, or private business information.
- Remove private values from exported blueprints before publishing.
- Grant connected accounts only the permissions required for their roles.
- Send the minimum caller information needed in Slack alerts.
- Avoid sending full transcripts to Slack unless there is a specific, approved need.
- Review Vapi call-recording, transcript-retention, and disclosure settings for your use case.
- Test with synthetic data before using real callers.

## 🧰 Troubleshooting

| Symptom | What to check |
| --- | --- |
| Make receives no Vapi tool requests | Confirm the Vapi hook is configured and the assistant invokes the generated tool. |
| Vapi rejects the tool response | Verify the tool-call ID mapping and ensure the agent output matches the configured Vapi schema. |
| Availability results look wrong | Check the selected calendar, requested time window, and business timezone. |
| An event is not created | Confirm explicit caller approval and verify the Calendar connection and event fields. |
| Slack alert is not delivered | Check the Slack connection, destination, and permission to post. |
| The Make AI Agent cannot run | Confirm a supported model-provider connection and all required agent fields are configured. |

## 📁 Repository layout

```text
.
├── README.md
├── agent-prompt.md
└── GhostReception.json
```

Keep the blueprint filename consistent with the actual file you commit. The uploaded scaffold may have a different filename; rename it to the example above only if that matches your repository convention.

## 📌 Project status

- [x] Scenario scaffold exported
- [x] Module chain and agent tools represented in blueprint
- [ ] Vapi connection and hook configured
- [ ] Model-provider connection and agent input configured
- [ ] Google Calendar and Slack fields configured
- [ ] End-to-end tests completed
- [ ] Scenario activated for live calls


## 👻 About GhostNomad

I'm GhostNomad — building AI automations and agents with Make.com and sharing the journey, experiments, and reusable blueprints.

Follow the build, adapt the blueprint to your own use case, and test carefully before connecting it to real callers.

---

**Disclaimer:** This project is a technical scaffold. You are responsible for configuring, testing, securing, and validating all integrations and business rules before use.
