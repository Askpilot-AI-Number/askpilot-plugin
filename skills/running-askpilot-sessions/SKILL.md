---
name: running-askpilot-sessions
description: Runs Askpilot workflows and follows their sessions to the end, answers what the Ask agent asks, schedules follow-ups, finds out why a run went wrong, and reviews a workflow's past sessions to improve it. Use when the user wants to run or start an Askpilot workflow, asks what a workflow or session did, whether something is waiting for them, wants to answer, approve, stop or reschedule a session, clean up sessions, asks why a workflow did not do what they expected, or wants to know how a workflow is doing and how to make it better.
---

# Running Askpilot sessions

This skill is the order of work for running Askpilot workflows and looking after their sessions. The facts live in the Askpilot knowledge base, so read them through the Askpilot tools and do not rely on memory for them.

The signed-in user is the Askpilot user you are working for, the one who signed in to Askpilot MCP; before they sign in, it is the person you are talking to.

The tools named below, such as `session_start`, are the tools of the Askpilot connector.

## Before you start

Read the articles named `get-started`, `sessions` and `ask-agent` with `article_get`, in full, if you have not read them in this conversation. Call `organization_list` to see their organizations. If the signed-in user belongs to only one organization, work in that one without asking; if they belong to more than one, ask which one to work in.

## Run a workflow

1. **Find a workflow the signed-in user can run.** Call `workflow_list` with `member_id` set to their id from `account_get`. Only a workflow whose `runs_as` is the signed-in user can be run by them.
2. **Read what a run needs.** Read the process text with `workflow_get` and note every value the run needs: the customer, a reference, a phone number or an email. Ask the signed-in user for anything missing; never invent a value.
3. **Select the sources.** Pass them in `source_ids`, never leave it out: the session can then use every source the signed-in user has access to, and may pick the wrong mailbox or number. Include:
   - every source the process names, by its exact name, as `source_list` returns it;
   - the sources in the `sources` of the workflow's triggers, from `workflow_get`, since those are the ones the workflow runs with;
   - every source whose events the process waits for, in its Events section, or the session cannot resume on them;
   - Web research or Deep web research, when a step needs the web.

   Check with `source_get` that each one is `connected`, and give the signed-in user the url of any that is not, before you start. Never put the API source in `source_ids`: it gives no subagent.
4. **Start it.** Call `session_start` with the message "Run <workflow-name>. Context: <every detail the run needs>" and the sources from step 3.
5. **Leave timing to the Ask agent.** If the signed-in user wants a follow-up at a later time, put it in the context, for example "follow up tomorrow at 9:00 if there is no reply", and the Ask agent schedules it. Set `auto_run` only when the signed-in user asks for it.
6. **Tell the signed-in user it has started**, with the session's url.

## Follow a session

- Check where it stands with `session_status`: after you start it or send it a message, or when the signed-in user asks. A run takes seconds to minutes; do not check it in a tight loop.
- When the status is no longer `running`, read what happened with `session_read`. Use `detail` `full` only when the question is what exactly was done in a tool. What it returns, the Ask agent's reasoning, when there is any, and its own steps such as reading its process file or its memory included, is meant to be seen: use it to understand why the Ask agent did something.
- Tell the signed-in user what happened in plain words: what was sent, to whom, what was decided, and what comes next.

What each status means:

| Status | What to do |
| --- | --- |
| `running` | Wait. Check again later. |
| `idle` | Nothing is waited for. Report what happened. It may still continue on its own on a schedule or on an event. |
| `awaiting_reply` | The Ask agent asked a question or asks for an approval. Follow "Answer the Ask agent" below. |
| Folder `done` | The work is finished. Read the outcome with `session_read`. |

## Answer the Ask agent

1. Read the question with `session_read`, including its context and options.
2. Show it to the signed-in user, and offer two ways to answer: they tell you, or they answer themselves in the Askpilot app at the session's url.
3. Send their answer with `session_send`, in their words, with the session's current sources from `session_status`.

An approval the process asks for comes the same way, as a question, a form to review or an email draft, and several can come at once. Answer each item, for example "Approve the email to the landlord, skip the SMS". A no or a skip is final: the Ask agent drops that action and does not ask again, so tell the signed-in user before you send it.

Never answer a question of the Ask agent on your own, even when the answer seems obvious.

## Change what happens next

| The signed-in user wants | Do this |
| --- | --- |
| A follow-up at a set time | Ask the Ask agent with `session_send`, for example "follow up tomorrow at 9:00 if there is no reply". To set it yourself, use `session_auto_run_set` with `once` and `run_at` on a 10-minute mark: the next mark at or after the time it is due, never an earlier one. |
| A repeating run | `session_auto_run_set` with `hourly`, `daily`, `weekly` or `monthly`, only when the signed-in user asks for it. |
| To stop listening for replies | Ask the Ask agent with `session_send`; no tool removes a subscription directly. |
| To stop the run in progress | Confirm with the signed-in user, then `session_stop`. It does not switch auto-run off. |
| To close finished sessions | Confirm which ones, then `session_mark_done`. It starts no run and spends no credits. While a session is done, its auto-run and the events it is subscribed to do not run it, but both are kept. |
| To reopen a session | `session_mark_open`. It starts no run. It comes back as it was: its auto-run runs it again and its subscribed events resume it, so tell the signed-in user first. One exception: a single run at an exact time that passed while the session was done does not happen late, and auto-run switches off. |

