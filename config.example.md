# Private configuration template (do not fill this file in the public repository)

Keep an actual `config.local.md` **outside Git** or supply these values privately when the skill runs. Do not commit secrets, personal learner data, or a filled configuration.

- Supabase project ID: `<YOUR_PROJECT_ID>`
- Learner slug: `<YOUR_LEARNER_SLUG>`
- Learner timezone: `<YOUR_IANA_TIMEZONE>`
- Database schema: `english`
- Daily review count: `5`

The Supabase MCP connector must already be authorized to the selected project. **Never put an API key, service-role token, access token, password, or URL with embedded credentials into the skill, prompts, or GitHub repository.** The private learner's level and history are read from the connected database at runtime.