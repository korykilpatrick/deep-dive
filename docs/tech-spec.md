# Technical Specification: DeepDive Chrome Extension

## Overview

DeepDive is a Chrome extension that allows users to highlight text on any webpage and trigger automated deep-research queries via AI agents. These agents recursively query search engine APIs and large language models (LLMs) to gather information. The extension supports Google OAuth-based authentication, stores user queries and results (with an option for a Private Mode to skip storage), offers configurable summary lengths for results, and provides downloadable full reports of the research findings.

## System Architecture

The DeepDive system consists of a Chrome extension frontend, a backend service for processing queries, a database for storing data, and several external integrations for search and AI capabilities.

### Frontend (Chrome Extension UI)

- **Tech Stack:** Developed with TypeScript and React, styled using Tailwind CSS.
- **Key Components:**
  - **Popup UI:** Provides a minimalist interface to initiate deep research, manage API keys, toggle Private Mode, and select summary length preferences.
  - **Content Script:** Runs in the context of web pages to detect text highlights and display the DeepDive popup. It triggers research actions when the user clicks the "Run Deep Research" button.
  - **Background Script:** Acts as a bridge between the extension UI and the backend. It handles API call requests and manages user session data (e.g., storing authentication tokens).

### Backend (API & Processing Layer)

- **Implementation:** A traditional backend server is used (recommended: FastAPI in Python) rather than a serverless approach. This choice offers more control over long-lived connections and better suits the complex, multi-step query processing required. It also aligns with existing deep-research infrastructure.
- **Options Considered:**
  - *Serverless (Firebase Functions, AWS Lambda):* Provides lower maintenance overhead, automatic scaling, and a pay-per-use model.
  - *Traditional Server (Node.js/Express or Python/FastAPI):* Offers persistent connections and greater control, which is beneficial for complex and stateful operations.
  - *Recommendation:* **Use a FastAPI (Python) backend** for greater control and alignment with existing systems.
- **Core Responsibilities:**
  - Handle incoming research requests by querying external Search APIs and LLM APIs.
  - Manage user query data — storing queries and results in the database (skipping storage entirely if Private Mode is enabled for that request).
  - Serve stored query history to the frontend and provide endpoints for downloading full research reports.

### Database (PostgreSQL)

- **Schema Design:** A relational database (PostgreSQL) is used to store user data and query history. Key tables include:
  - **users:** Stores user information and credentials (e.g., `id`, `email`, OAuth tokens, encrypted API keys).
  - **queries:** Logs each research query made by a user (e.g., `id`, `user_id` reference, the `highlighted_text`, `query_results` summary or link to full report, `created_at` timestamp).
  - **settings:** Contains user-specific configurations (e.g., `id`, `user_id` reference, and preference fields such as summary length or Private Mode defaults).
- **Rationale:** A SQL database ensures structured storage and relational integrity between users and their queries. It allows for efficient indexing (e.g., by user or date) to quickly retrieve past queries or reports. This setup is also robust and future-proof for more complex querying and analytics needs as the product grows.

### External Integrations

- **Search Engine API:** Integrates with a SERP (Search Engine Results Page) API such as Firecrawl or SerpAPI to fetch Google search results relevant to the highlighted text.
- **LLM APIs:** Utilizes Large Language Model APIs (e.g., OpenAI or Anthropic) to generate summaries, explanations, or deep-dive answers based on the gathered search results and context.
- **Google OAuth:** Uses Google OAuth 2.0 for user authentication, allowing users to sign in with their Google accounts securely. This provides a convenient login method and access control without managing separate passwords.

## Key Workflows

DeepDive supports several core user workflows, from initiating a research query to managing authentication and preferences. Below are the primary workflows and their step-by-step processes.

### User Highlights Text & Initiates Query

1. **Text Highlight:** The user highlights a piece of text on a webpage that they want to research further.
2. **Popup Appears:** The extension's content script detects the highlight and displays the DeepDive popup UI near the selected text, offering a "Run Deep Research" button.
3. **Research Triggered:** The user clicks the "Run Deep Research" button in the popup. The content script sends a request to the background script, which in turn calls the DeepDive backend API with the highlighted text and user context.
4. **Search Query Execution:** The backend receives the request and first uses the SERP API integration to fetch relevant Google search results for the highlighted text.
5. **LLM Summarization:** The backend then takes the search results and queries the LLM API to generate a coherent summary or deep-dive explanation from those results.
6. **Result Storage:** The resulting summary (and possibly the full detailed report) is saved in the database under the user's query history, unless the user has enabled Private Mode for this query (in which case, no data is stored).
7. **UI Update:** The extension UI is updated with the summary of the research. The user sees a concise summary directly in the popup, and there is an option to download or view the full detailed report.

### Authentication & API Key Management

1. **OAuth Login:** The user clicks a login button (for example, "Sign in with Google") in the extension. This initiates the Google OAuth flow.
2. **Token Issuance:** Upon successful Google authentication, the backend receives an OAuth token and creates or updates the user record in the database. The backend then issues a session token or uses the OAuth token to maintain the user's logged-in session in the extension.
3. **API Key Entry:** Within the extension's settings (in the popup UI), the user can enter their personal API keys (for services like OpenAI or the chosen SERP API, if required). These keys are needed if the extension is to use the user's own quotas or accounts for those external services.
4. **Secure Storage:** The extension sends these API keys to the backend, where they are encrypted and stored in the database associated with the user's account. This ensures the keys are kept secure and are not exposed in plaintext, either in transit or at rest.
5. **Usage:** Subsequent research queries use these stored API keys to authenticate requests to the LLM or search services, unless the user switches to a built-in or default API key mode.

