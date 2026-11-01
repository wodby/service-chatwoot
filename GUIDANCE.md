# Chatwoot on Wodby

What Wodby sets up for Chatwoot on this service. It runs the official `chatwoot/chatwoot` image: Rails on port 3000 in the main workload, and Sidekiq in the required derivative service `chatwoot-sidekiq`.

## First administrator

- The action `create_admin` (`post_deploy`) runs after every deployment. The rollout has finished by then, so the web container has already prepared the database. Its output is in the deployment task's log and never includes the password.
- When no super admin exists, it creates one with Chatwoot's own `AccountBuilder`, the code behind the onboarding page. The user is confirmed, so no mail is needed. The sign-in email is the required setting **Admin email** (`CHATWOOT_ADMIN_EMAIL`), the password is the generated token `admin_password` (`CHATWOOT_ADMIN_PASSWORD`), and the first Chatwoot account is named `Chatwoot`. Rename it under **Settings > Account Settings**.
- In every case except an empty email or password, it then deletes the Redis key `CHATWOOT_INSTALLATION_ONBOARDING`, so `/installation/onboarding` redirects away. It does not call `ChatwootHub.register_instance`, so nothing is sent to Chatwoot's hub.
- When a super admin already exists, it creates nothing and changes no user. Changing **Admin email** or the token later does not change an existing administrator.
- **Admin email** defaults to the placeholder `admin@example.com`, which works for sign-in but cannot receive mail. A required setting with no value would stop the deployment that follows a stack upgrade, so a default is needed. Change the address of an existing administrator in the Super Admin console at `/super_admin`, where editing a user's email applies at once without a confirmation message.
- An environment upgraded before anyone finished onboarding gets its administrator from this action, with the placeholder address unless **Admin email** was changed first.
- When **Admin email** already belongs to a user who is not a super admin, the action logs a warning, creates nobody and closes the onboarding page. Set another address and deploy again.
- When **Admin email** or the token is empty, the action logs a warning, creates nobody and leaves the onboarding page as it is.
- An invalid email address fails the action and the deployment, with Chatwoot's message in the log. Nothing is created, and the onboarding page stays open until the address is fixed and the service is deployed again.
- The password follows Chatwoot's rules: at least 6 characters, with an upper-case letter, a lower-case letter, a number and a special character. The token is 32 characters and always contains all four.
- The page does not depend on the **Account signup** setting (`ENABLE_ACCOUNT_SIGNUP`), which is `false` by default. Do not turn signup on to get a first user.

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

The token `admin_password` is the first administrator's password, described above. Chatwoot itself does not read it.

`FRONTEND_URL` follows the service's primary host.

## Database

The web container runs `rails db:chatwoot_prepare` before it starts the server. It loads the schema into a new database and applies migrations on later deployments. Do not add a separate migration step.

## Attachments

Attachments are on the shared `storage` volume at `/app/storage` (`ACTIVE_STORAGE_SERVICE=local`). For object storage, attach a variable integration with the variables Chatwoot requires for that provider, and override `ACTIVE_STORAGE_SERVICE`.

## Changing configuration

Change the service's settings or add environment variables, then deploy. Variables that the settings do not cover go in the integration "Additional Chatwoot environment variables".

## Check the result

- `GET /health` on port 3000 is the startup and liveness check, and `GET /api` is the readiness check. Send `X-Forwarded-Proto: https`, because **Force SSL** redirects plain HTTP.
- `/installation/onboarding` redirects to `/`, and `/` opens the sign-in page. `POST /auth/sign_in` with **Admin email** and `admin_password` answers `200`. Between the rollout and the end of the action, the onboarding page is still open.
