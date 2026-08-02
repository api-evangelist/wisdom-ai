---
name: Build and refresh a WisdomAI dashboard
description: >-
  List existing dashboards, create a new dashboard, add widgets backed by
  natural-language queries, set global filters, and trigger a data refresh —
  all through the WisdomAI GraphQL API.
api: https://docs.wisdom.ai/integrations/graphql-api/GraphQL-API
generated: '2026-07-21'
method: generated
source: https://docs.wisdom.ai/integrations/graphql-api/mutations/dashboard/create-dashboard
operations:
- dashboards
- createDashboard
- addWidgetToDashboard
- updateDashboardFilters
- refreshDashboard
---

# Build and refresh a WisdomAI dashboard

Authenticate first (see the "Ask WisdomAI a data question" skill — exchange an
API key for a JWT via `exchangeAccessToken` and send it as a Bearer token).

## Steps

1. **Inventory what exists** with the `dashboards` query
   (`query GetDashboards($scope: DashboardScope!) { dashboards(scope: $scope) { nodes { id name createdAt domains { id name } } } }`).
   Scope controls whose dashboards you see; results honor role assignments.
2. **Create the dashboard** with the `createDashboard` mutation. A Dashboard
   binds `domains[]`, `widgets[]`, `filters[]`, and `roleAssignments[]`
   (see data-model/wisdom-ai-data-model.yml).
3. **Add widgets** with `addWidgetToDashboard`. Each `DashboardWidget` carries
   a `title`, an `nlQuery` (the natural-language question that powers it), a
   `visualizationType`, and a `layout`. Widgets promoted from chat answers
   keep `conversationId`/`messageId` provenance.
4. **Set global filters** with `updateDashboardFilters`
   (`DashboardFilterDefinition[]` applied across the dashboard).
5. **Refresh the data** with `refreshDashboard`; check widget
   `dataRefreshedAt` timestamps to confirm.

## Rules

- HTTP status is always 200 — check the top-level `errors` array and
  mutation `ResponseStatus` (`code`, `message`) instead
  (errors/wisdom-ai-problem-types.yml).
- Sharing is explicit: use scope-role-assignment mutations
  (`addScopeRoleAssignmentsForSharing` page in the docs) rather than mutating
  `roleAssignments` ad hoc.
- Dashboard results respect row-level / column-level security per viewer.
