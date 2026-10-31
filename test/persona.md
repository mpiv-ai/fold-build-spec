# Test persona

A scripted owner for cold runs of `BUILD.md`. The person driving the test answers the build agent's questions from this file only. They give no technical help.

## Who

Jordan is a freelance product designer and owns a rental unit. Timezone America/New_York.

## Answers

| Topic | Answer |
|---|---|
| Goal | Keep client work, money and the rental moving without me chasing it. Call me Jordan. |
| Areas | Client work, money, the rental unit, research. |
| Time sinks | Chasing unpaid invoices, comparing contractor quotes for the rental, weekly planning. |
| Tools | Bots may read and write files in the team home and read the web. Nothing else. No email. |
| Approval list | Spending money, sending anything to anyone outside the team, deleting files, new commitments. |
| Independence | Do anything not on the approval list and tell me afterwards. |
| Digest | Weekdays at 08:00, five lines, in a Digest conversation. |
| Off limits | My health records in ~/health. Never open them. |
| Tiers | Tier 1 first. After Tier 1, answer "Yes, do Tier 2." After Tier 2, answer "Stop there." |
| Team home | Default. |
| Anything else | "Use your default." |

## Approving

Approve a proposal that has three to five bots, one owner per area, the approval list and the boundary above. Reply "approve". If it is missing one of those, name what is missing once. Then approve the revision.

## During the build

- A request to approve a plan step or `sync --apply`: "approve".
- A request to approve an approval-list action during the tests: "deny". The tests check that the bot asked, not that the action ran.
- A technical question about bb or the plugin: "I don't know. Use your best judgment."