Read the current setting with `session_status` before you change auto-run, because a new setting replaces the one the Ask agent made.

## Find out why a run went wrong

When a workflow did something unexpected, did nothing, or stopped, work through this checklist.

**Be careful with conclusions about how Askpilot works.** You see Askpilot only through its tools and articles, so your idea of how a feature behaves can be wrong, and a wrong cause sends the signed-in user to fix the wrong thing. Before you name a cause, check it: find it in what the session shows, or in the knowledge base with `article_get`. Say what you checked and what you only suspect, and never state a guess about Askpilot as a fact. If you still cannot tell, you can ask Askpilot, but only with the signed-in user's approval first: asking starts a run in Askpilot and spends their credits, so never ask without it. Once they approve, start a new session without a workflow, with `session_start` and the question as its message, and read the answer. If it does not give you a solution you can check, say so, and ask the signed-in user to email support@askpilot.com; prepare the email for them, with the session's url, what happened, and what you checked.

```
Diagnosis progress:
- [ ] 1. Read the session
- [ ] 2. Find the cause
- [ ] 3. Fix the workflow
- [ ] 4. Run it again
```

**1. Read the session.** `session_status` for its status, sources, todos and the events it waits for; `session_read` with `detail` `full` for every action and `error` entry.

**2. Find the cause.** The usual ones:

| What you see | Likely cause |
| --- | --- |
| The Ask agent says it cannot find a tool or source, or retries | The source is not in the session's sources, or its name in the process is wrong |
| The workflow never started on its event | The workflow is switched off, the filter never matches, or the event is not registered in the tool (`webhook_setup`) |
| It stopped and asked for a value | The run's data was not passed in the context or the event, and the process does not say where to find it |
| A follow-up came at the wrong time | The timing in the process is vague, or a single run was not scheduled on the next 10-minute mark |
| It did something the signed-in user did not want | The process text does not say it must not, or a Rule is missing |
| An `error` entry | Read its text; a source may be disconnected, so check it with `source_get` |

**3. Fix the workflow.** Change the process text with `workflow_update`, and leave `assets` out so every file stays as it is. Or fix the source, the trigger's sources or the filter. Read the articles named `workflows` and `process-md` first if you have not read them in this conversation.

**4. Run it again** on the same case, and check the result. Repeat until it works.

## Improve a workflow from its sessions

When the signed-in user asks how a workflow is doing, or how to make it better, review its past sessions together, not one by one. Work through this checklist:

```
Review progress:
- [ ] 1. Find the workflow's sessions
- [ ] 2. Read them
- [ ] 3. Find what repeats
- [ ] 4. Agree the changes
- [ ] 5. Change the workflow and check the next runs
```

**1. Find the workflow's sessions.** No tool lists the sessions of one workflow, so find them by their first message. Agree the period with the signed-in user, for example the last 30 days, and call `session_list` with `since`, once for `open` and once for `done`. Read the first page of each session with `session_read`, and keep those whose first message names the workflow: a session a trigger or the API started opens with "Run process" and the workflow's name, shown as its first `event` entry; one started by hand opens with "Run <workflow-name>" when it was started the way this skill says. A workflow can also start later in a session, when the signed-in user asks for it in any message, and one session can run several workflows. So when a session's first message names no workflow, or names another one, read its later pages for a message that starts this workflow. In a session that ran several, review only the runs from the message that started this workflow until another one started. Tell the signed-in user what the review cannot see: sessions where the workflow was asked for without its name, and the sessions of a workflow that runs for another member, which only that member's `session_list` returns.

**2. Read them.** Read each session with `session_read`, `detail` `summary`, and use `full` only where you need to see what was done in a tool. Read its credits with `session_status`. Read the workflow itself with `workflow_get`, so you compare what happened with what the process says.

**3. Find what repeats.** One odd run is a case; the same thing in several runs is a fault in the workflow. Check each finding as "Find out why a run went wrong" says, before you call it a cause. Look for:

| What repeats | What to change |
| --- | --- |
| The Ask agent asks for the same value | Say in the process where the value comes from |
| Runs stop at the same step or show the same `error` | Fix the step, the source or its enabled tools |
| The signed-in user refuses the same approval | Change the draft, its asset or the rule behind it |
| Follow-ups get no reply | Change the timing, the channel or the message, or add a Stop rule |
| A case the process does not cover | Add a Rule or a Path for it |
| The same source is called again and again, or fails and retries | Check that it is enabled for the session and named exactly, that the tool it needs is on with `source_tool_set_enabled`, and that it is connected |
| A source looks things up before it acts, when the process already gave it what it needs | Tell it in the todo what to skip, and keep the look-up where the step needs it |
| The Ask agent searches for a value the run should have had | Pass it in the context or the event, or say where it comes from |
| Runs that find nothing to do | Use a follow-up at a set time instead of a repeating auto-run, filter events to the ones that matter, and add Done and Stop rules |
| Every run is fine, but the work waits on the signed-in user | Ask whether a step still needs approval |

For the credit rows, the article named `workflows` explains each fix under "Write the process text so each run costs only what the work needs".

**4. Agree the changes.** Tell the signed-in user what you found, with the number of runs for each finding and one example session's url, and the change you suggest for each. Change only what they agree to.

**5. Change the workflow and check the next runs.** Change it with `workflow_update` and leave `assets` out unless a file changes, as "Find out why a run went wrong" says. A change reaches running sessions too, so say so before you change a live workflow. Once new runs have happened, read them and check the change worked.

