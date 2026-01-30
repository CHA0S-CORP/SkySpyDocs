---
title: /api/v1/admin/users/{id}/
excerpt: |-
  ViewSet for managing users.

  Requires `users.*` permissions.
  Uses Django User ID for lookups but returns SkyspyUser profile data.
api:
  file: skyspy-api.yaml
  operationId: api_v1_admin_users_partial_update
hidden: false
---