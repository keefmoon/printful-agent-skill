# Printful AgentSkills

This repository provides **two** [AgentSkills](https://agentskills.io/) designed to allow AI agents to interact seamlessly with the Printful REST API. 

Because Printful is actively migrating to a new API version, we offer independent skills for both the mature V1 API and the new V2 Beta API.

By providing **one** of these skill directories to a compatible AI Agent, it immediately gains the context, instructional material, and completely resolved schemas necessary to manage a Printful store, browse the product catalog, create fulfillment orders, and generate product mockups.

## ⚠️ Important Usage Warning
**Do NOT provide both the V1 and V2 skills to your agent at the same time.** 
Giving an agent both versions of the API simultaneously is highly likely to cause confusion, hallucinated endpoints, and incorrect JSON schemas. You should select **only one** skill to provide to the agent based on your preferred API version.

## Available Skills
- `skills/printful-rest-api/`: The stable, production-ready Printful API (V1). Use this for absolute stability or features like Store Sync that are not yet available in V2.
- `skills/printful-rest-api-v2/`: The new modern Printful API V2 (Beta). Features Leaky Bucket rate limiting, Itemized order building, and RFC 9457 errors. 

## Features
- **Comprehensive API Guidelines**: Setting the skill active natively injects `SKILL.md`, containing an overview of Printful's authentication, rate-limiting, and best practices.
- **Detailed Component References**: The `references/` directories contain auto-generated, highly detailed markdown references for every specific part of the Printful API (Orders, Catalog, Mockups, etc.). These fully resolve the complex Printful OpenAPI specification, making it cost-effective and lightning-fast for an LLM to read.

## Usage
1. Choose **one** directory (`skills/printful-rest-api` OR `skills/printful-rest-api-v2`) and provide it to your AgentSkills-compatible AI agent.
2. Ensure you have obtained a Printful Private API Token or set up Public App OAuth credentials via the [Printful Developer Portal](https://developers.printful.com/).
3. Instruct the agent to perform tasks! (e.g., *"Use the Printful API to find the variant ID for a white unisex staple t-shirt, and create a draft order sending it to Sydney."*)
