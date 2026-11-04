# Google Maps Platform Utility Skills

Task-specific agent skills for Google Maps Platform, such as SDK migrations.
These skills are independently maintained by the owning product teams and are
not part of the `google-maps-platform` entry-point skill, so install them
directly.

## Skills

| Skill | Use it to |
| :---- | :-------- |
| [`migrating-navsdk-7x-to-8x-android`](./migrating-navsdk-7x-to-8x-android/SKILL.md) | Upgrade an Android app from Navigation SDK 7.x to 8.0. |

## Installation

Install a single skill with the `skills` CLI:

```bash
npx skills add googlemaps/agent-skills --skill migrating-navsdk-7x-to-8x-android
```

Or copy the skill directory into your agent's skills folder, for example
`.gemini/skills/` or `.claude/skills/`.

## Feedback

To report a problem with a skill or suggest an improvement, open an issue in
[googlemaps/agent-skills](https://github.com/googlemaps/agent-skills/issues)
and include the skill name in the title.
