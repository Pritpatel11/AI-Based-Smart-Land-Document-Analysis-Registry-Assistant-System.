# AI-Based Smart Land Document Analysis & Registry Assistant System
## Comprehensive Test Case Documentation (Part 2/2)

---

### Module 6: Financial Tools (Loan & Estimator)
Logic for calculating stamp duty, registration fees, and predicting loan eligibility based on land value.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_FIN_01 | Estimator | Calculate Stamp Duty | Correct fees displayed for state. | Fees calculated accurately. | PASS | Solved using custom JS tax logic with state-specific rate mapping. |
| TC_FIN_02 | Estimator | Area Unit Conversion | Convert Square Feet to Acres. | Correct conversion factor. | PASS | Implemented `conversionUtil.js` for unit handling. |
| TC_FIN_03 | Loan | Credit Score vs Loan Limit | Higher score gets higher limit. | Limit adjusted dynamically. | PASS | Applied linear regression formula for limit estimation. |
| TC_FIN_04 | Loan | Property Value impact | Loan should not exceed 80% LTV. | Loan capped at 80%. | PASS | Added validation check `if(request > 0.8 * landValue)`. |
| TC_FIN_05 | Estimator | Zero Value Input | Handle empty price/area fields. | Show "Enter valid values". | PASS | Added Frontend field validation and prevented `NaN`. |
| TC_FIN_06 | Loan | Multiple Active Loans Check | Flag users with high debt ratio. | Flagged "High Debt". | PASS | Mocked external Credit Bureau API response for testing. |
| TC_FIN_07 | Estimator | Discount for Female Owners | Apply 1-2% discount where applicable. | Discount applied in total. | PASS | Logic checks `owner.gender` before finalizing tax. |
| TC_FIN_08 | Estimator | Update Govt Tax Rates | Admin should be able to update rates. | Global rates updated. | PASS | Backend `Settings` endpoint updates rate constants in DB. |
| TC_FIN_09 | Estimator | Joint Ownership Calculation | Split tax between 2+ owners. | Tax was calculated for only one. | FAIL | Solved by adding a loop for active owners in `TaxEstimator` service. |
| TC_FIN_10 | Loan | Self-Employed User Logic | Check eligibility for non-salaried. | All self-employed were rejected. | FAIL | Solved by adding an "ITR Upload" and alternate income verification path. |

---

### Module 7: Property Inquiry & History
Retrieval of past ownership records, dispute status, and land geolocation.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_HIS_01 | History | Navigate to Session History | Lists past analyzed documents. | Past sessions retrieved from DB. | PASS | Solved using MongoDB indexed queries on `userId`. |
| TC_HIS_02 | History | Search Property by Survey No. | Return ownership history. | 5-year history displayed. | PASS | Linked `Document` model with `PropertyHistory` archive. |
| TC_HIS_03 | History | Land Dispute Check (UI) | Show RED icon if land is in court. | Icon displayed on Map. | PASS | Checked against govt "Disputed Property" dataset. |
| TC_HIS_04 | History | Map Geolocation Accuracy | Pin land location on Google Maps. | Pin shows within 5m radius. | PASS | Used Google Maps Geocoding API with Survey Co-ordinates. |
| TC_HIS_05 | History | Export History to PDF | User can save history report. | PDF generated with timestamps. | PASS | Used `html2canvas` and `jspdf` on the frontend. |
| TC_HIS_06 | History | Load History for New User | Show "No history found" state. | Empty state handled gracefully. | PASS | Added conditional rendering in `PropertyHistory.jsx`. |
| TC_HIS_07 | History | Map Zoom on Rural Lands | Auto-zoom to show plot boundaries. | Manual zoom was required. | FAIL | Solved by calculating `bounding-box` from GPS points in Map component. |
| TC_HIS_08 | History | Search non-existent Survey No. | Show "Not found" warning. | System hung for 30s searching. | FAIL | Solved by adding a TTL and early-exit for empty DB queries. |

---

### Module 8: Notification & Task Management
Real-time messaging system for analysis completion and administrative tasks.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_NOT_01 | Notif | Real-time Analysis Update | Notify user when AI finishes OCR. | Notification popup appeared. | PASS | Implemented using Socket.io `emit` from backend. |
| TC_NOT_02 | Notif | Mark All as Read | Clear notification badge count. | Badge cleared instantly. | PASS | Backend `PUT /notifications/read-all` updates status. |
| TC_NOT_03 | Notif | Push Notification (Offline) | Send email if user is offline. | Email received in Inbox. | PASS | Integrated BullMQ for background job processing. |
| TC_NOT_04 | Task | Assign Verification to Staff | Staff sees task in dashboard. | Task listed in "Pending". | PASS | `Task` model assigns `staffId` and tracks status. |
| TC_NOT_05 | Notif | Frequency Control | User can mute non-critical alerts. | Only High-Risk alerts shown. | PASS | Added `notification_settings` schema to User model. |
| TC_NOT_06 | Task | Deadline Reminder | Task turns ORANGE if > 24h old. | UI updated color code. | PASS | Frontend logic calculates `now() - createdAt`. |
| TC_NOT_07 | Notif | Double notification push | Only one alert per event. | User received 2-3 same alerts. | FAIL | Solved by adding a `deduplication_key` in the task queue. |
| TC_NOT_08 | Notif | Mobile Push Permission | Ask for notification permission. | System tried to push without ask. | FAIL | Solved by adding `Notification.requestPermission()` check in Frontend. |

