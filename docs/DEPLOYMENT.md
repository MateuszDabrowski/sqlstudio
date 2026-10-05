# Deployment

Step-by-step deployment for a Marketing Cloud Engagement (MCE) admin. SQL Studio runs entirely inside MCE: one Cloud Page, two Code Resources and an Installed Package. There is no external hosting and no build step at deploy time. The release files are ready to paste.

The [SQL Studio page](https://mateuszdabrowski.pl/sql-studio) explains what each piece does and how they work together. This page only covers deployment.

**Publish nothing until step 4.** Each piece is published once, with its code and its settings already in place. A piece published earlier, for example with empty settings, can keep answering with that first version for a few minutes after you publish the finished one, and a sign-in against it fails. So step 2 only creates the pieces, to get their URLs, and step 4 publishes them.

## 1. Get the code

Download the release as one ZIP file, [sqlstudio-main.zip](https://github.com/MateuszDabrowski/sqlstudio/archive/refs/heads/main.zip), and unpack it. The three files you need are in its `src` folder. You fill in their settings in steps 2 and 3, and paste them into MCE in step 4.

| File | Name in Web Studio | What it becomes |
|---|---|---|
| `src/code-resources/sql-studio-backend.html` | SQL Studio Backend | a JSON Code Resource |
| `src/code-resources/sql-studio-frontend.min.js` | SQL Studio Frontend | a JavaScript Code Resource, with the styles built in |
| `src/cloud-page/sql-studio.html` | SQL Studio | the Cloud Page users open |

The same folder holds `sql-studio-frontend.js`, the same code with its comments, for reading. Do not paste that one: Marketing Cloud Engagement served it in 18 seconds against 2 for the `.min.js` on the author's account, so SQL Studio opened that much slower. Its first line says so.

To download one file at a time instead, use the download links on the [SQL Studio page](https://mateuszdabrowski.pl/sql-studio#deployment-guide), or open the file on GitHub and use the "Download raw file" button at the top right of its code.

Open the files in a code editor, such as the free VS Code. Do not use Word, TextEdit or another word processor: they can turn the code's straight quotes into curly ones, and the code then stops working.

To check that you have the right files, look at their first lines:

| File | Its first line |
|---|---|
| `sql-studio-backend.html` | `<script runat="server">` |
| `sql-studio.html` | `<script runat="server">` |
| `sql-studio-frontend.min.js` | `/* SQL Studio 1.1.0 - SQL Studio Frontend, ...`, with the release's version |

## 2. Create the pieces, without publishing

Web Studio > CloudPages. Open a CloudPages folder, or create one for SQL Studio, and create these three in it. Name each one exactly as in the "Name it" column. Save each one and open it: its URL shows at the top of the editor before any publish. Copy the three URLs, and do not publish the pieces yet.

If your account has private domains, CloudPages asks for URL settings as you create each piece:

1. **URL:** pick the domain.
2. **Site Key:** give each piece its own readable name, for example `sql-studio`, `sql-studio-backend` and `sql-studio-frontend`. It becomes the path in the piece's URL. Do not leave it blank: a blank Site Key makes the piece the root page of that domain, and a domain has only one.
3. **HTTPS:** turn it on. Marketing Cloud Engagement's own page runs on HTTPS, and browsers block an HTTP page inside it. HTTPS needs an SSL certificate for CloudPages on that domain, so pick a domain that has one.

| Name it | Create it with |
|---|---|
| SQL Studio Backend | Code Resources > New > JSON |
| SQL Studio Frontend | Code Resources > New > JavaScript |
| SQL Studio | New Landing Page, in Code View, with no layout |

Start SQL Studio from a blank page with no layout, as with any Cloud Page App. A layout, for example one with a Code Snippet block, wraps the page in its own HTML and styles, which break the app's full-height layout.

Then paste the three URLs into two of your downloaded files:

| In this file | On this line | Paste the URL of |
|---|---|---|
| `sql-studio-backend.html` | line 18: `var pageURL = 'SQL_STUDIO_PAGE_URL';` | SQL Studio, the Landing Page |
| `sql-studio-backend.html` | line 19: `var backendURL = 'SQL_STUDIO_BACKEND_URL';` | SQL Studio Backend |
| `sql-studio.html` | line 19: `var backendURL = 'SQL_STUDIO_BACKEND_URL';` | SQL Studio Backend |
| `sql-studio.html` | line 20: `var frontendURL = 'SQL_STUDIO_FRONTEND_URL';` | SQL Studio Frontend |
| `sql-studio-frontend.min.js` | none | nothing: it has no settings, and you paste it as it is in step 4 |

These settings are not a menu in Marketing Cloud Engagement. They are lines of code near the top of each file, under `1. CONFIGURATION`. Open the file in your code editor and find the line: search for its placeholder with Ctrl+F (Cmd+F on a Mac), for example `SQL_STUDIO_BACKEND_URL`, or go to the line number with Ctrl+G. Select the placeholder, paste the URL over it, keep the quotes around it, and save the file with Ctrl+S (Cmd+S on a Mac):

```js
var backendURL = 'SQL_STUDIO_BACKEND_URL';
```

becomes

```js
var backendURL = 'https://pages.example.com/sql-studio-backend';
```

## 3. Create the Installed Package

Setup > Platform Tools > Apps > Installed Packages > New. It needs SQL Studio Backend's URL from step 2, and it gives you the three values the Backend's settings still need.

### Marketing Cloud App component

- Login endpoint: SQL Studio Backend's URL.
- Logout endpoint: the same URL.

### API Integration component (Web App type)

- Redirect URI: the same Backend URL again.
- Scopes: `data_extensions_read`, `data_extensions_write`, `automations_read`, `automations_write`, `automations_execute`. Leave "offline access" off. SQL Studio does not request refresh tokens (see `docs/EXTENDING.md` if you want to add them back).

### Access

On the Installed Package's Access tab, grant the Business Units and roles that should see SQL Studio in the AppExchange (Marketing Cloud App) menu. For Business Units under the one that holds SQL Studio, see step 6.

### The three values for the Backend

After saving, paste three of the package's values into the settings of `sql-studio-backend.html`, the same way as the URLs in step 2:

| In this file | On this line | Paste |
|---|---|---|
| `sql-studio-backend.html` | line 20: `var clientID = 'CLIENT_ID';` | The API Integration's Client ID |
| `sql-studio-backend.html` | line 21: `var clientSecret = 'CLIENT_SECRET';` | The API Integration's Client Secret |
| `sql-studio-backend.html` | line 22: `var clientBase = 'API_BASE_URI';` | The tenant subdomain of the API Base URI, not the Client ID |

The tenant subdomain is the 28 characters that start with `mc`, between `https://` and `.auth.marketingcloudapis.com`, for example `mc563885gzs27c5t9-63k636ttgm`. Select it by dragging, as a double-click stops at its hyphen.

## 4. Paste the code, then publish

The Backend and the Cloud Page each start with a section called `1. CONFIGURATION`. Everything an organisation sets lives there, above this line:

```js
/* =================== APP CODE - replace from here on update =================== */
```

Steps 2 and 3 filled in the settings every deployment needs. The tables below list every setting. Leave the others as they are. The one choice to make is `historyDE`: whether query history stays in each browser, or is also kept in MCE, where other users can read it. The Frontend has no settings at all.

### Backend (`sql-studio-backend.html`)

| Setting | Value |
|---|---|
| `configVersion` | Leave as it is. It tells the code which settings to expect. |
| `pageURL` | The URL of "SQL Studio", the Cloud Page, from step 2 |
| `backendURL` | The URL of "SQL Studio Backend" itself (this Code Resource's own URL), from step 2 |
| `clientID` | Client ID, from step 3 |
| `clientSecret` | Client Secret, from step 3 |
| `clientBase` | The tenant subdomain, from step 3: 28 characters that start with `mc`, not the Client ID |
| `authDE` | `SQL Studio Auth Log`. Change it only for a Data Extension you created yourself under another name (see step 7), with the `ENT.` prefix when it is shared. |
| `errorDE` | `SQL Studio Error Log`, with the same rule as `authDE` |
| `historyDE` | Where each user's query history is kept. Empty, the default: in each user's own browser only. `SQLStudioHistory`: also in a Data Extension in MCE for 90 days, so it follows users to other browsers, but every user with Data Extension access in this Business Unit can read every user's queries. Choose before you publish (see step 7). |
| `allowedReferrers` | Optional list of URL prefixes allowed to call the Backend. Empty turns the check off. A referrer matches a prefix only when it is that prefix, or continues with `/`, `?` or `#` after it, so `https://host` does not allow `https://host.evil.io`. |
| `debugging` | `false`. `true` writes debug output into responses instead of logging to the error log, and is for troubleshooting only. |

### Cloud Page (`sql-studio.html`)

| Setting | Value |
|---|---|
| `configVersion` | Leave as it is. |
| `backendURL` | The URL of "SQL Studio Backend", from step 2 |
| `frontendURL` | The URL of "SQL Studio Frontend", from step 2 |
| `authDE` | The same value as the Backend's `authDE` |

The Backend refuses to run, and says so with `CONFIG_INVALID` naming the setting, while `pageURL`, `backendURL`, `clientID` or `clientSecret` is empty or still holds its placeholder. The Cloud Page shows a "The settings section is not filled in" card while its `backendURL` or `frontendURL` does. Neither shows the value.

### Paste and publish

Paste each whole file, with its settings filled in, into its piece. In your code editor, select the whole file with Ctrl+A (Cmd+A on a Mac) and copy it. In Web Studio, open the piece, click into its code, select everything already there the same way, and paste over it. Then save the piece.

| Piece | Paste |
|---|---|
| SQL Studio Backend | The whole of `sql-studio-backend.html` |
| SQL Studio Frontend | The whole of `sql-studio-frontend.min.js` |
| SQL Studio | The whole of `sql-studio.html` |

Publish the two Code Resources first and SQL Studio last. SQL Studio loads the other two, so this way it never opens before they are live. Code Resources also tend to go live sooner after publishing than Cloud Pages.

There is no separate error page. When Marketing Cloud Engagement sends the visitor back with `?error=`, the same Cloud Page skips the session check and shows a small "SQL Studio could not sign you in" block with a "Try again" link.

The Monaco Editor's `loader.js` is loaded from jsdelivr with a `<script integrity="sha384-..." crossorigin="anonymous">` tag pinned to `monaco-editor@0.52.2`. The browser refuses the file if it does not match the hash baked into the Cloud Page. `loader.js` then fetches the rest of the editor from the same pinned `https://cdn.jsdelivr.net/npm/monaco-editor@0.52.2/min/vs` path. Upgrading the version means regenerating the hash (`curl -s <loader.js URL> | openssl dgst -sha384 -binary | openssl base64 -A`) and updating both the Cloud Page and this note.

### Where the client secret lives

The client secret sits in the Backend's settings section. Opening the Backend's URL runs the code on the server and never returns its source. No response, log row or debug output carries the secret, an access token or the key derived from the secret. Users with access to Web Studio in this Business Unit can still open the Code Resource and read it, as with any Cloud Page App. The release files ship with placeholders only, so no real value ever reaches the repository.

### What the Backend reads through WSProxy

WSProxy is a way for server-side scripts to call MCE from inside MCE. Until Salesforce confirms whether its API limit counts WSProxy calls, SQL Studio counts them as API calls. A run's API call count includes them, and hovering the count shows how many went through the API and how many through WSProxy. The Backend uses it in its own Business Unit, the one that holds the Backend, and never signs in for another one. It reads a run's status, SQL Studio's own Query Activities and temporary Data Extensions, and the names, keys, folders and fields of that Business Unit's shared and synchronized Data Extensions. It never reads their rows. It reads a run's status this way only for the user who started the run, in any of that user's sessions. In the same Business Unit it also reads the rows of a run's temporary Data Extension for Export CSV, up to 2,500 rows at a time. Each 2,500-row read counts as one API call. A sorted result, a result of more than 100 columns and every session in a child BU export through the REST API instead. If the WSProxy read fails, the export reads all the rows again through the REST API. The confirmation before the export says how many API calls that takes. It also deletes a user's own old temporary Data Extensions when cleaning up, and creates SQL Studio's own folder and log Data Extensions during the first-run setup in step 7. Everything else that creates, changes or starts something for a user uses the signed-in user's own login and permissions. That covers the temporary Data Extension, the Query Activity, starting a run, Save, Save As and folders. When a WSProxy read fails, the Backend does the same step with the user's login, and the user sees no error. The list of the parent's shared Data Extensions has no such step, because the user's login in a child BU cannot see them. When that read fails, the sidebar's folder tree shows none of the parent's Data Extensions, and a search shows no "Parent BU" group. SQL Studio asks for the list again the next time it opens, or when the user clicks Reload under the sidebar's Data Extensions section.

## 5. First open

Code Resources and Cloud Pages take a few minutes to go live after publishing, so wait about five minutes after step 4.

1. Open SQL Studio from the AppExchange menu as a user granted access in step 3. If you are already logged in to MCE, you land on the editor with no login screen.
2. On this first sign-in, the Backend creates SQL Studio Auth Log. Once the app has loaded, it creates SQL Studio Error Log, moves both into the SQL Studio folder, and sets their retention: 1 day for SQL Studio Auth Log and 180 days for SQL Studio Error Log. The Status tab then says once "First run: SQL Studio created its Data Extensions in the SQL Studio folder." and nothing more, and there is nothing for you to do. Only if Marketing Cloud Engagement did not let it set the retention, a second notice asks you to set it by hand in each Data Extension's properties (see step 7).
3. Run `SELECT TOP 10 SubscriberKey FROM _Subscribers` and confirm it returns rows. In a child Business Unit, query any Data Extension you have instead.

If a check fails, see the troubleshooting table below.

## 6. Child Business Units

One deployment in the parent Business Unit serves its child BUs too. A user opens SQL Studio from the child BU, and every query runs there with their own login and permissions. Grant the child BUs on the Installed Package's Access tab (step 3).

**Shared Data Extensions.** With SQL Studio installed in the parent, a child BU user sees the parent's shared and synchronized Data Extensions in the sidebar's folder tree, in folders with the parent's own names, such as "Shared Items" and "Synchronized Data Extensions", with a SHARED or SYNCED badge on each. A search lists them in a group called "Parent BU", with each folder path under the name. Completion and inserts add the `ENT.` prefix for them, which MCE needs when a child BU queries its parent. With SQL Studio installed in a child BU instead, the sidebar lists only that BU's own Data Extensions.

The list shows every shared Data Extension of the parent. It cannot tell which ones are shared with the child BU. If a user picks one that is not shared with their BU, MCE's own error says so when the query runs.

**API calls.** It costs more API calls in a child BU, by design. SQL Studio reads a run's row count and its first 2,000 rows without an API call only in the Business Unit that holds the Backend. In that Business Unit, the Backend also checks a run's status, looks up SQL Studio's own Query Activity, cleans up old temporary Data Extensions and reads the rows for Export CSV through WSProxy. SQL Studio counts these calls as API calls too, and a run's breakdown lists them apart. A child BU session still uses the user's own login for those steps and for reading results, so each of them is an API call. In a test with 1.0.0, a run cost about 15 API calls from a child BU instead of about 10 in the parent. To avoid that for a child BU that runs many queries, deploy SQL Studio again inside that BU, with its own Installed Package, and give its users that one.

SQL Studio does not read a child BU's temporary Data Extensions from the parent. The Backend never switches into another Business Unit, so everything that creates, changes or starts something in a child BU runs with the user's own login.

## 7. Data Extensions

SQL Studio uses two Data Extensions, plus an optional third for history. It follows the Cloud Page App pattern described at [mateuszdabrowski.pl](https://mateuszdabrowski.pl/docs/salesforce/marketing-cloud-engagement/ssjs/snippets/sfmc-cloud-page-apps/).

### Created for you

On the first sign-in, the Backend finds that its sign-in log, `SQL Studio Auth Log` (the "AuthLog" below), does not exist, creates it with its description, and signs you in. The rest waits until the app has loaded, because doing it all during sign-in ran past Marketing Cloud Engagement's time limit on the first org deployment. The app then runs three short steps, one Backend call each:

1. It finds or creates the `SQL Studio` folder under Data Extensions, the folder the temporary results also use.
2. It moves the Auth Log into that folder and sets its retention to 1 day.
3. It creates `SQL Studio Error Log` (the "ErrorLog" below) in that folder, with its description and a 180-day retention.

All of it happens in the Business Unit that holds the Backend Code Resource, and the Status tab reports the result once. A step that fails runs again the next time the app opens. The app also checks for the Error Log every time it opens, and recreates it in the folder when it is missing, for example after someone deleted it. It skips names with the `ENT.` prefix, because it does not create shared Data Extensions, and it never changes a Data Extension that already existed. Two users signing in for the first time at the same moment both get in: whichever create loses finds the Auth Log already there and writes to it.

Setting a retention policy needs the "Data Extension | Manage Data Extension Retention" permission (Salesforce's DataExtension API reference). The Backend's own context lacked it on the first org deployment, so the app sets the retention with the signed-in user's token. If that user lacks the permission too, the app tells you to set it by hand: Auth Log to 1 day and Error Log to 180 days, in each Data Extension's properties. Do it straight away. The Auth Log's tokens are encrypted, and the 1-day retention keeps each row no longer than the day it is needed.

Create them yourself instead when you want them in another folder, under other names, or shared. Use the tables below, then set `authDE` and `errorDE` to match in step 4. An `ErrorLog` from another Cloud Page App built on the same pattern can be reused, as SQL Studio's rows are told apart by the `appName` value `SQLStudio`. An `AuthLog` from one can be reused only with the fields below, as SQL Studio stores hashed sessions, encrypted tokens and a user id: without the `userId` field, sign-in stops with "SQL Studio Auth Log is from an older release". When the Error Log is missing, SQL Studio creates it in the SQL Studio folder the next time the app opens.

### Query Activities

SQL Studio does not create a Query Activity during setup. A user's first run creates one, `SQL Studio - <user name> - <hash>`, in the `SQL Studio` folder under Automation Studio Queries, and later runs reuse it. Each user can have up to 4 of these, `SQL Studio - <user name> - <hash>` through `... - <hash> - 4`, one per query tab. A second one is created only the first time a run needs it, when the user runs queries in two tabs at the same time. A user who never does that keeps just the one activity. There is nothing to configure for this.

### AuthLog

| Name | Data type | Length | Nullable |
|---|---|---|---|
| session (Primary key) | Text | 64 | No |
| appName | Text | 100 | Yes |
| createdDate | Date | | Yes |
| token | Text | 2000 | Yes |
| tokenExpire | Date | | Yes |
| userName | Text | 100 | Yes |
| userEmail | Text | 254 | Yes |
| userId | Text | 100 | Yes |

`session` holds the SHA256 of the session id, 64 lowercase hex characters, never the id itself: the browser keeps the id, and the Backend and the Cloud Page hash it before every lookup. `token` holds the access token encrypted with AES through AMPscript's `EncryptSymmetric`, as base64 text about a third longer than the token. The key comes from the Backend's `clientSecret`, so there is nothing new to set, and a new secret ends the sessions signed in under the old one. `userId` is the signed-in user's Marketing Cloud Engagement user id (`user.sub` from `/v2/userinfo`, or its `preferred_username` when there is none), which every per-user key derives from. `userName` and `userEmail` are for display only.

Retention: **Individual records, 1 day**. A row only needs to live as long as the session it backs, about 20 minutes. There is no `refreshToken` column, because SQL Studio does not request or store refresh tokens. See `docs/EXTENDING.md` for why.

### ErrorLog

| Name | Data type | Length | Nullable |
|---|---|---|---|
| id (Primary key) | Text | 36 | No |
| appName | Text | 100 | Yes |
| errorMessage | Text | 2000 | Yes |
| errorDescription | Text | 2000 | Yes |
| errorDate | Date | | Yes |

Retention: **Individual records, 180 days**. Error rows are diagnostic, not credentials, so they can live longer than AuthLog rows.

SQL Studio writes `createdDate` and `errorDate` itself, so the fields need no default value.

### SQLStudioHistory (optional)

Server-side history keeps each user's runs in a Data Extension as well as in the browser. History then survives cleared site data and follows the user across devices. The rows hold each query's full SQL text, including any values written into it, such as an email address in a `WHERE` clause. MCE has no folder-level restrictions for Data Extensions, so every user with Data Extension access in the Business Unit that holds SQL Studio can read every user's history, child BU users' queries included. Each user's History dialog says where their history is kept. Turning it on starts the history in MCE from the next run: entries already in a browser stay in that browser, and are not copied. To turn it on, set `historyDE = 'SQLStudioHistory'` in step 4. SQL Studio creates the Data Extension on the first run that saves history, with a 90-day retention on individual records.

| Name | Data type | Length | Nullable |
|--|--|--|--|
| id (Primary key) | Text | 36 | No |
| userId | Text | 100 | No |
| userEmail | Text | 254 | Yes |
| createdDate | Date | | Yes |
| sql | Text | (none) | Yes |
| rowCount | Number | | Yes |
| durationMs | Number | | Yes |
| status | Text | 20 | Yes |
| deKey | Text | 36 | Yes |

Leave the `sql` field length blank if you create it yourself, so long queries fit. Prefix `historyDE` with `ENT.` when it lives in a shared folder. Each user's rows are found by `userId`. `userEmail` is stored beside it for anyone reading the Data Extension.

### What SQL Studio stores and who can read it

- **AuthLog**: the SHA256 of each session id, an access token valid for about 20 minutes and stored encrypted, and the signed-in user's id, name and e-mail. Any user with Data Extension access in the Business Unit can read it, but not use it: the hash does not open a session, and the token is unreadable without the client secret. Keep the 1-day retention anyway, which limits how long the row exists.
- **ErrorLog**: backend error messages and descriptions, cut to 2000 characters each. They can include fragments of Marketing Cloud Engagement API error responses, but never a token, a session id or the client secret.
- **The parent's shared Data Extensions, for child BU users**: with SQL Studio installed in the parent, a user who opens it from a child BU sees the names, keys, folders and fields of all the parent's shared and synchronized Data Extensions, whether or not that BU has access to them. They never see the rows. The Backend reads this with its own access, not the user's. To keep it from a child BU's users, install SQL Studio in that BU instead (step 6).
- **SQLStudioHistory**: every user's own SQL text, when history is on. Any user with Data Extension access can read every other user's history. Query text does not include results, but it can reveal Data Extension and field names, business logic, or values written into a `WHERE` clause.

## 8. Updating

Update the Code Resources first and the Cloud Page last, for the same reason as in step 4.

1. **Frontend**: open "SQL Studio Frontend", replace its whole content with the new `sql-studio-frontend.min.js`, and publish. It has no settings.
2. **Backend**: open "SQL Studio Backend". Select from the `APP CODE - replace from here on update` line to the end of the file, paste the same range from the new `sql-studio-backend.html`, and publish. Your settings above the line stay as they are.
3. **Cloud Page**: do the same as for the Backend, when the release changed it.

A release that needs new settings raises `configVersion`, and the CHANGELOG says so. Until you update the settings, the Backend and the Cloud Page each show a message that their `1. CONFIGURATION` section is from an older release. Copy the new settings section from the release file, fill in your values again, and publish.

SQL Studio tells its users about a new release itself. Once a day per browser it reads `latest.json` from the public repository on GitHub, sending no cookies and no page address, and shows an Update badge in the toolbar when a newer version is out. The badge opens the release's key changes and a link to this section. A browser that cannot reach GitHub simply never shows it.

Code Resources and Cloud Pages take a few minutes to go live after publishing. Until then, a mix of old and new versions can answer, and an old Cloud Page with a new Backend can even send users round in circles. Wait about five minutes after publishing before you test, then do a hard refresh.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| The file in your code editor starts with `<!DOCTYPE html>`, mentions `github.githubassets.com`, and has none of the lines step 2 names | It is GitHub's web page, saved from the browser, not the release file | Download the ZIP from step 1 and use the files in its `src` folder. |
| SQL Studio Backend or SQL Studio answers with an error page, or nothing at all, right after you pasted it | A setting lost one of its quotes or its closing `';`, or a word processor turned the straight quotes `'` into curly ones | Edit the settings again in a code editor. Each value sits between two straight quotes, and each line ends with `';`, as in step 2. |
| Creating a piece in CloudPages asks for a URL and a "Site Key" | The account has private domains, so each piece gets a URL on one of them | Pick the domain, give each piece its own Site Key such as `sql-studio-backend`, never a blank one, and turn HTTPS on (step 2). |
| Opening SQL Studio ends on a browser error such as "...auth.marketingcloudapis.com's server IP address could not be found", or on raw JSON with `CONFIG_INVALID` | The Backend's `clientBase` is not the tenant subdomain: most often only its part before the hyphen, since a double-click stops selecting there, or the Client ID, a full URL or the placeholder. The Backend refuses a value that cannot be a subdomain and says which mistake it looks like, without showing the value | Copy the 28 characters that start with `mc` from the API Integration's Authentication Base URI into `clientBase`, publish SQL Studio Backend again, and wait a few minutes before you open SQL Studio. |
| Sign-in ends on Marketing Cloud Engagement's own error about the redirect URI, or SQL Studio says it signed you in but cannot find the session | The Backend's `backendURL` does not exactly match the Redirect URI on the API Integration component (step 3), or the Cloud Page and the Backend use different `authDE` values | Compare both URLs character by character, including `https://` and any trailing slash. Confirm both `authDE` settings name the same Data Extension. |
| Blank page, or the editor area never appears | The Monaco Editor CDN (`cdn.jsdelivr.net`) is blocked by a network policy, or the pinned file no longer matches the integrity hash in the Cloud Page | SQL Studio falls back to a plain text box with a warning after 15 seconds when the CDN is unreachable. With no fallback and no warning, open the browser console. Check whether the `frontendURL` Code Resource (SQL Studio Frontend) failed to load (a 404 or an unpublished resource), or whether the browser blocked `loader.js` for failing its integrity check. |
| "SQL Studio could not sign you in" instead of the editor | Marketing Cloud Engagement refused the sign-in or the token exchange | Read the error text on the page, and the `ErrorLog` row, which carries Marketing Cloud Engagement's own error code (for example `invalid_client` for a wrong Client ID or Client Secret). It is usually a mismatched Redirect URI or a misconfigured Installed Package. Fix the cause, then use the page's "Try again" link. |
| "SQL Studio could not sign you in", saying it could not save your sign-in because of its AuthLog Data Extension | SQL Studio could not create or write AuthLog: `authDE` has the `ENT.` prefix and does not exist, or the name is taken by a Data Extension with other fields | Create the Data Extension yourself from the tables in step 7, or point `authDE` at one that matches them. |
| "SQL Studio Auth Log is from an older release: delete it in Contact Builder, and SQL Studio creates it again on the next sign-in" | The Auth Log has the layout of a release before 1.0: no `userId` field, a 50-character `session`, a 520-character `token`. The Backend cannot write a hashed session and an encrypted token into it | Delete `SQL Studio Auth Log` in Contact Builder and sign in again. Do the same for the history Data Extension if `historyDE` is set and a call answers `HISTORY_OLD_LAYOUT`. |
| "Marketing Cloud Engagement did not tell SQL Studio which user you are" | `/v2/userinfo` returned neither `user.sub` nor `user.preferred_username` (or one over 100 characters) for that user, so SQL Studio has nothing to tell users apart by. It refuses the sign-in instead of sharing one user's objects | Read the `ErrorLog` row, which names the HTTP status but never the value. Report it in [GitHub issues](https://github.com/MateuszDabrowski/sqlstudio/issues) with the org type. |
| "SQL Studio cannot find your sign-in", after a few quick automatic retries | The Backend stored the session, but the Cloud Page cannot find it: the two `authDE` settings name different Data Extensions | Make both `authDE` settings match, publish both, and open SQL Studio from the AppExchange menu again. |
| "SQL Studio cannot read its sign-in log" | AuthLog was deleted or renamed, or the Cloud Page's `authDE` does not match the Backend's | Open SQL Studio from the AppExchange menu, which signs in through the Backend and recreates a missing AuthLog. Otherwise make both `authDE` settings match. An AuthLog from an older release also lands here: delete it once and sign in again. |
| "The settings section is from an older version" on the Cloud Page, or a message that the Backend's "1. CONFIGURATION" section is from an older release (as raw JSON on sign-in, or in a banner saying SQL Studio cannot run until the admin updates SQL Studio Backend) | The Cloud Page or Backend code was updated without its settings section, and the new release needs a new setting | Copy the new settings section from the release file, fill in your values again, and publish. |
| `SESSION_EXPIRED`, or a Session expired or Session ending dialog | The access token (about 20 minutes, of which SQL Studio uses about 15) expired, and the background renewal did not run or failed. It does not run after 15 minutes without activity, and it fails once you sign out of Marketing Cloud Engagement | Click Renew. It signs in again with one click and keeps your query text and results. The console lines prefixed `[SQL Studio]` give the time left after sign-in and the reason a background renewal failed. |
| The browser console shows `408 (Request Timeout)` for the Backend URL, and a run waits long before it starts | A Backend call ran past Marketing Cloud Engagement's time limit, and Marketing Cloud Engagement stopped it | Filter the console by `[SQL Studio]`: each slow or failed call is listed with its time. Report those lines in [GitHub issues](https://github.com/MateuszDabrowski/sqlstudio/issues). The rest of the console, the "[Report Only]" security messages, comes from Marketing Cloud Engagement's own page and does not affect SQL Studio. |
| Status line: "This org does not allow retention on API-created Data Extensions" | The user lacks the "Data Extension \| Manage Data Extension Retention" permission | Grant it if temporary Data Extensions should expire on their own. Otherwise SQL Studio deletes the user's own leftovers each time the app opens, once they are more than six hours old. |
| A child BU user sees no parent Data Extension in the sidebar's tree, and no "Parent BU" group in a search | SQL Studio is installed in the child BU itself, so it can list only that BU's own Data Extensions. Or the parent has no shared or synchronized Data Extensions. Or the Backend's read of the parent's list failed. Or the list was loaded before one was added | Install SQL Studio in the parent (step 6), or click Reload under the sidebar's Data Extensions section. |
| 404 on `/automation/v1/queries` from a child Business Unit | Community reports mixed results calling these endpoints from a non-top-level Business Unit | Confirm the user's Automation Studio permissions in that Business Unit, and that the Installed Package is granted to it (step 3). If it still fails, report it in [GitHub issues](https://github.com/MateuszDabrowski/sqlstudio/issues). |
| Temporary `SQLStudio_*` Data Extensions still exist after two days | Retention was not applied (see the permission row above), or MCE's retention flags behave differently than assumed | Open SQL Studio as the user who ran them, which deletes that user's leftovers older than six hours, or delete them in Contact Builder. Report it in [GitHub issues](https://github.com/MateuszDabrowski/sqlstudio/issues). |
