---
name: building-askpilot-workflows
description: Builds business workflows in Askpilot that run on their own, from the job the user describes to a tested PROCESS.md with its sources, triggers and files. Use when the user wants to automate a business process with Askpilot, for example chase unpaid invoices, follow up leads, collect documents, answer customers on WhatsApp or turn calls into admin, or asks to create, change, fix or copy an Askpilot workflow or a PROCESS.md.
---

# Building Askpilot workflows

This skill is the order of work for building an Askpilot workflow. The facts live in the Askpilot knowledge base and the templates, so read them through the Askpilot tools when a step says so, and do not rely on memory for them.

The signed-in user is the Askpilot user you are working for, the one who signed in to Askpilot MCP; before they sign in, it is the person you are talking to.

The tools named below, such as `workflow_create`, are the tools of the Askpilot connector. `article_get` reads an article by its name.

## Before you start

Read these articles with `article_get`, in full, if you have not read them in this conversation: `get-started`, `workflows`, `process-md`, `workflow-examples`, `sources` and `ask-agent`. Then call `organization_list` to see their organizations. If the signed-in user belongs to only one organization, work in that one without asking; if they belong to more than one, ask which one to work in.

## Build a workflow

Copy this checklist into your answer and tick each step as you finish it:

```
Workflow progress:
- [ ] 1. Learn how Askpilot works
- [ ] 2. Discuss the workflow with the signed-in user
- [ ] 3. Agree the plan
- [ ] 4. Set up the sources
- [ ] 5. Write the process text
- [ ] 6. Add the files
- [ ] 7. Review and create
- [ ] 8. Finish the setup with the signed-in user
- [ ] 9. Test and fix
```

### 1. Learn how Askpilot works

Before you ask the signed-in user anything, learn what Askpilot can do for this job, so your questions and suggestions fit what is possible:

- Read the articles named in Before you start, if you have not read them in this conversation.
- Read every template in full with `process_template_get`, with their assets, after listing them with `process_template_list`. They are short, and each one shows a different way to build a workflow: how it starts, how it follows up, how it hands over and what it sends back. If there are more than 20, read those in the job's category, and any that start, follow up or send their result the way this job needs. Use them for structure and wording later; a template is an example, not a file to copy as it is. If the signed-in user is not sure what to automate, start from the templates that need only email, WhatsApp or a calendar, such as lead follow-up, invoice chasing or document collection: they run the soonest. These are a starting point, not a limit: suggest any workflow that fits their business, including ones no template covers.
- Look at what the organization already has: its sources with `source_list`, the tools and events of the ones the job needs with `source_get` or `source_type_get`, and its workflows with `workflow_list`, so you do not build one that already exists.
- **Learn how each outside tool the job uses works.** Every source works through a real service, such as WhatsApp, Outlook or a CRM, and that service has its own rules, which Askpilot does not change: a step written against them fails. Start from what Askpilot shows: the source's tools and events, with `source_get` or `source_type_get`, tell you what the Ask agent can do in the tool and what can start or resume a session. Then, if you can search the web, read the service's own documentation to learn how it works and which of its rules affect the job. For example, WhatsApp allows a free-form message only within 24 hours of the contact's last message, so a later follow-up needs a message template approved in advance. Bring what you learn into step 2, and tell the signed-in user when a rule changes what the workflow can do.
- **When a tool the job needs has no source, try a Webhook source before you give up on it.** A Webhook source sends data from a session to any system that accepts webhooks, so a workflow can still update a tool Askpilot has no integration for, for example put a qualified lead into the company's own CRM or a finished job into their accounting tool. Put extra effort into this: if you can search the web, check in the tool's own documentation whether it accepts incoming webhooks or has an API that does, and what the request must contain. If the tool does not accept them, check whether the company already uses an automation platform, for example Zapier, Make or n8n, that can receive the webhook and update the tool. Suggest it only as a bridge for this one step; Askpilot still runs the workflow. A Webhook source only sends: it cannot read from the tool, so a step that must read data there still needs an integration. Tell the signed-in user what you found, and if nothing works, suggest they ask Askpilot to build the integration, which it does on request at no extra cost.

### 2. Discuss the workflow with the signed-in user

Talk the workflow through with the signed-in user until it is complete. Treat it as a conversation, not a form:

- Ask 2 to 4 questions at a time, grouped by topic, each with the answer you suggest or a few options, so they can simply agree.
- Do not ask what you can find out yourself, such as which sources exist or which events a tool sends.
- Say the default you will use when they have no preference, for example "I will stop after 3 reminders unless you want more".
- Use what you learned in step 1: name their real sources, and point out the limits of their tools, such as WhatsApp's 24-hour window for free-form messages.

