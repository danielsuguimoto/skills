# User-driven PR testing

The user performs all test actions, including UI-based setup, API requests, worker/job or CLI invocations, and shipped migrations, seeders, operations, and scripts, even when they are prerequisites. The agent prepares minimal non-UI prerequisites with existing generic tooling. Never run shipped artifacts or take over UI actions as hidden setup.

For each scenario, record exact navigation, field values, clicks or command, expected observation, and any persisted effect to verify. Show a short overview first; give details when the user reaches the scenario. Artifact instructions include the pre-check, command, and post-check. Inspect each artifact's target and pre-state and leave it pending for the user.

Guide one scenario at a time. State what setup is ready, give numbered actions and expected observations, and wait for the user's report. Use the interactive question tool at meaningful checkpoints, not after every click. Guide one retry for an apparent missed step; otherwise record the application failure without investigating beyond scope.

Never infer success from silence or take over an unperformed user action. A declined or unavailable action is `BLOCKED`; scenarios unreached when the user ends the session are `DEFERRED`. For artifacts, wait for the user's command output and verify the post-state. Supporting automated checks may run only if they do not execute shipped artifacts; give artifact-executing commands to the user.

Record the user's observation as UI evidence. Request a screenshot or exact error only when needed to resolve ambiguity or support a finding. Distinguish user observations from agent-executed checks in the report.
