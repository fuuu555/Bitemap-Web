# Explore Distance Query Fix

## Problem

The explore endpoint accepts `distance_km` as a URL query parameter. Query parameters arrive as strings, but the current numeric `Literal[2, 5, 10]` validation rejects values such as `distance_km=10`, returning HTTP 422 before the distance filter runs.

## Design

Parse `distance_km` as an integer at the endpoint boundary, then validate that it is one of the supported values: 2, 5, or 10. Keep the existing query name, frontend options, and distance filtering behavior unchanged. Unsupported or non-numeric values should continue to return HTTP 422.

## Scope and verification

Change only the explore route's query validation and the smallest supporting type declaration if needed. Verify the route accepts `distance_km=10` and rejects unsupported values, without changing filtering or ranking behavior.
