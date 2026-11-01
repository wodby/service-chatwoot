# Chatwoot on Wodby

What Wodby sets up for Chatwoot on this service. It runs the official `chatwoot/chatwoot` image: Rails on port 3000 in the main workload, and Sidekiq in the required derivative service `chatwoot-sidekiq`.

## First administrator

- A new app has no user. Wodby does not create one and generates no administrator password.
- After the first deployment, the main domain redirects to `/installation/onboarding`. The form asks for a name, company name, work email and password. **Finish Setup** creates the first Chatwoot account and a confirmed super admin user, so no mail is needed. The page then redirects to the sign-in page.
- The page is open to anyone who can reach the address until it is submitted, so finish it right after the first deployment.
- The password needs at least 6 characters, with an upper-case letter, a lower-case letter, a number and a special character.
- The subscription checkbox is on by default. When it stays on, Chatwoot sends the company name, name and email to its hub for release notes and newsletters.
- The page does not depend on the **Account signup** setting (`ENABLE_ACCOUNT_SIGNUP`), which is `false` by default. Do not turn signup on to get a first user.
- Chatwoot marks onboarding as pending with a Redis key when the database is first seeded, and deletes the key after the form is submitted.
- The Super Admin console for that user is at `/super_admin`.

## Linked services

| Link | Required | Variables |
| --- | --- | --- |
| `db`: PostgreSQL with pgvector | yes | `DATABASE_URL` |
| `redis`: Redis | yes | `REDIS_URL` |
| `storage`: shared attachment storage | yes | none; provides the shared volume |
| `sendmail`: mail transfer agent | no | `SMTP_ADDRESS`, `SMTP_PORT`, `SMTP_ENABLE_STARTTLS_AUTO` |

Do not copy the values of `DATABASE_URL` and `REDIS_URL` into a repository: both contain a password.

## Generated values

Wodby generates the tokens `secret_key_base`, `ar_primary_key`, `ar_deterministic_key` and `ar_key_derivation_salt`. They reach the container as `SECRET_KEY_BASE` and the three `ACTIVE_RECORD_ENCRYPTION_*` variables. Keep them stable, because Chatwoot encrypts stored data with them.

`FRONTEND_URL` follows the service's primary host.

## Database

The web container runs `rails db:chatwoot_prepare` before it starts the server. It loads the schema into a new database and applies migrations on later deployments. Do not add a separate migration step.

## Attachments

Attachments are on the shared `storage` volume at `/app/storage` (`ACTIVE_STORAGE_SERVICE=local`). For object storage, attach a variable integration with the variables Chatwoot requires for that provider, and override `ACTIVE_STORAGE_SERVICE`.

## Changing configuration

Change the service's settings or add environment variables, then deploy. Variables that the settings do not cover go in the integration "Additional Chatwoot environment variables".

## Check the result

- `GET /health` on port 3000 is the startup and liveness check, and `GET /api` is the readiness check. Send `X-Forwarded-Proto: https`, because **Force SSL** redirects plain HTTP.
- A new app redirects `/` to `/installation/onboarding`. After onboarding it redirects to the sign-in page.
