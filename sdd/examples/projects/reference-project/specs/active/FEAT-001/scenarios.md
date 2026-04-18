# Scenarios — FEAT-001: Send Notification

> Derived from `requirements.md` EARS clauses.  
> Each scenario maps to at least one test in `TEST_FIRST_GATE.md`.

---

## Happy Path Scenarios

### SC-01: Successful email notification
**Ref:** EARS-01, EARS-02, EARS-03

```
Given a caller with a valid API key
  And a valid request body with channel "email", recipient_ref "usr_abc123",
      template_slug "welcome_email", variables {"first_name": "Alex"}
  And template "welcome_email" exists for channel "email"
  And User Service resolves "usr_abc123" to "alex@example.com"
When POST /notifications is called
Then the response status is 202 Accepted
  And the response body contains a notification_id and status "queued"
  And a Notification record exists in the DB with status "queued"
  And delivery is attempted within 5 seconds
  And on delivery success, status is updated to "delivered"
  And a DeliveryAttempt row is created with outcome "sent"
```

### SC-02: Get notification status
**Ref:** EARS-04

```
Given a caller with a valid API key
  And a notification with id "ntf_7f3a9b" exists with status "delivered"
  And one DeliveryAttempt exists for this notification
When GET /notifications/ntf_7f3a9b is called
Then the response status is 200 OK
  And the response body contains status "delivered" and the delivery_attempts list
```

---

## Retry Scenarios

### SC-03: Retry after failed delivery
**Ref:** EARS-05, EARS-06

```
Given a notification has been queued
  And the first delivery attempt fails with provider error
When the retry schedule fires
Then a second DeliveryAttempt is recorded with outcome "failed"
  And a third attempt is scheduled after 120 seconds
  And notification status remains "queued"
```

### SC-04: Notification exhausts all retries
**Ref:** EARS-07

```
Given a notification has made 3 failed delivery attempts
When the system evaluates retry eligibility
Then no further attempts are made
  And notification status is set to "failed"
```

---

## Error Scenarios

### SC-05: Missing API key
**Ref:** EARS-08

```
Given no X-API-Key header is present
When POST /notifications is called
Then the response status is 401 Unauthorized
  And the response body is {"error": "invalid_api_key"}
  And no notification record is created
```

### SC-06: Invalid request body
**Ref:** EARS-09

```
Given a valid API key
  And a request body missing required field "template_slug"
When POST /notifications is called
Then the response status is 422 Unprocessable Entity
  And the response body contains a validation error listing "template_slug"
```

### SC-07: Template not found
**Ref:** EARS-10

```
Given a valid API key and a valid request body
  And template slug "nonexistent_template" does not exist for channel "email"
When POST /notifications is called
Then the response status is 422 Unprocessable Entity
  And the response body is {"error": "template_not_found", "slug": "nonexistent_template"}
```

### SC-08: User Service unavailable
**Ref:** EARS-11

```
Given a valid API key and a valid request body
  And the User Service returns a 503 during recipient resolution
When POST /notifications is called
Then the response status is 503 Service Unavailable
  And the response body is {"error": "recipient_resolution_failed"}
  And no notification record is created
```

### SC-09: Notification not found
**Ref:** EARS-12

```
Given a valid API key
  And no notification with id "ntf_unknown" exists
When GET /notifications/ntf_unknown is called
Then the response status is 404 Not Found
  And the response body is {"error": "notification_not_found"}
```

---

## Security Scenarios

### SC-10: No PII in log output
**Ref:** Constitution § Security Gate

```
Given a notification is sent and delivered successfully
When the application logs are inspected
Then no email address, phone number, or resolved recipient address appears in any log line
```

### SC-11: No stack trace in error response
**Ref:** Constitution § Security Gate

```
Given an unexpected exception occurs during notification processing
When the API returns a 500 response
Then the response body is {"error": "internal_error"}
  And no exception message, stack frame, or file path is included
```
