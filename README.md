# Town Fuss — Outreach Admin

A single admin webpage for sending cold-outreach emails to Town Fuss
businesses — replaces an earlier terminal-script version. Live at
**https://townfuss-outreach.web.app**.

This is its own separate repo/site, but it runs on the **same Firebase
project as the main Town Fuss app** (Pauls Valley, `town-talk-87ff7`) —
same Google Sign-in, same admin permission list, no new login or account
to manage. It's a second Hosting *site* attached to that one project, and
a handful of new Cloud Functions in the main app's `index.js` (deployed
to that project only, same "prove it on Pauls Valley first" rule the rest
of that project follows). See that repo's `outreach/` folder (superseded
by this) and its `.claude/plans/sparkling-humming-reef.md` for the full
history/reasoning.

## How it works

Two checkpoints, nothing sent without both:

1. **Load today's leads** — pulls candidates (unpaid Town Fuss business
   listings + anyone you've added manually), shown right on the page.
   Nothing is recorded as "contacted" yet at this point — uncheck/edit
   anything before moving on.
2. **Create drafts** — for whatever's still checked, creates real Gmail
   Drafts (From `info@titanbusinesspros.com`). *This* is the point
   someone's marked as contacted.
3. Open Gmail yourself, review the actual drafts, apply the
   **Outreach/Approved** label to whichever you want sent.
4. A Google Apps Script (see the main repo's
   `outreach/apps-script/SendScheduler.gs`) sends the approved ones
   starting 9:00 AM Central, 10 minutes apart, on its own.

## One-time setup still required before this fully works

The page itself is already deployed and the sign-in/admin-check/lead-list
parts work today. **Creating drafts won't work yet** — it needs a Gmail
API OAuth connection, done once:

1. Verify `info@titanbusinesspros.com` as a "Send mail as" address under
   `titanbuesinesspros@gmail.com` in Gmail settings.
2. Create a Google Cloud OAuth client (Gmail API enabled, Desktop app
   type) — see the main repo's `outreach/README.md` Step 2 for the exact
   click-path (same Google Cloud project/credentials work here).
3. Run `node outreach/authorize.js` (from the main Town-Talk repo) once
   to get a refresh token via the consent flow.
4. Store the three values as Cloud Functions secrets (only needs doing
   once — they're currently set to a placeholder so the functions could
   deploy at all):
   ```
   firebase functions:secrets:set GMAIL_OAUTH_CLIENT_ID --project town-talk-87ff7
   firebase functions:secrets:set GMAIL_OAUTH_CLIENT_SECRET --project town-talk-87ff7
   firebase functions:secrets:set GMAIL_REFRESH_TOKEN --project town-talk-87ff7
   ```
   (each prompts for the value — paste the real one from steps 2-3, not
   the placeholder)
5. Redeploy just that one function so it picks up the new secret values:
   ```
   firebase deploy --only functions:outreachCreateDraft --project town-talk-87ff7
   ```
6. In Gmail, create the label **Outreach/Approved** once (Settings →
   Labels → Create new label) — the Apps Script trigger looks for it.

## Deploying changes to this page

```
firebase deploy --only hosting --project town-talk-87ff7
```

No `--config` flag needed here (unlike the main Town-Talk repo's
multi-edition deploys) — this repo's `firebase.json` only has one
hosting target, already pointed at the `townfuss-outreach` site.
