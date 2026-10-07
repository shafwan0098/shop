# SHAFWAN’S store — setup (about 30 minutes)

Static storefront (`/`) + owner-only admin (`/admin/`) on Supabase. No build step.
Prices, stock, discounts, delivery fees and totals are decided **inside the database** (`place_order`); the browser only sends variant IDs and quantities.

## 1. Supabase
1. Create a project at supabase.com (pick a region near Bangladesh, e.g. Singapore).
2. SQL Editor → run `supabase/migrations/001_schema.sql`, then `002_extras.sql`.
3. **Authentication → Providers → Email:** turn **OFF "Allow new users to sign up"**. Authentication → Policies: set a long minimum password.
4. Authentication → Users → **Add user** (your email + a strong unique password, auto-confirm).
5. Copy that user's UUID and run in the SQL Editor: `insert into admins (user_id) values ('PASTE-UUID');`
6. Turn on **MFA/2FA** for your Supabase account itself, and use a password manager.
7. Project Settings → API: copy the **URL** and the **anon public** key.

## 2. Site
1. Put the URL and anon key into `config.js`. **Never** put the `service_role` key anywhere in this folder.
2. Download the Supabase client once, so no third-party script is loaded at runtime:
   `mkdir -p vendor && curl -L https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.45.4/dist/umd/supabase.js -o vendor/supabase.js`
3. Deploy this folder to Cloudflare Pages or Netlify (drag-and-drop works). `_headers` already adds CSP, HSTS, no-framing and `noindex` for `/admin/`.
4. Connect your domain, then in Supabase → Authentication → URL Configuration set the Site URL to it.

## 3. First run
1. Open `https://yourdomain/admin/` (there is no link to it anywhere on the storefront). Sign in.
2. Settings → set the real size chart, phone, social links, delivery fees. Products → upload real photos; the demo hoodies are hidden-able/deletable.
3. Place a test order on the storefront, confirm it shows in Admin → Orders and stock drops; cancel it and confirm stock returns.

## Security model
- Admin access = Supabase Auth login **and** a row in `admins`, enforced by Row Level Security on every table. Hiding the URL is not the protection; the database is.
- The public can read only active products, images, variants, size chart and store settings. Customers, orders and discount codes are not readable without admin login.
- Orders can only be created through `place_order()`, which validates phone/quantities, locks stock rows (no overselling) and recomputes everything server-side.
- Uploads go to the `product-images` bucket; only admins can write. Images are resized/converted to WebP in the browser before upload.
- Admin auto-signs out after 30 min idle; failed sign-ins are slowed down (and rate-limited by Supabase).

## Not included yet (be aware before launch)
Online payments (bKash/Nagad/SSLCommerz — schema is ready for it), email/SMS notifications, wishlist, separate product/FAQ/policy/contact pages (write your real Privacy, Terms and Return policies — I did not invent any), image gallery on the product popup, bot protection on checkout (add Cloudflare Turnstile before heavy advertising), per-product SEO pages. This code was written without being able to run it against a live Supabase project: do the test order in step 3 before announcing the shop.
