[README.md](https://github.com/user-attachments/files/34907438/README.md)
# ofefit backend (Phase 1: database)

Free-tier stack: Supabase (Postgres, login, row-level security). The public site and the admin
site will be static pages on a free host (Netlify or Cloudflare Pages) that talk to Supabase.

## Set up (about 20 minutes)

1. Create a free Supabase project at supabase.com. Save the database password somewhere safe.
2. Open SQL Editor, paste `supabase/migrations/001_init.sql`, and run it.
3. Create your staff login: Authentication > Users > Add user (email and password).
4. Make yourself the owner. In SQL Editor, replace the email:

   insert into public.staff (user_id, full_name, role)
   select id, 'Owner', 'owner' from auth.users where email = 'YOUR-EMAIL-HERE';

5. Add gate staff the same way, with role 'gate'. Gate staff can check members in but cannot
   confirm payments or see revenue.

6. Keep the project awake (free). Follow `KEEP-ALIVE-SETUP.md`. In short:
   - Upload this folder to a private GitHub repository, including the hidden `.github` folder.
   - Add two repository secrets under Settings > Secrets and variables > Actions:
     - `SUPABASE_URL`: your project URL, e.g. https://xxxxxxxx.supabase.co
     - `SUPABASE_ANON_KEY`: the **Publishable key** (starts with `sb_publishable_`). If your project
       still shows an anon key under the Legacy API Keys tab, use that instead.
   - Run the "ofefit keep-alive" workflow once from the Actions tab to check for a green tick.

Never use the **Secret key** (starts with `sb_secret_`) in GitHub, the website or any public place.

## What each piece does

| Thing | Who can use it | Purpose |
|---|---|---|
| join_membership | Anyone | Creates a pending member and returns a member code |
| submit_payment | Anyone with a member code | Records a MoMo transaction ID for confirmation |
| get_member_card | Anyone with a member code | Returns name, tier, status and expiry only (no phone) |
| confirm_payment | Owner and admin | Activates the membership and extends the expiry date |
| check_in | Owner, admin and gate | Records one check-in per member per session per day |
| revenue_summary | Owner and admin | Confirmed revenue by tier for a date range |
| members, payments, checkins | Staff (read only) | Direct reads, limited by role |

Nobody can read the member tables directly without a staff login, so the member list and phone
numbers stay private.

## Known limits (read before going live)

- **Weekly keep-alive is set up by the workflow** in `.github/workflows/keep-alive.yml`. Check it runs green each month.

- **Not tested yet.** The SQL has not been run against a live database. Run it in a Supabase
  test project first and fix any errors it reports.
- **Public sign-up has no spam protection yet.** Anyone can call join_membership repeatedly.
  Add a rate limit or a CAPTCHA before launch.
- **Founders count** is checked without a lock, so two people signing up at the same moment could
  both get the 25th spot. Acceptable for a small community, but it can be tightened.
- **Free projects pause after 7 days of no activity.** That would stop check-in and sign-up
  without warning. Either upgrade to Supabase Pro (about USD 25 a month) before launch or
  arrange a weekly ping.
- **MoMo is still manual.** Staff confirm each payment after checking the transaction ID in the
  MoMo app. Automatic confirmation needs your MoMo merchant or aggregator credentials. Those are
  Phase 2.

## Next steps

- Phase 2: port the public site and admin screens to call these functions (the same screens you
  have now, with the same wording).
- Phase 3: connect MoMo for automatic payment confirmation.