### Configurable Response Handling

1. **Summary Length Selection:** The user can choose a preferred summary length (e.g., *Short*, *Medium*, *Long*) in the extension settings or prior to running a query.
2. **Result Formatting:** When a deep research query is processed, the backend and LLM tailor the output to approximately match the requested length. The summary content is trimmed or expanded according to this setting while preserving essential information.
3. **Full Report Access:** Regardless of summary length, the full detailed report of the query (including all findings, sources, and possibly the raw LLM output) is prepared. The extension provides an option for the user to download this full report as a `.txt` or `.pdf` file for further reading or reference.

## UI & Component Design

The user interface of DeepDive is designed to be unobtrusive and user-friendly, appearing only when needed (on text highlight) and providing clear options. The design is broken down into several components:

### Popup UI

- **Trigger & Display:** A small popup interface that appears on the webpage when the user highlights text. It contains the main controls for DeepDive.
- **Controls:** Includes the "Run Deep Research" button to start the query. It also provides toggles or options such as enabling/disabling Private Mode (to decide whether to save the query) and a dropdown or setting for selecting summary length (short or long summary).
- **User Feedback:** Displays the returned summary once the research is complete. It also shows a link or button to download the full report. The design is minimalist to avoid cluttering the browsing experience.

### Content Script

- **Event Listener:** Runs in the context of web pages and listens for user actions (specifically, text highlights). When text is highlighted, it triggers the logic to display the DeepDive popup UI at the cursor location.
- **Popup Management:** Injects or toggles the visibility of the popup UI component in the page. It ensures the popup appears next to the selected text and is dismissed if the user clicks away.
- **Action Relay:** Captures user interactions with the popup (like clicking the research button or changing settings) and relays these events to the background script for processing.

### Background Script

- **API Communication:** Receives instructions from the content script (e.g., "user initiated a research query") and makes the appropriate calls to the DeepDive backend API. It acts as a middleman between the extension (frontend) and the backend server.
- **Session Management:** Keeps track of whether the user is logged in and attaches necessary authentication tokens or identifiers to requests. It also listens for OAuth callbacks if the authentication flow is performed via a redirect.
- **State Management:** Maintains any needed extension state in the background (such as caching recent query results or storing the Private Mode setting in memory if needed for quick access).

## Authentication & Security

- **OAuth Authentication:** DeepDive relies on Google OAuth for user login, which offloads credential management to Google. This means the extension never sees the user's password, and it receives a secure token upon authentication.
- **API Key Security:** Any API keys provided by the user (for third-party services like OpenAI or SERP APIs) are transmitted securely to the backend and stored in encrypted form. This protects the keys from exposure, as only the backend (with proper decryption) can use them when needed.
- **Data Privacy (Private Mode):** If Private Mode is enabled by the user for a query, the system will refrain from logging the query content or results to the database. This mode ensures that no trace of the query is stored, catering to users who want additional privacy for certain searches.
- **General Security:** All communication between the extension and the backend is done over HTTPS to prevent eavesdropping. At this stage, no additional custom encryption or security layers are added beyond the industry-standard practices, though future enhancements may include more robust encryption or security audits.

## Testing Strategy

To ensure reliability, the DeepDive extension will undergo multiple levels of testing:

- **Unit Tests:** Individual components and functions (especially in the frontend React code and backend query processing logic) will be tested with frameworks like Jest and React Testing Library for UI, and PyTest for the Python backend. This ensures that each unit performs as expected in isolation.
- **Integration Tests:** Testing will cover the interaction between components, such as API calls from the frontend to the backend and database operations. For example, using a testing environment to simulate a full query cycle (highlight to summary) and verifying each part (extension to API, API to DB, etc.) works together correctly.
- **End-to-End Tests:** Simulated user scenarios will be run, possibly with tools like Selenium or Puppeteer, to automate a Chrome browser instance. These tests will replicate a real user highlighting text, logging in, running a research query, and receiving results. This level of testing ensures the entire system works seamlessly from the user's perspective.
- **Manual Testing & QA:** In addition to automated tests, the extension will be manually tested across different websites and content types to ensure the highlight detection and popup display are robust. QA will also verify the OAuth flow and error handling (e.g., network failures, expired tokens).

## Future Considerations

- **Multi-Browser Support:** Extend compatibility to other browsers like Firefox and Microsoft Edge using their respective extension frameworks. This may involve code adjustments and testing in those environments.
- **Performance & Cost Optimization:** Implement rate limiting on how often the extension can call external APIs (especially the LLM and SERP services) to control costs. Caching of recent query results could be added to avoid duplicate external calls for the same highlights or queries.
- **Analytics & Monitoring:** Introduce analytics to track usage patterns (e.g., how often users use the service, common query lengths) and error monitoring to quickly catch and fix issues. This might include integrating services like Sentry for error reporting or Google Analytics (or a privacy-conscious alternative) for usage stats.
- **Enhanced Security:** As the user base grows, consider additional security measures such as two-factor authentication for accessing stored query history, auditing and logging admin access to the backend, and periodic security reviews of the codebase.
- **UI/UX Improvements:** Gather user feedback to refine the UI. Potential improvements could include customizable themes for the popup, keyboard shortcuts to trigger DeepDive, or the ability to handle multiple highlighted sections in one query.

