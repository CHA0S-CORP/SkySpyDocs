---
title: Get all flight pattern and geographic statistics
excerpt: |2-

          Returns combined flight pattern and geographic statistics in a single response.

          Useful when you need both stat types and want to minimize API calls.
          Includes:
          - Flight patterns (routes, busiest hours, duration by type, aircraft types)
          - Geographic stats (countries, operators, airports, military breakdown)
          
api:
  file: skyspy-api.yaml
  operationId: api_v1_stats_combined_retrieve
hidden: false
---