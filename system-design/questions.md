# Architect sending 5 000 000 telegram messages with stopping and editing

* Create a campaigns table in Postgres (id, status = draft/running/paused/stopped/done, message_template, created_at, updated_at) and a campaign_recipients table (campaign_id, user_id, chat_id, state = pending/sent/failed, message_id nullable).

* When you start a campaign, insert 5M rows into campaign_recipients (or pre-generate them) and enqueue lightweight tasks into RabbitMQ (e.g. campaign_id, recipient_id) in batches.

* Have a pool of sender workers that consume from a send_campaign queue, and for each task: load the current campaigns.message_template, render per user, check that campaign.status = running, then send the message to Telegram.

* After a successful send, workers update campaign_recipients.state = 'sent' and store the returned message_id (per chat) so future edits are possible and retries are idempotent.

* To stop a campaign, set campaign.status = paused or stopped in Postgres; workers must re-read status before each send and skip/ack messages when campaign is not running, while the producer stops publishing new jobs.

* To edit the message for users not yet processed, just update campaigns.message_template; new worker tasks will pick up the latest text automatically on each send.

* To edit the message already sent to some users, enqueue separate edit_message tasks (with chat_id, message_id, campaign_id) into another queue, and workers call Telegram’s edit message API using the updated template.

* Use an outbox pattern or a dedicated outbox table to ensure “DB update + enqueue to RabbitMQ” are atomic from the app’s perspective, and periodically aggregate campaign_recipients to update overall progress (sent/failed/remaining) for UI/monitoring.
