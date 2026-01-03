---
title: Get OpenSky Database Statistics
excerpt: >-
  Get statistics about the local OpenSky aircraft database.


  The OpenSky database provides offline aircraft information lookups,

  reducing external API calls and improving response times.


  **To populate the database:**

  1. Download from:
  https://s3.opensky-network.org/data-samples/metadata/aircraft-database-complete-2025-08.csv

  2. Place in `/data/opensky/aircraft-database.csv`

  3. Restart the API


  Returns:

  - **loaded**: Whether the database is loaded

  - **total_aircraft**: Number of aircraft in the database

  - **db_path**: Path to the database file
api:
  file: openapi (1).json
  operationId: get_opensky_db_status_api_v1_opensky_stats_get
hidden: false
---