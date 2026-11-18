# MOFA Click to Dial — Turnstile removed (no other feature changes)

This package is a targeted update of the immediately previous **MOFA Global Click to Dial** package.

## Changes only
- Removes the Cloudflare Turnstile widget, its JavaScript, its request token, and the backend challenge verification.
- Leaves existing callback API URL, number parsing (Jordanian local and international E.164), consent checkbox, Genesys Data Action execution, CORS settings, rate limiting, translations and Messenger unchanged.
- Does NOT add OTP, CAPTCHA, new authentication, or other features.

## Update the deployed apps
1. **GitHub Pages website:** In `Yazandmo/MOFA-Genesys-Demo` replace the root `MOFA_Genesys_Bilingual_Aligned.html` with this package's HTML file (exact same name), commit on `main`.
2. **Separate Render callback API:** In the repository connected to `callback_api`, replace `app.py` with `callback_api/app.py` here. `requirements.txt` is unchanged; replace only if needed. Commit on its deployment branch.
3. **Render Environment:** Remove `TURNSTILE_SECRET_KEY` and `TURNSTILE_EXPECTED_HOSTNAME` if previously added. Keep existing GENESYS, origin, and other settings exactly as they are.
4. Wait for Render deploy to be `Live`. Hard refresh GitHub Pages. Test only on an authorized number and check POST status and Genesys results.

## Important risk
This website is public, and without phone ownership verification or a bot check, strangers may request calls to other people's phone numbers. The existing in-memory limiter is not strong protection and resets on restarts. International calls may incur charges. Keep this proof of concept closed to public use, and do not deploy as a real public callback service without an approved protection plan. No new protection has been added in this change.
