# Update existing MOFA GitHub Pages site (same URL)

**Scope:** Exactly one existing GitHub repo `Yazandmo/MOFA-Genesys-Demo` and the existing HTML path `MOFA_Genesys_Bilingual_Aligned.html`; **do not modify the separate WhatsApp callback project.** The website remains on GitHub Pages, and a **separate Render Web Service** handles Genesys OAuth and `MOFA_Create_Callback` execution. No new website and no new GitHub repo.

## 1. Publish the API (Render)

1. Upload the **entire package structure** to the existing GitHub repository's `main` branch. This creates `callback_api/` beside the existing HTML file; never upload secrets. If you want to preview UI first, upload only the HTML, but calls require the API.
2. In Render create a **new Web Service** named for example `mofa-github-dial-api` by connecting the **same GitHub repo** `Yazandmo/MOFA-Genesys-Demo`. Do **not** modify the old `mofa-callback` service.
3. Important Render setting: **Root Directory** `callback_api`, so Render runs only the Flask backend; it does not replace the GitHub Pages website.
4. Choose Python, Free for controlled test. **Build Command** `pip install -r requirements.txt`; **Start Command** `gunicorn --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 90 app:app`.
5. In **Render → Environment**, add:
   - `GENESYS_LOGIN_DOMAIN=login.mypurecloud.ie`, `GENESYS_API_DOMAIN=api.mypurecloud.ie` (ONLY if this matches your Genesys organization).
   - `GENESYS_CLIENT_ID`: restricted Client Credentials OAuth ID.
   - `GENESYS_CLIENT_SECRET`: newly ROTATED OAuth secret; keep private.
   - `GENESYS_CALLBACK_ACTION_ID`: exact **published action ID** for `MOFA_Create_Callback`, including any `custom_-` prefix from its URL. **Do not strip it down to a UUID**; screenshot showed a `custom_-` prefix and a Draft view, so verify the published version exists.
   - `ALLOWED_WEBSITE_ORIGIN=https://yazandmo.github.io` (origin only, NOT path).
   - `TEST_ALLOWED_NUMBERS=+96279XXXXXXXX` (your own approved test number; add more numbers separated by commas).
   - `ACTION_PHONE_FORMAT=e164` for `+962...` or `national` for `079...` IF your Action contract requires it.
6. Confirm **Live** and open `https://YOUR-RENDER-HOST.onrender.com/healthz`. Should show `{"status":"up","service":"mofa-github-pages-click-to-dial-api"}`.

## 2. Connect the GitHub Pages button

1. In the HTML file `MOFA_Genesys_Bilingual_Aligned.html` find:

   ```js
   const WEBSITE_CALLBACK_ENDPOINT = 'https://YOUR-CALLBACK-API.onrender.com/api/click-to-dial';
   ```

2. Replace `YOUR-CALLBACK-API` with **your new Render service hostname** only. Leave the `/api/click-to-dial` path unchanged. This is a **public backend URL only**; NO Genesys secrets in HTML.
3. In GitHub repo, **replace the existing `MOFA_Genesys_Bilingual_Aligned.html`** with the updated version (same case-sensitive filename). Commit to the `main` branch. Don't rename it `index.html`.
4. Open the existing URL: https://yazandmo.github.io/MOFA-Genesys-Demo/MOFA_Genesys_Bilingual_Aligned.html . It still uses the original Genesys Web Messenger deployment and saved-language reload behavior.

## 3. Test the website

1. English → beside **Chat with support**, click **Click to Dial** (phone icon); enter an approved Jordanian number `079...` or `+96279...`, tick consent and click **Request a call**.
2. Arabic → change language; page reloads so Genesys Messenger stays aligned; button label **اطلب مكالمة**; try the same form after cooldown.
3. Verify Render logs (no secrets) and Genesys Callback/Interactions to confirm `MOFA_Create_Callback` executed. **Request submitted** ≠ connected telephone call.
4. Confirm the original **Chat with support** still opens Messenger.

## Troubleshooting

- `403 ORIGIN_NOT_ALLOWED`: Use the real GitHub Pages URL. `ALLOWED_WEBSITE_ORIGIN` should be `https://yazandmo.github.io` (no `/MOFA-Genesys-Demo/...` path). Do not disable this check.
- `403 TEST_NUMBER_NOT_APPROVED`: add an approved test number to `TEST_ALLOWED_NUMBERS` in the new Render service.
- `502 GENESYS_ACTION_FAILED`: verify full prefixed Action ID, **Published** status, input contract (`phoneNumber`, `customerName`), Genesys region, OAuth permissions, whether the action's underlying callback operation works.
- The old Data Action was validated in a bot flow; this separate website runs it through the **Execute Action API**, not by triggering the bot flow. A published action must be executable with the OAuth client's division/permission access.
- `429 RATE_LIMITED`: five-minute per-phone cooldown; limit 2 calls/hour/phone, 4/hour/IP, 12/hour global per active process. Render Free restarts reset RAM.
- Render Free can spin down after idle and first request may be delayed; this is not production-grade.

## IMPORTANT SECURITY

This is a controlled POC. The GitHub Pages site is public; anybody who knows the test number could submit the form. CORS, checkbox and RAM throttling are not identity verification or strong anti-fraud safeguards. Restrict to your own approved test phone numbers. **Before allowing arbitrary public numbers, add OTP ownership verification, bot protection, persistent per-number quotas, monitoring, review and telecom fraud controls.** Credentials must remain in Render environment, not the public GitHub repository.

**The prior OAuth client secret exposed in chat must be rotated.** If the action's output depends on non-website context from Architect, update or test the data action contract accordingly.

## References

- Genesys Data Action execution: https://help.genesys.cloud/articles/request-configuration-data-actions/
- Genesys integration requirements: https://help.genesys.cloud/articles/requirements-for-the-data-actions-integration/
- Render Flask deployment: https://render.com/docs/deploy-flask
