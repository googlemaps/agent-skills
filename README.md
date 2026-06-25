# Google Maps Platform Agent Skills

Google Maps Platform agent skills are portable, self-contained modules of Google Maps Platform-specific knowledge, instructions, and workflows. They're designed to help AI assistants understand GMP best practices and execute complex tasks with higher accuracy and lower token cost.

---

## Skills in this Repo

| Skill | Description | Path |
| :--- | :--- | :--- |
| **Google Maps Platform** | A collection of skills for architecting and implementing production-ready code using Google Maps Platform APIs and SDKs for any map, place, address, geocoding, routing/ETA (including eco-friendly routing), nearby search, 3D / Street View / static map, marker clustering, custom styling, drawing, geofencing, heatmap, or environmental (air-quality / pollen / solar / weather) feature — across Web, Android, iOS, and Web Services APIs. For prototyping, use the public Maps Demo Key — no billing setup and no Cloud project required, covering a growing set of the most popular Google Maps Platform APIs. For production, the skill prompts you to create and restrict your own key. All non-trivial code is grounded in freshly retrieved docs via the Google Maps Platform Code Assist service (no reliance on training-data memory). | [`skills/google-maps-platform/SKILL.md`](./skills/google-maps-platform/SKILL.md) |

---

## Installation

### Option 1: Agent Skills CLI 

For most popular AI-assistive tools, you can use the `skills` CLI to install Google Maps Platform agent skills:

```bash
npx skills add googlemaps/agent-skills
```

### Option 2: Gemini CLI Extension

This repository is configured as a Gemini CLI extension. You can add it using the Gemini CLI:

```bash
gemini extensions install https://github.com/googlemaps/agent-skills
```

### Option 3: Manual setup

1. Clone this repository:

   ```bash
   git clone https://github.com/googlemaps/agent-skills.git
   ```

2. Copy the contents of the `google-maps-platform` directory to the appropriate location for your AI tool. Common locations include:
   - **Cursor**: `.cursor/rules/`
   - **Windsurf**: `.windsurfrules/`
   - **GitHub Copilot**: `.github/copilot-instructions.md` (or project-specific instruction files)

---

## Contributing
See our [CONTRIBUTING](CONTRIBUTING.md) guide for guidance about reporting issues, suggesting new skills, and providing feedback.

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

## Disclaimer

This is not an officially supported Google product. This project is not eligible for the Google Open Source Software Vulnerability Rewards Program.
