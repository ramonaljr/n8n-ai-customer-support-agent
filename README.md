# AI WhatsApp Customer Support Agent

Customers get an answer at any hour, taken from the store's own records rather than guessed.

A WhatsApp support agent for an online outdoor gear store that answers from the store policies, looks up orders and stock, accepts photos of damaged items, and passes refunds and upset customers to a person in Slack. Built with n8n, the WhatsApp Business Cloud API, OpenAI and Airtable.

**Full case study:** [ramonvallejera.com/case-study/ai-customer-support-agent](https://ramonvallejera.com/case-study/ai-customer-support-agent)

![Process map](docs/process-map.svg)

## How it works

1. Customer writes in
2. Agent answers from store data
3. Refunds and handovers
4. A person decides
5. Logs and daily summary

Built an n8n support agent on WhatsApp, using OpenAI, that answers only from the store policies document and the real order and stock records in Airtable. It asks for the order number and email before sharing anything about an order. Anything involving money, and any upset or unusual case, goes to a person in Slack instead of being promised in the chat.

## What's in this repo

| File | What it is |
|---|---|
| `workflow.json` | The support agent, 46 nodes (39 working nodes and 7 sticky notes), ready to import |
| `whatsapp-channel.json` | The WhatsApp channel, 13 nodes (12 working nodes and 1 sticky note), ready to import |
| `docs/process-map.svg` | The process map from the case study |

The two workflows talk over one webhook: the WhatsApp channel posts each customer message to the agent's `Chat Message In` webhook and sends the reply back on WhatsApp. A website chat can use the same webhook, so one agent serves both channels.

**Support agent** (`workflow.json`). Triggers: the webhook `Chat Message In`, `Team Request In` (called by the agent's `Request Team Action` tool), `Every Day At 17:00`, `Reload Policies On Start` and `Load Policies (Run Manually)`.

Fixed rules decide, not the AI: `Apply Rules` re-checks the order number and email, sends money requests (refund, replacement, exchange, cancellation, discount) on a verified order to Slack for approval, and hands everything else to a person. The approval is a Slack form, not a plain link, so a link preview can never approve anything.

**WhatsApp channel** (`whatsapp-channel.json`). Trigger: `WhatsApp Message In`. `Read Message` keeps real customer messages and drops delivery and read receipts.

- **Text** goes to `Ask Fernhill Agent` with the sender's number as the conversation ID, so the agent remembers the conversation, and `Send Reply` sends the answer. If the agent fails, `Send Sorry` apologises instead of leaving the customer waiting.
- **Photos** (a damaged item, say) are saved to Google Drive, and the agent gets the link and caption, so the team sees the photo in the case.
- **Anything else** (voice notes, stickers) gets a short note from `Ask For Text` asking the customer to type.

## Set it up

### 1. The support agent (`workflow.json`)

1. In n8n, open **Workflows → Import from File** and choose `workflow.json`.
2. Create these credentials and select them on the nodes that need them:

   - Airtable (personal access token) (7 nodes)
   - Slack (OAuth2) (4 nodes)
   - OpenAI (3 nodes): the chat model and both embeddings nodes
   - Google Drive (OAuth2) (1 node)
   - Gmail (OAuth2) (1 node)
   - Header Auth (1 node): protects the chat webhook, so only your channels can call it

3. Replace the placeholders with your own records:

   - `THIS_WORKFLOW_ID` (2×): the workflow calls itself, so use its own ID from the URL after import
   - `YOUR_AIRTABLE_BASE_ID` (7×)
   - `YOUR_AIRTABLE_TABLE_ID` (7×): each node keeps the original table name (`Orders`, `Products`, `Support Cases`) as a hint
   - `YOUR_SLACK_CHANNEL_ID` (4×): the channel that receives approvals, handovers, alerts and the daily summary
   - `YOUR_DRIVE_FILE_ID` (1×): your store policies and product FAQ document

4. In `Compose Customer Email`, change the test-address pattern: this portfolio version only ever emails `you+test-…@example.com` addresses.
5. Run `Load Policies (Run Manually)` once, then activate it. The policy search reloads whenever the workflow starts or is published.

### 2. The WhatsApp channel (`whatsapp-channel.json`)

1. In [Meta for Developers](https://developers.facebook.com/), create an app with the **WhatsApp** product. Meta gives you a free test number; add your own phone as a recipient to try it.
2. Import `whatsapp-channel.json` and create these credentials:

   - WhatsApp OAuth API (1 node): the app ID and secret, for `WhatsApp Message In`
   - WhatsApp API (5 nodes): an access token and your WhatsApp Business Account ID, for sending replies and fetching photos
   - Header Auth (1 node): the same credential as the agent's `Chat Message In`
   - Google Drive (OAuth2) (1 node)

3. Replace the placeholders:

   - `YOUR_WHATSAPP_PHONE_NUMBER_ID` (3×): the phone number ID from the app's WhatsApp → API Setup page
   - `https://your-n8n-instance.example.com` (1×): your n8n address, in `Ask Fernhill Agent`
   - `YOUR_DRIVE_ID` (1×) and `YOUR_DRIVE_FOLDER_ID` (1×): where customer photos are saved

4. Activate it. n8n registers the webhook with Meta when the trigger is activated.
5. **Subscribe the WhatsApp Business Account to your app, or no messages arrive.** Activating the trigger does not do this. Call `POST https://graph.facebook.com/v21.0/{WABA_ID}/subscribed_apps` once with your access token, then send a message to the test number.

Tools: n8n, WhatsApp, OpenAI, Airtable, Slack, Gmail, Google Drive.

## Notes

- Fernhill Outdoor Supply is a fictional sample client.
- Credentials, webhook IDs, phone number IDs, pinned test data and account-specific IDs were removed before publishing. n8n assigns new webhook URLs on import.
- Built by Ramon Vallejera Jr., AI Automation Specialist.
