# QR-Code Food Safety Information System – Test Plan

## 1. Purpose

This test plan defines the expected quality, reliability, and performance standards for the QR-Code Food Safety Information System. The system allows consumers, food suppliers, retailers, and regulators to scan a QR code on a food product and retrieve structured safety, recall, origin, and supply-chain information. This document is intended to guide implementation and serve as a living quality-control reference for ongoing maintenance.

The main goal is to ensure that users can access accurate, current, portable, and understandable food safety information through a stable API and frontend interface.

---

## 2. System Overview

The information system has three major parts:

- **Frontend access point:** A mobile-friendly web or app interface opened by scanning a QR code on a food product.  
- **Backend API:** An API that receives the product or batch identifier and returns a structured safety record.  
- **Data sources and structure:** Standardized JSON records based on product identity, batch information, supplier data, inspection results, recall status, and consumer-facing safety information.

The system is designed to support portability by using structured formats such as JSON and CSV. It should allow authorized suppliers, regulators, and researchers to transfer or reuse relevant records across systems while protecting sensitive internal audit data.

---

## 3. Quality Objectives

The system should meet the following quality goals:

1. **Accuracy:** Product identity, recall status, inspection result, and safety warning fields should match the source records.  
2. **Availability:** Consumers should be able to access the system when scanning a valid QR code.  
3. **Performance:** Most user requests should return results quickly enough to support real-time decision-making while shopping or inspecting inventory.  
4. **Portability:** Records should be available in consistent JSON format, with optional CSV export for appropriate users.  
5. **Security and privacy:** Public users should only see consumer-safe information, while supplier or regulator-only fields should require authentication.  
6. **Traceability:** Errors, failed lookups, data updates, and recall checks should be logged for later review.

---

## 4. Functional Testing

| Test Case | Description | Method | Expected Result | Frequency |  
|---|---|---|---|---|  
| Valid QR Code Scan | Scan a QR code linked to an existing product batch. | Manual frontend test and API request. | The system returns HTTP 200 and displays the correct product safety record. | Before each release |  
| Invalid QR Code | Scan a QR code with a malformed or unknown batch ID. | Manual and automated test. | The system returns a clear “record not found” message and does not crash. | Before each release |  
| Product Identity Lookup | Search by product ID, batch ID, or GS1-style identifier. | Automated API test. | Correct product name, supplier, batch number, and category are returned. | Each GitHub push |  
| Recall Status Check | Request a product with active, inactive, and unknown recall status. | Automated backend test using sample records. | Recall status, recall date, reason, and source are correctly displayed. | Daily and before release |  
| Consumer View Filter | Access public product record as a consumer. | Role-based test. | Sensitive fields such as internal audit logs, supplier notes, and backend update history are hidden. | Before each release |  
| Regulator/Supplier Access | Access protected update or audit fields with authentication. | Authenticated API test. | Authorized users can view or update allowed fields; unauthorized users are rejected. | Before each release |  
| Data Export | Export a product or recall record as JSON or CSV. | Manual and automated test. | Exported file contains stable field names, valid formatting, and no restricted fields for public users. | Weekly |  
| Missing Field Handling | Test records with missing optional fields such as inspection date or supplier contact. | Unit test. | The system returns a structured response with null or “not available,” not a broken page. | Each GitHub push |  
| Data Validation | Submit records with wrong date format, missing batch ID, or invalid recall value. | Unit/API test. | The system rejects invalid records with clear 400-level errors. | Each GitHub push |  
| Frontend Error Handling | Stop backend service and scan a valid QR code. | Manual test. | The frontend shows a user-friendly error message and does not expose technical stack traces. | Before release |

---

## 5. Data Quality Tests

