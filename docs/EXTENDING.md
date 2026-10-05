# Extending SQL Studio

Notes for anyone adapting the Backend beyond what ships in `src/code-resources/sql-studio-backend.html`. Nothing on this page is needed to deploy or run SQL Studio. See `docs/DEPLOYMENT.md` for that.

## Refresh tokens (optional, not shipped)

SQL Studio does not request or store OAuth refresh tokens. A session lasts as long as Marketing Cloud Engagement's access token, about 20 minutes. While the user works, SQL Studio signs in again in the background a few minutes before the session ends, and after a break their next action does the same. Only when that fails, for example because they signed out of MCE, does a Session expired dialog ask them to renew with one click.

### Why they are not shipped

The Auth Log Data Extension is readable by every user with Data Extension access in the Business Unit. SQL Studio stores each session under the SHA256 of its id, and the access token encrypted with AMPscript's `EncryptSymmetric`, bound to its own row and user, so reading the Auth Log gives nothing a caller can use. The key is derived from the client secret, though, and anyone who can open the Backend in Web Studio can read the client secret.

For an access token that exposure lasts about 20 minutes. A refresh token stays usable for 30 days and rotates on use, so the same exposure would last a month, and one read would be enough. Keeping the Auth Log to short-lived tokens, with its 1-day retention, is what makes the design safe without a key stored outside the Backend.

### What adding them takes

1. Turn on "Offline access" for the API Integration component of the Installed Package.
2. Append `offline` to the `scope` of the authorize request: the existing scopes plus `%20offline`.
3. Add a `refreshToken` field to the Auth Log: Text, 2000 characters, nullable. Add it to `authLogFields()` as well, so a new Auth Log gets it.
4. Store the refresh token encrypted, exactly as the access token is. `encryptToken(token, sessionKey, userId)` and `decryptToken(cipherText, sessionKey, userId)` bind a ciphertext to its row and user, and the same pair works for a second token. Never store a refresh token in plain text.
5. Consider a key that does not come from the client secret: a Key Management symmetric key (Setup > Key Management), passed to `EncryptSymmetric` and `DecryptSymmetric` by its external key instead of the `@null` and the derived password the Backend uses today. Then reading the Backend's code no longer gives the key.

Never write the AMPscript openers `%%[` or `%%=` literally anywhere in the Backend or the Cloud Page. Marketing Cloud Engagement evaluates those sequences wherever they appear in the file, not only inside the SSJS block. Build them from two pieces instead, as the Backend's `cryptToken` does: `'%' + '%='`. The same goes for a personalization string such as `%%memberid%%`, which the Backend writes as `'%' + '%memberid%' + '%'`.

### The refresh itself

It belongs in the POST flow's session check, where an expired session now ends with `SESSION_EXPIRED`:

1. Read the stored refresh token with `decryptToken`, from the row found by the session key.
2. POST `grant_type=refresh_token`, with `client_id`, `client_secret` and the refresh token, to `{authBase}/v2/token`.
3. On success, write the new access token and refresh token, both through `encryptToken` with the row's own session key and user id, and the new `tokenExpire`, with `UpdateData` keyed on the session key.

Wrap the write in `try` and `catch`. If an Auth Log lacks the `refreshToken` field, the session should fall back to today's behaviour, not fail on every request.

### Two requests at the same moment

The app checks run status on a timer, so two requests often arrive together after the token expired. Both read the same refresh token and send it. Marketing Cloud Engagement answers the first with a new pair, and refuses the second, as the refresh token has rotated.

So on a failed refresh, read the row again before giving up. If another request has already stored a newer token, its `tokenExpire` is in the future: decrypt that token and carry on with it. Only a refresh that fails with no newer token in the row ends the session.
