# Changelog

All notable changes to SQL Studio are documented in this file.

## 1.0.0 - 2026-09-29

The first public release. SQL Studio rebuilds Salesforce Labs' Query Studio as a Cloud Page App that runs entirely inside your Marketing Cloud Engagement account, with each user's own login and permissions.

### Highlights

- Four query tabs that run at the same time, each on its own Query Activity.
- Lint for what Marketing Cloud Engagement refuses, with one-click fixes, and a formatter that follows the author's SQL style guide.
- Results that keep their types: dates with their seconds, sorted as dates, numbers sorted as numbers, paged, and exported in full to CSV.
- No server outside your account: every call runs with your own login, and the code is open to audit.
- Sessions stored hashed and encrypted, and renewed in the background while you work.

### What is in it

- **Editor:** T-SQL highlighting, completion for the System Data Views, your Data Extensions and their fields, the known values of status and category fields, and hover documentation that links to the author's field references, Salesforce's documentation and Microsoft's T-SQL reference.
- **Checks before a run:** Validate uses Marketing Cloud Engagement's own query check, so what passes and which functions work match Automation Studio's Query Activities. SQL Studio's lint adds what that check misses, such as an unpaired apostrophe in a comment, a `FOR JSON` at the top level, a column name a Data Extension field cannot have, or a Text column compared with a Number one, with a fix for most.
- **Runs:** four tabs, each with its own query, results and Status, kept in your browser per Business Unit and user. A run writes into a temporary Data Extension with a one-day retention, in a SQL Studio folder, and a column that names a Date, Number, Decimal or Boolean field keeps that type. A run that fails while it runs says so, with the cause SQL Studio knows for it.
- **Results:** a paged, sortable grid. The first 2,000 rows read without an API call, and later rows in 2,000-row chunks. Export CSV reads every row, reusing what is already loaded.
- **Saving:** Save as, Open and Save for Query Activities, with a check of your columns against the target Data Extension, and history in the browser or, if your admin turns it on, in a Data Extension.
- **Deployment:** one Cloud Page and two Code Resources, "SQL Studio", "SQL Studio Frontend" and "SQL Studio Backend", and an Installed Package. The first sign-in creates the sign-in log, and the app then creates the error log and the SQL Studio folders itself. The settings are checked before the first sign-in, and a mistake is named, never shown.
- **Security:** the sign-in log keys each session by a hash of its id, and stores each access token encrypted and bound to its own row and user. Browser storage is per user. There are no refresh tokens: `docs/EXTENDING.md` explains why, and what adding them takes.
- **Updates:** once a day, SQL Studio checks the public repository for a newer release and shows an Update badge with its key changes.
