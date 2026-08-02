---
name: Provision WisdomAI users and embed analytics
description: >-
  Create and attribute users via the GraphQL API, mint short-lived embedded
  user tokens with impersonateUser, and run scheduled analytics agents —
  the operator/embedding flow for WisdomAI.
api: https://docs.wisdom.ai/integrations/graphql-api/GraphQL-API
generated: '2026-07-21'
method: generated
source: https://docs.wisdom.ai/integrations/graphql-api/mutations/auth/impersonate-user
operations:
- createUsers
- setUserAttributes
- listUsers
- impersonateUser
- triggerSchedule
---

# Provision WisdomAI users and embed analytics

Authenticate with a Bearer JWT (exchange an API key via `exchangeAccessToken`).

## Steps

1. **Create users** with the `createUsers` mutation; audit the roster with the
   `listUsers` query. Accounts managed by SCIM show `isScimManaged: true` —
   prefer your IdP (SCIM/JIT) for those and use the API only for the rest.
2. **Set user attributes** with `setUserAttributes`. Attributes drive
   row-level security bindings, so set them before the user's first query.
3. **Mint an embedded-user token**: for embedding scenarios call the
   `impersonateUser` mutation with your Descope access key to obtain a
   short-lived JWT for the embedded user. Pass that JWT as the `token` query
   parameter in the iframe URL — NOT in an Authorization header. The React
   embedding SDK (@wisdomai/react, see components/wisdom-ai-components.yml)
   wraps this flow.
4. **Trigger an analytics agent** on demand with the `triggerSchedule`
   mutation (`mutation TriggerAgent($scheduleId: String!) { success: triggerSchedule(id: $scheduleId) }`).
   You need view access to the agent.

## Rules

- HTTP status is always 200 — inspect the `errors` array / `ResponseStatus`
  body instead (errors/wisdom-ai-problem-types.yml).
- `impersonateUser` tokens are short-lived by design; mint per session, never
  store.
- Row-level and column-level security follow the impersonated user, so test
  embedded views with a real least-privilege user, not an admin key.
