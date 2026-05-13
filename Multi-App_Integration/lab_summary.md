## Real-World Justification

A boutique real estate advisory team receives inbound enquiries daily via Telegram from tenants, investors, and landlords. Manually copying each message into a shared Google Sheet takes time, introduces errors, and delays logging until someone is available. This workflow removes that bottleneck by capturing each Telegram message as a structured row in the `Inbound Leads` sheet, giving the team a live tracker, cleaner audit trail, and faster follow-up.

## Success Criteria

- The Telegram and Google Sheets credentials both work.
- The workflow runs through the Google Sheets node without errors.
- The destination sheet receives rows that match the mapped fields.
- The mapping is explicit: timestamp, sender username, sender name, chat ID, message text, status, and source.
- The setup is useful for a real operations team that needs a lightweight lead logger without a full CRM.

## Integration Pair and Field Mapping

This workflow uses **Telegram Trigger → Set → Google Sheets**. The Telegram Trigger produces a nested JSON object, and the Set node flattens it into sheet-ready fields. The mapping is:

- `Timestamp` from `$now`
- `Sender Username` from `$json.message.from.username` with a `no_username` fallback
- `Sender Name` from `$json.message.from.first_name` + `last_name`
- `Chat ID` from `String($json.message.chat.id)`
- `Message Text` from `$json.message.text`
- `Status` as `New`
- `Source` as `Telegram`

The Set node is required because Google Sheets should receive flat values, not raw nested objects. Wrapping `Chat ID` in `String()` avoids precision issues with large or negative Telegram IDs.
