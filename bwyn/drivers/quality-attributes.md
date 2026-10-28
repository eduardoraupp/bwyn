# Quality Attributes

How well the system must perform: speed, security, reliability and ease of change.

Every user-facing page and every API response must load within 2 seconds under [normal load, e.g. X concurrent users]. This covers all user-facing endpoints including film listings, seat selection, and checkout steps.

All endpoints are secured using OAuth 2.0 authorisation code flow. Unauthenticated or unauthorised requests must be rejected before any business logic runs.
