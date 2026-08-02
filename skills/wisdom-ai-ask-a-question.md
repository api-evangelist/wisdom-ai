---
name: Ask WisdomAI a data question
description: >-
  Authenticate against a WisdomAI tenant, open a conversation in a domain, ask
  a natural-language business-intelligence question, and stream the structured
  answer (tables, SQL, visualizations) over a GraphQL WebSocket subscription.
api: https://docs.wisdom.ai/integrations/graphql-api/GraphQL-API
generated: '2026-07-21'
method: generated
source: https://docs.wisdom.ai/integrations/graphql-api/GraphQL-API
operations:
- exchangeAccessToken
- createConversation
- sendUserMessage
- subscribeConversation
---

# Ask WisdomAI a data question

All operations go to a single tenant-scoped GraphQL endpoint:
`https://{ACCOUNT}.askwisdom.ai/graphql` (WebSocket: `wss://{ACCOUNT}.askwisdom.ai/graphql`).
Both `askwisdom.ai` and `wisdom.ai` tenant domains are valid depending on deployment.

## Steps

1. **Exchange your API access key for a JWT.** API keys are created per-user in
   Settings > API Keys. Run the `exchangeAccessToken` query:
   `query ExchangeAccessToken($accessToken: String!) { exchangeAccessToken(accessToken: $accessToken) }`.
   Send the returned session token on every request as `Authorization: Bearer <jwt>`.
2. **Create a conversation** in the target domain with the `createConversation`
   mutation (`domainId: String!`, `hidden: Boolean!`). It returns the conversation id.
3. **Subscribe first, then ask.** Open the WebSocket and start
   `subscribeConversation` for the conversation so you receive streamed
   `ConversationUpdateOneOf` updates.
4. **Send the question** with the `sendUserMessage` mutation
   (`conversationId`, `domainId`, `query: DeltaInput!` — the question text is a
   Delta document). Optional flags include `enableDeepAnalysis`,
   `disableClarifications`, `chatThinkingEffort`, and `toolSelection`.
5. **Read answers from the subscription stream** — messages arrive as
   `ChatMessage` objects whose `body` is a Delta; visualizations and SQL arrive
   as structured artifacts.

## Rules

- **Never trust the HTTP status.** The API returns HTTP 200 even on failure.
  Inspect the body: request-level failures land in the top-level `errors`
  array; mutation-level failures come back as a `ResponseStatus` object
  (`code`, `message`). See errors/wisdom-ai-problem-types.yml.
- Answers respect the caller's row-level and column-level security; different
  users can legitimately get different results for the same question.
- JWTs are session tokens — re-run `exchangeAccessToken` when they expire
  rather than caching them long-term.
