Email verification with ZeroBounce API only if `ZERO_BOUNCE_API_KEY` env variable is configured (it's a feature flag at the same time)
Different possible status codes returned: https://www.zerobounce.net/docs/email-list-validation/status-codes/
Saving validation status in DB together with email (in `home_owners` or `vendors` table:
- `VALID` - ZeroBounce returned `Valid` status code treated as valid.
- `INVALID` - any other status returned by ZeroBounce
- `NOT_VERIFIED` - when `ZERO_BOUNCE_API_KEY` env variable is missing or ZeroBounce returned an error, for example: `{ error: 'Invalid API key or your account ran out of credits' }`

when ZeroBounce returns error it will be forwarded to Sentry as an error while FE will receive `NOT_VERIFIED` status

Email is verified in:
- homeowner flow - non-blocking warning displayed when `INVALID`
- vendor flow - non-blocking warning displayed when `INVALID`