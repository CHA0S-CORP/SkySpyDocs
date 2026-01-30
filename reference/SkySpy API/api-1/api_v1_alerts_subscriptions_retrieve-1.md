---
title: /api/v1/alerts/subscriptions/{rule_id}/
excerpt: >-
  ViewSet for alert rule subscriptions.


  Routes:

  - GET /api/v1/alerts/subscriptions/ - List user's subscriptions

  - POST /api/v1/alerts/subscriptions/ - Subscribe to a rule (with rule_id in
  body)

  - DELETE /api/v1/alerts/subscriptions/{rule_id}/ - Unsubscribe from a rule
api:
  file: skyspy-api.yaml
  operationId: api_v1_alerts_subscriptions_retrieve
hidden: false
---