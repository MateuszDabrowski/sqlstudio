# SQL Studio for Marketing Cloud Engagement

SQL Studio for MCE is an open-source rebuild of Salesforce Labs' Query Studio for Marketing Cloud Engagement (MCE). It runs entirely inside MCE as a Cloud Page App, with each user's own Marketing Cloud Engagement login and permissions. There is no external server, and no query or result leaves your account.

Read what it does, how it works and the FAQ on the [SQL Studio page](https://mateuszdabrowski.pl/sql-studio).

## What you get

- A SQL editor with T-SQL highlighting, autocompletion for Data Extensions, their fields and the System Data Views, and hover documentation.
- MCE-specific lint that catches what would make a run fail, with one-click fixes, and a formatter that follows the author's [SQL style guide](https://mateuszdabrowski.pl/docs/salesforce/marketing-cloud-engagement/sql/sql-style-guide/).
- Four query tabs that run at the same time, each on its own Query Activity from a pool of up to four per user, into a temporary Data Extension with a one-day retention. Date, number and boolean columns keep their types, and results page and export in full.
- Save as, Open and Save for Query Activities, and query history.

## Deploy

SQL Studio is one Cloud Page, two Code Resources and an Installed Package. An admin deploys it once, with no build step. Follow [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

| File | Name in Web Studio | What it becomes |
|---|---|---|
| `src/code-resources/sql-studio-backend.html` | SQL Studio Backend | a JSON Code Resource |
| `src/code-resources/sql-studio-frontend.js` | SQL Studio Frontend | a JavaScript Code Resource |
| `src/cloud-page/sql-studio.html` | SQL Studio | the Cloud Page users open |

To update, keep each file's settings section and replace everything below its "APP CODE" line. [CHANGELOG.md](CHANGELOG.md) lists what changed in each release.

## Refresh tokens

SQL Studio does not use OAuth refresh tokens, so a session lasts about 20 minutes. SQL Studio renews it in the background while you use it. [docs/EXTENDING.md](docs/EXTENDING.md) explains why, and what it takes to add them.

## Support

Report bugs and ideas in [GitHub issues](https://github.com/MateuszDabrowski/sqlstudio/issues).

## Licence

SQL Studio is licensed under the [European Union Public Licence 1.2](LICENSE), the EUPL 1.2. In short: any use that keeps the credit to me is fine, commercial use included.

- You can use it for free in any Marketing Cloud Engagement account, your company's commercial ones included.
- You can change it, and share it, changed or not. Installing it in someone else's account, such as a client's, counts as sharing it.
- A shared copy, changed or not, comes with its source code and keeps its copyright and licence notices intact. Those notices are the credit to me.
- A shared copy stays under the EUPL 1.2. Merged into a larger work, it may instead use a licence the EUPL lists as compatible, such as the GPL.
- The SQL Studio name and the MD logo are not licensed under the EUPL. An unchanged copy may keep them.
- A changed copy is your own version, so it gets your own name and logo. It keeps the credit to me in its copyright and licence notices, and says that it is changed, and when, as the EUPL requires. Filling in the settings sections is not a change.
- Presenting SQL Studio, changed or not, as someone else's work is not fine.
- Disputes about the licence go to the courts where I live, under Polish law (EUPL Articles 14 and 15).

This summary only explains what I mean by the licence. The text in [LICENSE](LICENSE) is what applies.

### Third-party components

These come with SQL Studio under their own licences, not the EUPL:

- [Salesforce Lightning Design System](https://www.lightningdesignsystem.com/) icons 2.29.1: 25 utility icons, unchanged, in SQL Studio Frontend's icon sprite - © Salesforce, Inc., [Creative Commons Attribution-NoDerivatives 4.0](https://creativecommons.org/licenses/by-nd/4.0/).
- [Monaco Editor](https://microsoft.github.io/monaco-editor/) 0.52.2 - © Microsoft Corporation, MIT. The Cloud Page loads it from the jsDelivr CDN, so it is not part of these files.

Built by [Mateusz Dąbrowski](https://mateuszdabrowski.pl).
