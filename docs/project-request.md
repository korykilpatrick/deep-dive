# Deep Dive
## Project Description
A Chrome browser extension powered by deep-research, enabling users to highlight text on any webpage and initiate automated research using AI agents that recursively query SERP APIs and LLMs. The extension supports Google OAuth-based authentication, storage of user queries (with an optional “private mode”), configurable summary lengths, and downloadable full reports.

## Target Audience
- Researchers, knowledge workers, students
- Tech-savvy users who have their own OpenAI and Firecrawl API keys

## Desired Features

### Core Functionality
- [ ] Highlight text and trigger deep-research query via hotkey or popup
  - [ ] Standard query mode with default prompts
  - [ ] Advanced mode allowing custom prompts, follow-up questions, and link-only or summary-only outputs
- [ ] Integration with SERP API for automated Google searches
- [ ] LLM integration for summarizing results and generating responses

### Authentication & Keys
- [ ] Google OAuth flow for user login
- [ ] Simple UI to input and store user’s OpenAI and Firecrawl API keys

### Data Handling
- [ ] Default: Store user queries/results to remote DB
  - [ ] “Private Mode” toggle to skip saving any data
- [ ] Option to configure response length (short summary vs. longer summary)
  - [ ] When a user requests a full report, generate and download as a file

### User Settings & Customization
- [ ] Configurable prompt templates
  - [ ] Ability to save, edit, and delete templates
- [ ] Basic usage control (e.g., limiting calls, user pays directly through their own API keys)
- [ ] Minimal popup interface

## Design Requests
- [ ] Minimalist, single-view popup with:
  - [ ] Simple “Run Deep Research” button
  - [ ] Prominent toggle for “Private Mode”
  - [ ] Clear instructions for short/long summary and file download option
- [ ] Light branding and icons consistent with deep-research’s look

## Other Notes
- Start with Chrome only
- Monitor SERP API rate limits, consider caching to manage costs
- Potential for multi-browser support in the future