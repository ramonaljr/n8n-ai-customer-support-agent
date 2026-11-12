# AI Customer Support Agent

Customers get an answer at any hour, taken from the store's own records rather than guessed.

A support chatbot for an online outdoor gear store that answers from the store policies, looks up orders and stock, and passes refunds and upset customers to a person in Slack. Built with n8n, OpenAI and Airtable.

**Full case study:** [ramonvallejera.com/case-study/ai-customer-support-agent](https://ramonvallejera.com/case-study/ai-customer-support-agent)

![Process map](docs/process-map.svg)

## How it works

1. Customer writes in
2. Agent answers from store data
3. Refunds and handovers
4. A person decides
5. Logs and daily summary

Built an n8n chatbot on OpenAI that answers only from the store policies document and the real order and stock records in Airtable. It asks for the order number and email before sharing anything about an order. Anything involving money, and any upset or unusual case, goes to a person in Slack instead of being promised in the chat.

## What's in this repo

| File | What it is |
|---|---|
| `workflow.json` | The n8n workflow, 46 nodes (39 working nodes and 7 sticky notes), ready to import |
| `docs/process-map.svg` | The process map from the case study |

Triggers: the webhook `Chat Message In` (the website chat), `Team Request In` (called by the agent's `Request Team Action` tool), `Every Day At 17:00`, `Reload Policies On Start` and `Load Policies (Run Manually)`.

Fixed rules decide, not the AI: `Apply Rules` re-checks the order number and email, sends money requests (refund, replacement, exchange, cancellation, discount) on a verified order to Slack for approval, and hands everything else to a person. The approval is a Slack form, not a plain link, so a link preview can never approve anything.

## Set it up

1. In n8n, open **Workflows → Import from File** and choose `workflow.json`.
2. Create these credentials and select them on the nodes that need them:

   - Airtable (personal access token) (7 nodes)
   - Slack (OAuth2) (4 nodes)
   - OpenAI (3 nodes): the chat model and both embeddings nodes
   - Google Drive (OAuth2) (1 node)
   - Gmail (OAuth2) (1 node)
   - Header Auth (1 node): protects the chat webhook, so only your website can call it

3. Replace the placeholders with your own records:

   - `THIS_WORKFLOW_ID` (2×): the workflow calls itself, so use its own ID from the URL after import
   - `YOUR_AIRTABLE_BASE_ID` (7×)
   - `YOUR_AIRTABLE_TABLE_ID` (7×): each node keeps the original table name (`Orders`, `Products`, `Support Cases`) as a hint
   - `YOUR_SLACK_CHANNEL_ID` (4×): the channel that receives approvals, handovers, alerts and the daily summary
   - `YOUR_DRIVE_FILE_ID` (1×): your store policies and product FAQ document

4. In `Compose Customer Email`, change the test-address pattern: this portfolio version only ever emails `you+test-…@example.com` addresses.
5. Run `Load Policies (Run Manually)` once, test with a few chat messages, then activate it. The policy search reloads whenever the workflow starts or is published.

Tools: n8n, OpenAI, Airtable, Slack, Gmail, Google Drive.

## Notes

- Fernhill Outdoor Supply is a fictional sample client.
- Credentials, webhook IDs, pinned test data and account-specific IDs were removed before publishing. n8n assigns new webhook URLs on import.
- Built by Ramon Vallejera Jr., AI Automation Specialist.