**Suggest a start that runs on its own.** Prefer an event from a source, for example a new enquiry in the CRM or a WhatsApp message, and check it exists with its filter fields by passing `event`. Next best is the Askpilot API, when their own system, or a tool Askpilot has no source for, should start it: any system that can send an HTTPS request to the workflow's `endpoint_url`, with an API key and the context of the run, can start it. Put extra effort into this before you settle for a start by hand: if you can search the web, check in the tool's own documentation whether it can send an outgoing webhook or an HTTPS request when something happens, and whether it can add the API key to the request. If it cannot, check whether the company already uses an automation platform, for example Zapier, Make or n8n, that can watch the tool and call the endpoint. Suggest it only as a bridge for this one step; Askpilot still runs the workflow. To learn how the call works, read https://askpilot.com/docs/developers/workflows.md, the whole API at https://askpilot.com/docs/developers/, and its OpenAPI spec at https://public-api.askpilot.com/openapi.json. If they have their own system, offer to explain the call from those docs, or to write it if you can work in their code. By hand, with `session_start`, when nothing else fits, when they want it, or when starting by hand is a good fit: a person decides which cases to run, for example which overdue invoices to chase this week; the case comes from a conversation or a phone call that no tool records, for example a tenant reporting a repair in person; the job is run now and then on request, for example a briefing before a meeting; or the tools it uses send no events, such as Outlook or iamproperty. Do not tell them a tool cannot start a workflow before you have checked both an event and the API.

**Offer a first version that runs on what they already have.** If the workflow needs tools that are not connected yet, such as a CRM or a phone system, suggest a first version on the sources that already work, usually email, WhatsApp or a calendar, so it runs today, and add the rest once those tools are connected. For example, reply to enquiries from the inbox now, and start them from the CRM later. If the signed-in user wants the full version from the start, build that.

Keep going until every point below has an answer or an agreed default:

| Topic | What you need to know |
| --- | --- |
| Start | What starts it: an event, their own system through the API, or a person by hand, and which occurrences count |
| Run data | What each run works on, and where each value comes from |
| Who | Who it runs for, a person, a team account or the whole company, who it talks to, and who it acts as |
| Steps | The usual path, step by step, and the tool each step uses |
| Branches | What changes the path: a reply, a question, a refusal, a payment |
| Timing | Every step due later, with its gap, its condition and its contact hours |
| Approvals | Which steps the signed-in user approves first |
| Done | When the work is finished, for example the invoice is paid or the viewing is booked, and what happens then |
| Stop | The Stop rules: the conditions that end the work at any step, for example the customer asks not to be contacted or the case is withdrawn |
| Escalation | When a human takes over, who, and how to reach them |
| Edge cases | Missing or wrong data, no reply, an unexpected reply, a duplicate event, an opt-out, a failed step in a tool |
| Result | What the signed-in user gets at the end: a summary, a record updated, a file |

**Walk through the workflow as stories to find the edge cases.** Play it through 3 or 4 concrete cases with the signed-in user, for example "the customer pays after the second reminder", "they reply with a question", "nobody replies", "the email bounces". Ask what should happen in each, and add what you learn to the plan.

### 3. Agree the plan

When every point is covered, write the whole workflow back to the signed-in user in plain language, not as a PROCESS.md: how it starts, what it does step by step, the sources it uses, the timing, the approvals, when the work is done, its Stop rules and what happens in each edge case you discussed. Ask them to confirm or correct it. Move on only once they agree, and do not keep asking questions once every point is covered.

### 4. Set up the sources

A workflow works only if every source it uses is set up, so finish this step before you write or create anything.

**List the sources the plan needs:** for each step, the source that does it and the tools it needs; for the start, the source that sends the event; for any Events line, the source whose events the session waits for; and any Files, Web pages, Google Drive or SharePoint source it answers from. For a shared workflow, the member it runs for must be able to use each one: check with `member_source_list`.

**Check each source,** and show the signed-in user the list with its state:

| Check | How |
| --- | --- |
| It exists | `source_list` |
| It is connected | `connected` is true in `source_get` |
| Its tools for the job are enabled | the `tools` of `source_get`; switch them on with `source_tool_set_enabled` |
| Its events for the start or the Events line exist | the `webhook_events` of `source_get` |
| Its content is added, for a Files, Web pages, Google Drive or SharePoint source | the `content` of `source_get` lists the files or pages and says whether they are ready |
| Its example body is set, for a Webhook source | the `webhook` of `source_get` |
| It is the right account | ask the signed-in user, for example which mailbox or which phone number is connected |
| What the service needs is in place | for example an approved WhatsApp message template, or the webhook urls registered in the tool; ask the signed-in user to confirm what cannot be checked through Askpilot |

**Create what is missing** with `source_create`: the right type, a short name, a description written for the Ask agent, and the access. Then guide the signed-in user to finish it in the Askpilot app: give them the source's url and say exactly what to do there, as the article named `sources` explains under Step 2, for example sign in to Outlook, add the price list PDF, or enter the webhook's endpoint and example body.

**Wait until it is done, and check it.** Ask the signed-in user to tell you when they have finished, then check the source again with `source_get`, and confirm with them that it is the right account and content. Repeat until every source on the list passes.

