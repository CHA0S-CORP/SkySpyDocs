---
title: Get Aircraft Information
excerpt: |-
  Get detailed information about an aircraft by its ICAO hex address.

  Returns comprehensive airframe data including:
  - **Registration**: Aircraft registration number (e.g., N12345)
  - **Type**: Aircraft type code and full name (e.g., B738, Boeing 737-800)
  - **Manufacturer**: Aircraft manufacturer (e.g., Boeing, Airbus)
  - **Age**: Year built and calculated age in years
  - **Operator**: Current operator/airline
  - **Owner**: Registered owner
  - **Country**: Country of registration
  - **Photo**: Aircraft photo URL if available

  Data sources:
  - hexdb.io for registration data
  - OpenSky Network for additional metadata
  - Planespotters.net for aircraft photos

  **Caching**: Data is cached for 7 days. Failed lookups retry after 24 hours.
  Use `refresh=true` to force a fresh lookup from external sources.
api:
  file: openapi (1).json
  operationId: get_airframe_info_api_v1_aircraft__icao_hex__info_get
hidden: false
---