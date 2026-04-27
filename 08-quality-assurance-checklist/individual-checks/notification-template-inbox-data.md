# Notification template inbox data (satellite table)

After migration `0329_inbox_variant_for_the_notifications` (and the Drizzle schema that matches it), **do not** read or write `notification_templates.inbox_message` — that column was removed.

## Adding or updating inbox copy in SQL or seeds

1. Insert into `notification_template_inbox_messages` (`body`, optional `conversation_strategy`).
2. Set `notification_templates.inbox_message_id` to the new row’s `id`.

Example shape:

```sql
INSERT INTO notification_template_inbox_messages (body, conversation_strategy)
VALUES ('Hi {{member_first_name}}, …', NULL)
RETURNING id;
-- then
UPDATE notification_templates SET inbox_message_id = <id> WHERE id = <template_id>;
```

Use `NULL` for `conversation_strategy` to mean “use the code default for this template’s `event_name`” (`EVENT_DEFAULT_STRATEGY` in `@bulqit/notifications`).

## Historical migrations

Older migrations that reference `inbox_message` on `notification_templates` are valid only **before** 0329 runs; new migrations must use the satellite table pattern above.
