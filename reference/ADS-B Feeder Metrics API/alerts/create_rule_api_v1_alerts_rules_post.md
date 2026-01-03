---
title: Create Alert Rule
excerpt: |-
  Create a new alert rule.

  **Simple rules** use type/operator/value:
  ```json
  {
      "name": "Low Altitude",
      "type": "altitude",
      "operator": "lt",
      "value": "3000",
      "priority": "warning"
  }
  ```

  **Complex rules** use conditions with AND/OR logic:
  ```json
  {
      "name": "Military Low Approach",
      "conditions": {
          "logic": "AND",
          "groups": [
              {"logic": "AND", "conditions": [
                  {"type": "military", "operator": "eq", "value": "true"},
                  {"type": "altitude", "operator": "lt", "value": "5000"}
              ]}
          ]
      },
      "priority": "critical"
  }
  ```

  Operators: eq, ne, lt, gt, le, ge, contains, startswith
  Priorities: info, warning, critical
api:
  file: openapi (1).json
  operationId: create_rule_api_v1_alerts_rules_post
hidden: false
---