# Chatwoot service for Kubernetes on Wodby

Run Chatwoot as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Chatwoot.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Chatwoot application stack](https://github.com/wodby/stack-chatwoot)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `chatwoot` |
| Type | Application service |
| Versions | `4.17` by default |
| Workloads | `main` (Deployment, primary) |
| Containers | `chatwoot` using `chatwoot/chatwoot` |
| Endpoints | `web`: HTTP 3000 (main) |
| Service links | PostgreSQL with pgvector (`db`), required; Redis (`redis`), required; Shared attachment storage (`storage`), required; Mail transfer agent (`sendmail`), optional |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateless`; version `0.2.1` |
| Configuration and operations | 7 settings, 1 integration slots, 1 volumes, 1 actions |

## Use this service

Use this service through [Chatwoot application stack](https://github.com/wodby/stack-chatwoot), or reference `chatwoot` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Runtime contract

Chatwoot runs separate Rails web and Sidekiq processes. PostgreSQL with
pgvector, Redis 7 or newer, and shared attachment storage are required. SMTP is
optional but recommended for password resets and notifications.

Wodby runs `rails db:chatwoot_prepare` after each web deployment. It generates
the application secret and the three Active Record encryption keys when an app
is created.

## Object storage

The default profile stores attachments on the shared `/app/storage` volume.
For S3, S3-compatible storage, Google Cloud Storage, or Azure Storage, attach a
variable integration with the environment variables required by Chatwoot and
override `ACTIVE_STORAGE_SERVICE`.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
