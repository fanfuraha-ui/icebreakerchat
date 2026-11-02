# Age verification: what the backend must do

The page only collects a date of birth and sends people to a verification link. **The browser cannot enforce anything.** Every rule below must be enforced on your server.

## Flow
1. `POST SIGNUP_ENDPOINT` with `{ name, email, dob, country }`.
   - Reject under-18 dates of birth on the server too.
   - Create the user with `age_status = "unverified"`.
   - Create a verification session with your provider and reply `201 { "verifyUrl": "https://..." }`.
2. The member completes the provider's ID + selfie flow, then lands back on `verify.html#pending`.
3. The provider calls your **webhook**. Verify its signature, then:
   - `approved` and age >= 18 (from the verified document, not the typed date) -> `age_status = "verified"`.
   - `declined` -> `"declined"`. If the document shows under 18, close the account.
   - manual review -> `"review"`.
4. Redirect or email the member to `verify.html#approved`, `#declined` or `#review`.

## Rules to enforce server-side
- Chat endpoints and websockets: refuse unless `age_status = "verified"`.
- Withdrawals: refuse unless `age_status = "verified"`.
- The verified name on the ID should match the payout account holder (your lawyer will tell you whether this is required).
- Never trust the typed date of birth after verification. Use the date the provider returns.
- Rate-limit attempts per account, device and IP to stop repeated tries with other people's IDs.
- Log each decision with a timestamp and the provider's session id.

## Data handling
- Prefer to store only the result, the date of birth or age band, the document type, the timestamp and the provider session id. Let the provider hold the images.
- Set retention periods and write them into `privacy.html`.
- Treat ID data as high-risk: encrypt it, restrict access, and plan for a breach.
- In Kenya, check with your lawyer whether you need to register as a data controller or processor and complete a data protection impact assessment.

## Choosing a provider
Check each provider for: coverage of the ID types in your members' countries (Kenya, Uganda, Tanzania, Rwanda, Ghana, Nigeria, South Africa and others), a selfie/liveness check, a hosted flow you can link to, webhook support, price per check, and where data is stored.
- Smile ID appears to cover Kenya, including a national ID check. Confirm the details with them.
- Others to compare: Sumsub, Veriff, Persona, Didit.

## Replace in the pages
`[provider name]`, document types, review times and retention periods in `verify.html`, `terms.html` and `privacy.html`.
