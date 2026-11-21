# Superbloo for Claude

Superbloo is an AI video editor. This plugin lets Claude hire Superbloo's editing agent, Bloo, from any conversation: give Claude a brief and some footage, and it comes back with a finished video, a review pack to check it against, and a download link. Claude can revise by timecode or by element name, and export when you approve.

It connects Claude to Superbloo's remote server at `https://api.superbloo.ai/mcp` and adds a skill that teaches Claude when and how to use it.

## What you can do

- Cut a long recording into short clips, reels or shorts, with captions.
- Reframe wide footage to vertical, square or 4:5.
- Add titles, lower thirds, motion graphics, music and sound design.
- Clean up a talking-head or podcast edit: remove silences and filler, balance loudness.
- Review every version with a script of the edit, contact sheets and automated checks, then ask for changes ("at 0:14 move the title up").
- Open the same project in the Superbloo web or desktop editor at any time.

Two modes are available. By default Bloo edits. If you ask, Claude can edit the timeline itself using Superbloo's editing tools (editing quality then depends on Claude), and hand single tasks such as captions to Bloo.

## Install

- **Claude (claude.ai, Cowork, Claude Code):** add Superbloo from Anthropic's directory once it is listed, or add the connector `https://api.superbloo.ai/mcp` yourself under Settings, Connectors.
- **Claude Code, connector only:** `claude mcp add --transport http superbloo https://api.superbloo.ai/mcp`
- **Claude Code, whole plugin (connector plus skill):** `/plugin install superbloo --marketplace Superbloo-AI/superbloo-plugin`

The first time Claude calls a Superbloo tool, you sign in with your Superbloo account in the browser (OAuth). You can revoke access at any time in your Superbloo settings. You need a Superbloo account with credits, see the pricing page.

## Try it

> Take this video https://example.com/interview.mp4 and make a 45 second vertical clip of the best moment, with captions and a title. Use Superbloo.

## Pricing

Jobs spend your Superbloo credits at the app's prices plus 10 percent for plugin and API use. When Claude edits itself, there is no Superbloo model charge; media processing and generation are billed as usual. Ask Claude to set a credit limit on a job. See https://superbloo.ai/pricing.

## Privacy

What Superbloo receives through this plugin: the brief Claude writes from your request, the footage links or files you provide, the edit notes you send, and the account you sign in with. Footage and results are stored in your Superbloo workspace (object storage on Cloudflare R2, project data in a Postgres database) so you can open, download and delete them. They are processed on GPU and AI providers only for the duration of a job. Superbloo does not sell personal data and does not train models on your private content without an explicit opt-in. Media is never routed through Claude; Claude receives links, status and text. Deleting a project or your account removes your content on the schedule in the privacy policy.

Full policy: https://superbloo.ai/privacy-policy. Terms: https://superbloo.ai/terms-of-service. Questions: privacy@superbloo.ai.

## Support

Email support@superbloo.ai or see https://superbloo.ai/support. Developer docs: https://superbloo.ai/docs/agents/overview.

## License

MIT, see [LICENSE](LICENSE). The Superbloo service itself is a hosted product governed by its terms of service.
