# PSP agent identity and visibility inconsistency following computer rename and re-registration

Hello PowerSyncPro Support Team,

We would like to report inconsistent agent registration and visibility behavior following a computer rename, reuse of its original name, and subsequent agent reinstallation.

We are submitting this as a suspected defect for investigation, together with an enhancement request for handling computer renames during long-running coexistence projects. The sequence below describes an observed incident; we have not yet isolated which steps are necessary to reproduce the final state.

## Operational context

Our migration involves multiple sites and local IT teams over an extended coexistence period. During this time, computers may be renamed, moved between OUs, and relocated to other sites. Their previous names may also be assigned to different computers that are themselves in scope for migration.

We are aware that renaming a computer after its initial PSP agent registration can cause issues. However, preventing all such changes across all site teams for the entire project is not a practical operational constraint.

All computer and site names below are illustrative.

## Sequence of events

1. A computer named **MAN01**, located in the **MAN OU** at site MAN, received the PSP agent. The agent successfully registered with the PSP server under that name.
2. The same computer was subsequently renamed to **LIB02** and moved to the corresponding OU for site LIB. It also appears to have been physically relocated to the new site and network.
3. A different computer was then assigned the original name, **MAN01**.
4. Approximately one month later, we received a request to migrate **LIB02**. We had not been informed of the earlier rename and relocation.
5. When we checked LIB02 in PSP, its last reported agent contact was approximately one month old.

## Findings on LIB02

When inspecting the computer directly, we found:

- A short JSON response in the migration agent’s ProgramData directory indicating that the agent was not registered.
- The original computer name, **MAN01**, still present in the agent’s registry configuration.
- No PSK remaining in that configuration; certificate information from the original registration was still present.

This appeared to leave the client with registration information associated with its previous identity. We have not confirmed how the server was correlating that information with the current computer object or with the other computer now using the original name.

## Recovery actions and subsequent behavior

We performed the following actions, in this order:

1. Uninstalled the PSP agent from LIB02.
2. Deleted the registry branch containing the agent’s configuration and registration information.
3. Deleted the agent record named MAN01 from the PSP server.
4. Reinstalled the agent on LIB02.

LIB02 was still included in an existing migration batch. We did not remove it from that batch before reinstalling the agent.

After reinstallation, the agent successfully completed registration, obtained a client certificate, established a secure connection, received its assigned runbook, and immediately started migration.

The immediate migration was acceptable in this case, since the computer was already intended to migrate. We mention the batch membership because it may be relevant to the sequence that produced the visibility issue.

## Inconsistent state after re-registration

Following successful re-registration:

- LIB02 appeared in the migration dashboard.
- The agent generated and uploaded logs, which were accessible in PSP.
- The computer object had already been imported and remained visible through **Single Object View**.
- However, we could not find a corresponding agent entry for LIB02 in the **Agents** tab.

The central issue is therefore that PSP was communicating with the agent and processing its migration activity, while the agent was not discoverable in the Agents view.

We do not know whether this resulted from stale registration data, reuse of the original computer name, immediate runbook execution after registration, or a combination of these factors.

## Requested investigation and guidance

Could you please:

- Investigate the sequence above and explain how an actively communicating agent can be absent from the Agents view.
- Confirm how PSP associates agent registrations with computer objects after a rename, particularly when another computer subsequently receives the original name.
- Provide the supported recovery procedure, including whether the computer must first be removed from its migration batch and how to identify the correct stale registration without affecting a different computer using the old name.
- Confirm whether this behavior is a known issue and whether a fix is available or planned.

## Enhancement request

For future versions, we would like PSP to handle computer renames, OU/site moves, and reuse of previous computer names more reliably.

Ideally, an existing agent should remain correctly associated with the same underlying computer after a rename, while a different computer reusing its previous name should be treated separately. Where automatic reconciliation is not possible, PSP should expose a clear identity conflict and provide a supported recovery workflow.

Consistent visibility across Agents, the migration dashboard, and agent logs would also make these cases substantially easier to diagnose and resolve.

Best regards,  
Vlad
