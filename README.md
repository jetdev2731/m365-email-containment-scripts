# m365-email-containment-scripts
Sanitized PowerShell playbook to contain mailbox misuse in Microsoft 365
Sanitized PowerShell playbook to contain and clean up suspected mailbox misuse in Microsoft 365 (Exchange Online + Microsoft Graph), with no tenant secrets hard-coded and no PII logged.

✅ Focus: stop auto-forwarding/redirect rules, clear mailbox-level forwarding, revoke sessions, force password change, and set org guardrails—then verify and report.
00-Inputs.ps1                # Shared helpers (logging, CSV loader, paths)
10-Exchange-Containment.ps1  # Disable risky inbox rules + clear forwarding
20-Graph-RevokeAndReset.ps1  # Revoke sessions + temp password reset
30-Tenant-Guardrails.ps1     # Block external auto-forwarding org-wide
40-Unblock-And-Resecure.ps1  # Post-remediation checklist (no changes)
90-Status-Report.ps1         # Verify and export current state
users.sample.csv             # Template: UserPrincipalName
Prerequisites

Windows PowerShell 5.1 or PowerShell 7+

Modules (scripts auto-install if missing)

ExchangeOnlineManagement

Microsoft.Graph.Authentication, Microsoft.Graph.Users

Permissions:

Exchange: Exchange Admin (or equivalent custom RBAC)

Graph: Directory.AccessAsUser.All (delegated), User admin permissions for password reset

Internet access to Microsoft 365 endpoints

No app registrations or client secrets required. Interactive sign-in prompts are used.
Quick Start

Download the bundle and extract anywhere (e.g., C:\Scripts\m365-email-containment-scripts).

Create users.csv from users.sample.csv and list the UPNs to process:

UserPrincipalName
alice@contoso.com
bob@contoso.com


Open PowerShell as Admin, set execution policy if needed:

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass


Run, in order:

.\10-Exchange-Containment.ps1
.\20-Graph-RevokeAndReset.ps1
.\30-Tenant-Guardrails.ps1
.\90-Status-Report.ps1


Review outputs in the .\Output folder:

Rules-Actions.csv, Forwarding-Actions.csv, PasswordReset-Actions.csv

Tenant-Guardrails.csv, Status-Report.csv

Logs\*.log

What Each Script Does
10-Exchange-Containment.ps1

Connects to Exchange Online.

Backs up inbox rules per user (JSON in Output\Backups\Rules).

Disables rules with any of:

ForwardTo, RedirectTo, ForwardAsAttachmentTo

External recipients (best-effort detection)

Clears ForwardingSmtpAddress and DeliverToMailboxAndForward.

Exports summary Rules-Actions.csv and Forwarding-Actions.csv.

20-Graph-RevokeAndReset.ps1

Signs in to Microsoft Graph.

Revokes all refresh tokens (forces re-auth across devices).

Sets a strong temporary password (random; user must change at next sign-in).

Logs only a masked preview (e.g., P@ssw0rd-******AB), never the full value.

30-Tenant-Guardrails.ps1

Connects to Exchange Online.

Ensures Hosted Outbound Spam Filter Policy blocks external auto-forwarding (On).

Ensures Exchange admin audit log ingestion is enabled.

Exports Tenant-Guardrails.csv.

40-Unblock-And-Resecure.ps1

No tenant changes. Prints a post-remediation checklist:

Enable MFA/Phishing-resistant auth

Review sign-ins, device compliance, security alerts

User security awareness steps (report suspicious emails, etc.)

90-Status-Report.ps1

Re-reads current user state:

Any remaining forwarding?

Any active risky rules?

Exports Status-Report.csv.

CSV Format

users.csv (same folder as scripts):

UserPrincipalName
user1@contoso.com
user2@contoso.com


Keep one UPN per line. Header must be UserPrincipalName.

Safety & Redaction

No raw passwords or secrets are written to disk.

Action logs capture what changed, not PII contents.

Rule backups are stored locally (JSON) for audit/rollback.

Temporary passwords are randomized and masked in logs.

Logging

Text logs in .\Logs\yyyyMMdd-HHmmss-<script>.log

CSV outputs in .\Output\

Backups in .\Output\Backups\

Common Questions

Q: Will this break legitimate internal forwarding?
A: The guardrail in 30-* blocks external auto-forwarding. Internal routing continues to work. Per-user mailbox-level forwarding is cleared; transport/mailflow rules are not altered.

Q: Do users get locked out?
A: Tokens are revoked and a temporary password is set with “must change at next sign-in.” If your tenant enforces MFA/Conditional Access, users will re-authenticate normally.

Q: Can I undo disabled rules?
A: Yes—see backups in Output\Backups\Rules. Re-enable selectively via EAC or Set-InboxRule.

Troubleshooting

Modules won’t import / install
Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force
Install-Module Microsoft.Graph -Scope CurrentUser -Force

You lack permissions
Use an Exchange admin role and a user admin role for pwd resets.

Graph throttling
Scripts already sleep between calls; try smaller user batches.

Contributing

Keep redaction intact (no PII, no secrets).

Follow the “one concern per script” structure.

PRs welcome (typos, docs, small features).

License

MIT — see LICENSE.