**Only then continue to step 5.** Do not write a step around a source that is not ready, do not swap in a similar source, and do not create the workflow before every source passes: a workflow with a source that is not set up does not work. If the signed-in user cannot finish a source now, stop here, keep the agreed plan, and tell them you will continue once the source is ready.

### 5. Write the process text

Write the PROCESS.md as the article named `process-md` explains: the name and description go into their own fields, and everything below the header into `process`. Use only the sections the job needs: Role, Instructions, Events, Rules, Todos, Paths.

Check every line against these points before you move on:

- **Sources by their exact name**, as `source_list` returns it, and every source the process names is in the trigger's `sources` or the session's `source_ids`.
- **Run data:** say where each value comes from, the event, the context of the run or a source, and what to do when it is missing.
- **Timing** written plainly, "2 hours after the first message", "3 days after the due date", with the condition and when to stop. Runs start only on 10-minute marks.
- **Events** named exactly, with filter fields that the event really carries. A wrong filter never matches and nothing warns you.
- **Approvals** on every step that sends something outside the company, spends money or makes a commitment, unless the signed-in user says otherwise.
- **Rules** for when to stop and when to escalate, with the contact details of whoever is escalated to.
- **Lean text:** each thing said once, no background that does not change what the Ask agent does.

**Check that the workflow can do everything it says.** The signed-in user judges Askpilot by what the workflow promises, so a promise the setup cannot keep looks like Askpilot not working. Go through the description and every step, and for each thing it says the workflow will do, name what makes it possible:

| It says it will | It needs |
| --- | --- |
| Do something in a tool | a source for that tool, with the tool that does it enabled |
| Use a value, such as a name, an amount or a date | the event, the context of the run or a source where it is found |
| React when something happens, such as a reply | an event from a source, or a check on each scheduled run when the tool sends no events |
| Act at a time | the timing written plainly, on 10-minute marks |
| Contact someone | a channel the service allows at that moment, such as an approved WhatsApp template after 24 hours |

If something has nothing behind it, do not write it as if it will happen: change the step so the setup can do it, add what is missing, or take it out. Then tell the signed-in user what the workflow will not do, and why, before you create it.

### 6. Add the files

Name every file in the process by its path, such as `assets/reminder.md`, and pass each one in `assets`:

- A text file, .md, .txt, .csv or .json: its path and its text.
- Any other file, a PDF, a .docx or an image: its path and `upload` true, then upload it to the `upload_url` in the answer, or give that link to the signed-in user to pick the file.

Keep long or shared material in files. A large handbook or knowledge base belongs in a Files or Web pages source, not in a file.

### 7. Review and create

Show the signed-in user the process text you wrote, in readable form, and the settings: the name, who it runs for, what starts it, the sources, the files, the timing and the approvals. Once they agree, call `workflow_create`.

If the answer has validation errors, fix them and call again.

### 8. Finish the setup with the signed-in user

- If a trigger in the answer has `webhook_setup`, give the signed-in user every url and the steps: the workflow does not fire until they register them in their tool. Ask them to tell you when it is done.
- If a file waits for upload, make sure it is uploaded.
- Give the signed-in user the workflow's url.

### 9. Test and fix

Test before the signed-in user relies on it; testing matters more than saving credits. Read the article named `sessions` first, if you have not read it in this conversation.

1. Run it on a real but harmless case with `session_start`, "Run <name>. Context: <every detail the run needs>", or fire the trigger for real.
2. Follow it with `session_status` and read it with `session_read`.
3. Test the stories you walked through in step 2: the usual case, a missing value, a reply and no reply, and each Rule and Path.
4. Fix the process text with `workflow_update` and run again, until it does what the signed-in user expects.

## Change an existing workflow

1. Read it with `workflow_get`.
2. Change only what the signed-in user asked for, and pass only the fields that change.
3. **To change the process text or a setting, leave `assets` out**, and every file stays as it is. Pass `assets` only to change the files, and then pass the full list: each file to keep by its path only.
4. A change reaches running sessions too. Tell the signed-in user before you change a workflow that is live.

## Common mistakes

| Mistake | What happens | Avoid it by |
| --- | --- | --- |
| A source is named in the process but not enabled for the session | The Ask agent cannot find it and retries, spending credits | Putting every named source in the trigger's `sources` |
| A filter on a field the event does not carry | The trigger or subscription never fires | Reading the event's filter fields with `event` |
| Not saying where a run's data comes from | Every run stops and asks for input | Naming the source of each value in the process |
| Timing that is vague, such as "soon" | The Ask agent asks instead of scheduling | Writing an exact gap, a condition and a stop |
| A shared workflow using the creator's own source | The member it runs as cannot use it | Checking `member_source_list` for that member |
| Passing `assets` to change only the process text | Files can be lost | Leaving `assets` out |
| Creating the workflow before its sources are set up | It fails on its first run | Finishing step 4 first |
| Writing the process before discussing edge cases | It breaks on the first unusual case | Walking through the stories in step 2 |
