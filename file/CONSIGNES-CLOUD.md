# Remplir Buffer : consignes de la routine cloud

You run in the cloud, every morning, with no access to the owner's computer. Your job is to keep
**10 scheduled posts per Buffer channel** (the free plan's maximum) for the faceless finance accounts
"Décrypte ton fonds", at the pace the owner chose on 17/11/2026:
**5 different videos a day per channel, at 09:00, 12:00, 15:00, 18:00 and 21:00 (Paris time).**

Everything you need is in this repository's `file/` folder. Do NOT modify the repository (no commit, no push).

## Buffer
Organisation `6af1b322f52fed9a0a8d08b9`. Three channels:

| key | channel id | language | queue file |
|---|---|---|---|
| instagram | `6af1b3d962b70b94b0ce96c5` (Instagram decrypte.ton.fonds) | English | `file/en.json` |
| tiktok | `6af1b4ea62b70b94b0ce9da2` (TikTok decrypte_ton_fonds) | English | `file/en.json` |
| instagram_fr | `6af3160662b70b94b0e1fab5` (Instagram decrypte.ton.fonds.fr) | French | `file/fr.json` |

The English videos never go to the French account, and the French videos never go to the English channels.

## Queue files
`file/en.json` and `file/fr.json` are lists, **in publishing order**. Each entry:
`fichier` (file name), `url` (public video URL to give Buffer), `texte` (ready caption, or null),
`script` (list of narration sentences) or, for a few older videos, `config` (the on-screen texts and numbers).

## Steps (do them for each of the 3 channels)
1. **What is already used.** `list_posts` for the channel with status scheduled, sending, sent and error
   (`first: 100`, follow `pageInfo.endCursor` until `hasNextPage` is false). For each post, the video file name
   is the last part of `assets[].source`. A video is **used** on this channel if it has a post that is
   scheduled, sending or sent. A video with **2 or more error posts** on this channel is **skipped** (report it).
   A video with only 1 error post is free again (it will be retried once).
2. **How many to add.** Missing = 10 − (number of scheduled posts on the channel). If 0, go to the next channel.
3. **Slots.** Slots are 09:00, 12:00, 15:00, 18:00, 21:00 Paris time. Start at the first slot strictly after
   the channel's latest scheduled post (or after now + 30 minutes if it has none), then take the following slots.
   Paris offset: `+01:00` until 28/03/2027 02:00, then `+02:00` (summer time) until the last Sunday of October.
4. **Which videos.** In queue order, the entries not used and not skipped on this channel. Instagram and TikTok
   follow the same English order.
5. **Caption.** If `texte` is not null, use it as is. Otherwise write one from `script` (or `config`), following
   `file/voice.md`:
   - English for `instagram` and `tiktok`; **French with "tu"** for `instagram_fr`.
   - First line of 125 characters or less, that makes people want to watch.
   - 2 or 3 short lines with the real numbers from the script. Never a number that isn't in the script.
   - ONE ask (save, send, follow or tag). Then 3 hashtags (French hashtags for the French account).
   - No em dashes, no curly quotes, no "free" / "gratuit", never mention the website or a fund to buy,
     no personalised investment advice, no hype words.
   - Use the same caption for Instagram and TikTok of the same English video.
6. **Schedule.** `create_post` with `channelId`, `schedulingType: "automatic"`, `mode: "customScheduled"`,
   `dueAt` (ISO with the Paris offset, e.g. `2026-11-22T09:00:00+01:00`), `text`, `assets: [{video: {url}}]` and:
   - instagram and instagram_fr: `metadata: {instagram: {type: "reel", shouldShareToFeed: true, isAiGenerated: true}}`
   - tiktok: `metadata: {tiktok: {isAiGenerated: true}}`
   If one post fails, report the error and continue with the next slot and video.

## Rules
- Never more than 10 scheduled posts per channel. Never the same video twice on the same channel.
- Never edit or delete an existing post in Buffer. Never touch Metricool.
- Never type a password. If a channel is disconnected, stop for that channel and report it.
- If a queue has no video left for a channel, that's normal: say so in the report (the owner must make more videos).

## Report (end of run)
Short, in French: for each channel, what was added (date, time, video), how many videos are left in its queue
(not used yet) and until which date the scheduled posts go; then any problem.