---

### Module 9: Intelligent Chat Assistant
Testing the context-aware chatbot that answers queries about uploaded documents.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_CHT_01 | Chat | Query about Owner Name | AI should extract from active doc. | "The owner is Prit Patel". | PASS | Injected document metadata into Gemini System Prompt. |
| TC_CHT_02 | Chat | Session Persistence | Chat history holds after refresh. | History reloaded from DB. | PASS | Stored chat messages in `ChatSession` MongoDB model. |
| TC_CHT_03 | Chat | Complex Legal Question | AI explains legal terminology. | Explain "Encumbrance Certificate". | PASS | Used Gemini Pro with legal-context fine-tuning. |
| TC_CHT_04 | Chat | Bot response latency | Reply within 3 seconds. | Intermittent 15s delays. | FAIL | Optimized by using `streaming` response mode (Server Sent Events). |
| TC_CHT_05 | Chat | Clear Conversation | Remove all messages from UI/DB. | Chat window reset to blank. | PASS | `DELETE /chat/:sessionId` endpoint handles cleanup. |
| TC_CHT_06 | Chat | Document Context Switching | Switch chat context to 2nd doc. | AI answers for 2nd document. | PASS | Updated context state on document selection sidebar. |
| TC_CHT_07 | Chat | Ask about deleted document | Should say "Document no longer exists". | AI hallucinated owner's name. | FAIL | Solved by validating `documentId` status before each chat query. |
| TC_CHT_08 | Chat | Multi-turn Conversation | Remember previous question's context. | Bot forgot owner's address. | FAIL | Solved by passing the last 5 messages in Gemini's `history` parameter. |

---

### Module 10: Performance & Security
System-wide metrics including load times, security headers, and protection against attacks.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_SEC_01 | Security | Access Dashboard without Login | Access should be denied. | Redirected to Login page. | PASS | Solved using PrivateRoute (Protected Routes) in React. |
| TC_SEC_02 | Perform | First Contentful Paint (FCP) | Page loads in under 2 seconds. | Home page loading in 4s. | FAIL | Solved by implementing Image Lazy Loading and Code Splitting (React.lazy). |
| TC_SEC_03 | Security | XSS Protection | Reject script tags in Input fields. | Input sanitized/escaped. | PASS | Used `dompurify` on frontend and `helmet` on backend. |
| TC_SEC_04 | Security | CORS Restriction | Reject requests from unknown origin. | Unauthorized domain blocked. | PASS | Configured CORS whitelist in `server.js`. |
| TC_SEC_05 | Security | Env Variable Leakage | ensure `.env` is not public. | Secret keys not in browser console. | PASS | Verified `.gitignore` hides `.env` and used `VITE_` prefix carefully. |
| TC_SEC_06 | Perform | Concurrent API Requests | Handle 50 concurrent searches. | System remained stable. | PASS | Configured PM2 cluster mode for Node.js backend. |
| TC_SEC_07 | Security | CSRF Protection | Prevent cross-site request forgery. | Malicious request rejected. | PASS | Implemented `csurf` middleware for session-based routes. |
| TC_SEC_08 | UI/UX | Test on Mobile Screen (360px) | Dashboard should be responsive. | Buttons were overlapping. | FAIL | Solved by adding Tailwind md: and lg: responsive grid breakpoints. |
| TC_SEC_09 | Perform | Lighthouse SEO Score | Score should be > 90. | Score was around 75. | FAIL | Solved by adding meta descriptions, alt tags, and semantic HTML5 tags. |
| TC_SEC_10 | Security | SQL/NoSQL Injection | Reject `$gt: ""` in JSON body. | Input validated via Joi/Mongoose. | PASS | Used parameterized queries/Mongoose schemas to block injection. |
| TC_SEC_11 | Security | Direct API URL access | Block `/api/admin` skip frontend. | Admin data exposed to anyone. | FAIL | Solved by adding `verifyAdmin` middleware to all sensitive routes. |
| TC_SEC_12 | Security | Brute Force on OTP | Block user after 10 failed OTPs. | Infinite retries allowed. | FAIL | Solved by adding Redis-based tracking for OTP attempts. |
