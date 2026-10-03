# OpenAI plugin source

`app-6a8ccd4370188191932fe94b18d9b3eb/` is the source package for the existing Maeve Social OpenAI plugin. It retains its identity and listing metadata and connects to the new `/mcp/openai` endpoint. Version 2.0.1 contains the individual-tool endpoint and updated skill.

The repository root remains the package for the existing four-tool `/mcp` connection. Its MCP configurations and shared skill are unchanged by the OpenAI endpoint work. OpenAI has its own skill because its tools take direct arguments and do not use catalog discovery or generic execution.

The OpenAI endpoint registers the existing MCP catalog operations as individual tools using their maintained wire schemas, descriptions and annotations. Calls use the existing dispatcher, workspace permissions, availability checks, confirmations and audit flow. No operation implementation is duplicated.

The endpoint has its own OAuth resource and protected-resource metadata at `/.well-known/oauth-protected-resource/mcp/openai`. Tokens and sessions cannot be reused between the two MCP endpoints. Existing `/mcp` clients retain their current resource and four tools.

This is local source, not a deployed or submitted release. Deployment of the backend and updating the existing OpenAI plugin's endpoint and imported skill are separate authorized release steps. Reconnect against the new resource when testing. OpenAI must scan the individual tools and updated skill before publication; local validation does not guarantee approval. Do not submit the repository-root package for this OpenAI configuration.
