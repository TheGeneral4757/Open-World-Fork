# Open World — Security Notes (for the game's owner)

Hey! I've been going through the code to plan the self-hosted backend. The game is really
impressive (the Voronoi globe, the 4-draw-call plot rendering, and the deterministic wars
are excellent work). While reading it I found some security issues you should know about
now, before the rebuild is done. Nothing here has been tested against or used on the live
game.

## The big ones

1. **Anyone can edit or delete the whole database.** The Supabase policies in
   `supabase-setup.sql` allow anyone to read, insert, update and **delete** every row. The
   key in the page is public, which is normal for Supabase, so the policies are the only
   protection, and right now they allow everything. Someone with browser devtools could
   wipe the world or give themselves unlimited money.

2. **Passwords are not safe.** Every player's browser downloads every account's password
   hash, and the hash (DJB2) is only 32 bits. It can be brute-forced in seconds to minutes,
   and short passwords can be recovered outright.
   👉 **Please tell players not to use a password they use anywhere else, and to change it
   anywhere they've reused it.**

3. **Admin is only checked in the browser.** Anyone can trigger the admin actions (set
   money, delete accounts, reset the world) directly, without the panel.

4. **Usernames can inject code.** Usernames go into the leaderboard and admin table
   without escaping. A name like `<img src=x onerror=...>` would run code in every
   player's browser, including yours.

## Quick fixes, if you want them before the rebuild

| Fix | Effort | Helps with |
|---|---|---|
| Escape usernames before putting them in `innerHTML` (the `clEsc()` helper already exists) | ~10 min | #4 |
| In `saveGame()`, don't fall back to `hashPassword('default')`; skip saving instead | ~5 min | Accounts silently getting the password "default" |
| Pin `@supabase/supabase-js@2` to an exact version | ~2 min | A bad library update breaking the game |
| Export the `openworld_data` table regularly (Supabase dashboard → Table → Export CSV) | ~2 min/day | Recovering if someone wipes it |
| Remove the public **delete** policy | ~15 min + testing | Mass deletion (the admin delete would need to use tombstones only) |

## The long-term fix

The underlying issue is that the browser is trusted with everything. A lasting fix is a small
server between the game and the database that makes the decisions (money, buying, battles,
slots), hashes passwords properly (e.g. argon2), and checks admin rights itself. Supabase can do
part of this with its own Auth + row-level-security policies + database functions.

Happy to answer questions about any of these.
