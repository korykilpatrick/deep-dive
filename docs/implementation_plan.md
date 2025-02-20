# Implementation Plan for DeepDive Chrome Extension

## Project Setup & Configuration

- [ ] **Step 1: Initialize the Chrome Extension Project**  
  - **Task**: Set up the Chrome extension project structure with TypeScript, React, and Tailwind CSS.  
  - **Files**:  
    - `manifest.json`: Define Chrome extension metadata and permissions.  
    - `src/popup/index.tsx`: Basic React component for the popup UI.  
    - `src/background.ts`: Background script initialization.  
    - `src/content.ts`: Content script to handle webpage interactions.  
    - `webpack.config.js`: Webpack configuration for bundling.  
  - **User Instructions**: Run `npm install` after setting up the project.

- [ ] **Step 2: AWS Deployment Setup**  
  - **Task**: Configure the AWS environment for deployment (EC2 for backend, RDS for PostgreSQL, Secrets Manager for API keys).  
  - **User Instructions**:  
    - Set up an AWS EC2 instance for backend hosting.  
    - Configure AWS RDS for PostgreSQL.  
    - Store API keys in AWS Secrets Manager.

## Backend Development

- [ ] **Step 3: Set Up FastAPI Backend**  
  - **Task**: Initialize FastAPI with endpoints for handling research queries, authentication, and database interactions.  
  - **Files**:  
    - `backend/main.py`: Main FastAPI app.  
    - `backend/routers/auth.py`: Google OAuth authentication flow.  
    - `backend/routers/research.py`: API endpoints for handling research queries.  
    - `backend/models.py`: Database schema definitions.  
    - `backend/config.py`: Configuration file to fetch secrets from AWS Secrets Manager.  
  - **Step Dependencies**: Step 1, Step 2.

- [ ] **Step 4: Implement PostgreSQL Database Schema**  
  - **Task**: Define tables for users, queries, and settings in PostgreSQL using SQLAlchemy.  
  - **Files**:  
    - `backend/models.py`: Define SQLAlchemy models.  
    - `backend/migrations/`: Database migration scripts.  
  - **User Instructions**: Apply migrations with `alembic upgrade head`.  
  - **Step Dependencies**: Step 3.

## Chrome Extension UI & Interactions

- [ ] **Step 5: Implement Popup UI**  
  - **Task**: Build the popup UI for initiating research queries, logging in, and managing settings.  
  - **Files**:  
    - `src/popup/App.tsx`: React UI components.  
    - `src/popup/Login.tsx`: Google OAuth login component.  
    - `src/popup/Settings.tsx`: Manage API keys and user preferences.  
  - **Step Dependencies**: Step 1.

- [ ] **Step 6: Implement Content Script & Event Handling**  
  - **Task**: Capture highlighted text on web pages, display the popup on selection, and send research requests to the background script.  
  - **Files**:  
    - `src/content.ts`: Handles text selection events and injects UI elements into web pages.  
  - **Step Dependencies**: Step 5.

- [ ] **Step 7: Implement Background Script & API Communication**  
  - **Task**: Relay research requests from the content script to the backend, handle user authentication, and manage external API calls.  
  - **Files**:  
    - `src/background.ts`: Handles API requests to the backend and manages session tokens.  
  - **Step Dependencies**: Step 6.

## Research Query Processing

- [ ] **Step 8: Integrate Firecrawl SERP API**  
  - **Task**: Implement integration with the Firecrawl Search Engine Results Page (SERP) API to fetch search results for user queries.  
  - **Files**:  
    - `backend/serp.py`: Firecrawl API integration logic.  
  - **Step Dependencies**: Step 3.

- [ ] **Step 9: Integrate OpenAI API for Summarization**  
  - **Task**: Use the OpenAI API to generate summaries from the search results obtained via Firecrawl.  
  - **Files**:  
    - `backend/llm.py`: OpenAI API integration for language model summarization.  
  - **Step Dependencies**: Step 8.

## Authentication & User Data Handling

- [ ] **Step 10: Implement Google OAuth Flow**  
  - **Task**: Allow users to log in via Google OAuth2 and manage session tokens for authenticated requests.  
  - **Files**:  
    - `backend/routers/auth.py`: OAuth authentication logic (requesting and handling Google login).  
    - `src/popup/Login.tsx`: UI for login/logout via Google account.  
  - **Step Dependencies**: Step 3, Step 5.

- [ ] **Step 11: Implement API Key Management**  
  - **Task**: Store and retrieve user API keys (for Firecrawl and OpenAI) securely using AWS Secrets Manager.  
  - **Files**:  
    - `backend/config.py`: Logic to fetch secrets from AWS Secrets Manager.  
    - `src/popup/Settings.tsx`: UI for entering and saving API keys and preferences.  
  - **Step Dependencies**: Step 10.

## Research Report Handling

- [ ] **Step 12: Generate and Store Research Reports**  
  - **Task**: Format the aggregated research results into a Markdown report and store it for user retrieval.  
  - **Files**:  
    - `backend/reports.py`: Markdown report generation and storage logic.  
  - **Step Dependencies**: Step 9.

- [ ] **Step 13: Implement Google Drive Integration for Saving Reports**  
  - **Task**: Allow users to save or export the research reports directly to their Google Drive.  
  - **Files**:  
    - `backend/drive.py`: Google Drive API integration for file upload.  
  - **Step Dependencies**: Step 12.

## Final Steps & Optimization

- [ ] **Step 14: Testing & Debugging**  
  - **Task**: Conduct unit tests, integration tests, and end-to-end tests to ensure the extension and backend work reliably together.  
  - **Step Dependencies**: All prior steps.

- [ ] **Step 15: Deploy Backend to AWS**  
  - **Task**: Deploy the FastAPI backend on AWS EC2, configure the PostgreSQL RDS instance, and set up an API Gateway if needed for external access.  
  - **User Instructions**: Deploy the backend either manually or via a CI/CD pipeline, ensuring environment variables and secrets are properly configured on the server.  
  - **Step Dependencies**: All prior backend-related steps.