| Data Quality Area | Test | Expected Standard | Action if Failed |  
|---|---|---|---|  
| Required Fields | Check that every record includes product ID, batch ID, product name, supplier, origin, and last update timestamp. | 100% of active records include required fields. | Block deployment or quarantine incomplete record. |  
| Recall Freshness | Compare \`last\_recall\_check\` against the update schedule. | Recall status checked at least once every 24 hours. | Trigger recall data refresh and notify maintainer. |  
| Schema Validity | Validate JSON against the project schema. | 100% of published records pass schema validation. | Reject update and log validation error. |  
| Source Traceability | Confirm each safety or recall claim has a source name and timestamp. | All safety claims have traceable source metadata. | Mark record as “source pending” and review manually. |  
| Duplicate Records | Check for duplicate product ID and batch ID combinations. | No duplicate active records for the same batch. | Merge or deactivate duplicate after review. |  
| Public Field Safety | Confirm public response excludes internal-only fields. | No backend-only fields appear in public API response. | Treat as security incident and patch immediately. |

---

## 6. Performance Testing

| Test Case | Description | Tool | Target | Frequency |  
|---|---|---|---|---|  
| Normal Load | Simulate regular users scanning QR codes. | Postman, Newman, or pytest API tests. | Average response time under 2 seconds. | Weekly |  
| Peak Load | Simulate increased traffic during a recall event. | Locust or Artillery. | API remains stable with no major error spike. | Before major release |  
| Batch Lookup | Request multiple batch records in a short period. | Scripted API test. | Responses remain under 3 seconds for normal batch size. | Monthly |  
| Cold Start | Load backend after idle time. | Browser, Postman, or automated script. | First response under 5 seconds if hosted on a sleep-based platform. | Weekly |  
| Large Record Response | Request product records with long supply-chain histories. | Automated test. | Response remains valid and readable; frontend does not freeze. | Before release |  
| Rate Limit Test | Send unusually high request volume from one client. | Scripted load test. | System throttles or blocks excessive requests without crashing. | Monthly |

---

## 7. Alarms, Monitoring, and Actions

| Alarm | Trigger | Monitoring Tool | Action |  
|---|---|---|---|  
| API Down | No successful response from backend for 1 minute. | UptimeRobot, Better Stack, or Azure Application Insights. | Notify maintainer; check hosting status and restart service if needed. |  
| High Latency | Average response time above 3 seconds for 5 minutes. | Application monitoring dashboard. | Review logs, check database/API dependency, and scale service if needed. |  
| 500 Error Spike | More than 3 server errors in 1 minute. | Backend logs and monitoring alert. | Create incident note, identify failing endpoint, and roll back recent changes if necessary. |  
| Recall Data Stale | \`last\_recall\_check\` older than 24 hours. | Scheduled data quality script. | Run recall refresh job and notify data owner. |  
| Schema Validation Failure | New or updated record fails JSON schema validation. | CI/CD validation test. | Reject record update and return validation details to maintainer. |  
| Unauthorized Access Attempt | Repeated failed access to supplier or regulator-only endpoints. | Auth logs. | Temporarily block suspicious client and review access logs. |  
| Broken QR Link | QR code resolves to missing or invalid product endpoint. | Weekly link checker. | Mark affected product batch for correction and notify supplier contact. |

---

## 8. Ongoing Implementation Plan

The test plan will be implemented through a mix of manual review, automated testing, scheduled monitoring, and release checks.

### Development Stage

During development, the team will create sample product records that include normal, missing, invalid, recalled, and unrecalled food batches. These records will be used to test both the API and frontend before real data is connected. Backend validation will check required fields, date formats, recall status values, and JSON schema consistency.

### Before Each Release

Before each release, the team will manually scan sample QR codes and confirm that the frontend displays the correct consumer-facing information. API tests will confirm that valid records return 200 responses, invalid requests return useful 400 or 404 responses, and protected endpoints reject unauthorized users. The team will also confirm that no restricted audit fields appear in public records.

### Continuous Testing

GitHub Actions or a similar CI/CD workflow should run unit tests and schema validation whenever code is pushed. Future versions should include automated frontend tests for the QR scan flow, product record display, error messages, and export behavior. Backend tests should be expanded using pytest or another automated testing framework.

### Scheduled Monitoring

A daily scheduled job should check recall data freshness, broken QR links, schema validity, and duplicate records. A weekly QA review should inspect a small sample of records manually to confirm that the information remains understandable to consumers and useful for regulators or suppliers.

---

## 9. Status Summary

| Area | Current Status | Next Step |  
|---|---|---|  
| Functional Tests | Planned and partially manual | Add automated API tests |  
| Data Quality Tests | Planned | Add JSON schema validation |  
| Performance Tests | Planned | Add Locust or Artillery load testing |  
| Alarms and Monitoring | Planned | Connect uptime and error monitoring |  
| CI/CD Testing | Planned | Add GitHub Actions workflow |  
| Documentation | In progress | Keep this test plan updated in repository |

---

## 10. Team Responsibilities

| Task | Owner |  
|---|---|  
| Frontend QR scan and display testing | Frontend developer |  
| Backend API and schema validation | Backend developer |  
| Recall freshness and data quality checks | Data maintainer |  
| Performance and load testing | QA or technical lead |  
| Monitoring setup and incident review | Project maintainer |  
| Weekly documentation update | Assigned team member |

---

## 11. Future Additions

Future versions of the test plan should include:

\- Automated pytest tests for every backend endpoint.  
\- Frontend tests for the QR scan flow and consumer display page.  
\- JSON schema validation for all published product records.  
\- Role-based access testing for consumer, supplier, and regulator users.  
\- Automated broken-link checks for QR code URLs.  
\- Error reporting connected to GitHub Issues or a similar tracking system.  
\- A dashboard showing uptime, average response time, recall freshness, and data validation status.

---

## 12. Conclusion

This test plan defines how the QR-Code Food Safety Information System will maintain trustworthy access to food safety information. Functional tests confirm that users receive the correct product and recall information. Performance tests confirm that the system remains usable during normal and high-demand situations. Alarms and monitoring create a process for detecting problems quickly, while ongoing validation protects the quality and portability of the information structure. As the project develops, this document should be updated whenever the system architecture, data structure, or user access model changes.  
