# Google Maps Platform Agent Skills

Google Maps Platform agent skills are portable, self-contained modules of Google Maps Platform-specific knowledge, instructions, and workflows. They're designed to help AI assistants understand GMP best practices and execute complex tasks with higher accuracy and lower token cost.

---

## Skills in this Repo

| Skill | Description | Path |
| :--- | :--- | :--- |
| **Google Maps Platform** | A collection of skills for architecting and implementing production-ready code using Google Maps Platform APIs and SDKs for any map, place, address, geocoding, routing/ETA (including eco-friendly routing), nearby search, 3D / Street View / static map, marker clustering, custom styling, drawing, geofencing, heatmap, or environmental (air-quality / pollen / solar / weather) feature — across Web, Android, iOS, and Web Services APIs. For prototyping, use the public Maps Demo Key — no billing setup and no Cloud project required, covering a growing set of the most popular Google Maps Platform APIs. For production, the skill prompts you to create and restrict your own key. All non-trivial code is grounded in freshly retrieved docs via the Google Maps Platform Code Assist service (no reliance on training-data memory). | [`skills/google-maps-platform/SKILL.md`](./skills/google-maps-platform/SKILL.md) |

---

## Installation

### Option 1: Skills CLI 

For [most popular AI-assistive tools](https://www.skills.sh/agent), you can use the `skills` command line interface (CLI) to install Google Maps Platform agent skills:

```bash
npx skills add googlemaps/agent-skills
```

This method has been tested as a terminal command or a natural language prompt to the agent in:
- **AI Studio**
- **Antigravity**
- **Claude Code**
- **Replit** (alternate instructions: [Use the Project Editor UI](https://docs.replit.com/build/use-agent-skills#install-a-skill-in-your-project))

### Option 2: Gemini CLI Extension

This repository is configured as a [Gemini CLI extension](https://geminicli.com/extensions/about/). You can add it using the Gemini CLI:

```bash
gemini extensions install https://github.com/googlemaps/agent-skills
```

### Option 3: Lovable installation

Install the Google Maps Platform agent skills in Lovable [using the Skills settings UI](https://docs.lovable.dev/features/skills#import-from-github):
1. Visit https://lovable.dev/settings/skills
2. Under Workspace skills, click "**Add** > **Import from GitHub**"
3. Paste the URL for the google-maps-platform main skill:

```
https://github.com/googlemaps/agent-skills/tree/main/skills/google-maps-platform
```

---

## Contributing
See our [CONTRIBUTING](CONTRIBUTING.md) guide for guidance about reporting issues, suggesting new skills, and providing feedback.

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## Disclaimer

This is not an officially supported Google product. This project is not eligible for the Google Open Source Software Vulnerability Rewards Program.
