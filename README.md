# Hopkin for Claude

**Stop switching tabs. Get your ad performance inside Claude.**

Hopkin connects your Meta, Google, LinkedIn, Reddit, TikTok, ChatGPT Ads, Mailchimp, Google Search Console, and Google Analytics accounts to Claude, so you can ask questions, pull reports, and surface insights without opening a dashboard.

## What you can do

Ask Claude things like:

- *"Which Meta campaigns are above target CPA this week?"*
- *"Compare Google vs Meta spend and ROAS for the last 30 days."*
- *"Rank my creatives by CPA and show demographic breakdowns."*
- *"Flag any campaigns that underspent their budget this month."*
- *"How are my LinkedIn Ads performing compared to Meta this quarter?"*
- *"Which subreddits are driving the most conversions for my Reddit campaigns?"*
- *"Which search queries lost the most clicks month over month?"*

Claude answers with live data from your own accounts.

## Install

### Claude Code

```bash
/plugin marketplace add hopkin-ai/claude-plugin
/plugin install hopkin@hopkin
```

### Claude Desktop and claude.ai

Go to **Customize → Personal plugins → + → Add marketplace** and enter:

```
https://github.com/hopkin-ai/claude-plugin
```

Then install the **Hopkin** plugin.

### Connect your accounts

The plugin requires a Hopkin account. Sign up at [hopkin.ai](https://www.hopkin.ai). The first time Claude uses a Hopkin tool, it walks you through a one-time OAuth login at [app.hopkin.ai](https://app.hopkin.ai), where you connect the ad and analytics accounts you want Claude to read.

## What's included

One install gives you every supported platform. Each platform has a skill that teaches Claude the reporting workflows for it, and a remote MCP server that supplies the data.

| Platform | Skill | MCP server |
|---|---|---|
| Meta (Facebook/Instagram) Ads | `hopkin-meta-ads` | `https://meta.mcp.hopkin.ai/mcp` |
| Google Ads | `hopkin-google-ads` | `https://google.mcp.hopkin.ai/mcp` |
| LinkedIn Ads | `hopkin-linkedin-ads` | `https://linkedin.mcp.hopkin.ai/mcp` |
| Reddit Ads | `hopkin-reddit-ads` | `https://reddit.mcp.hopkin.ai/mcp` |
| TikTok Ads | `hopkin-tiktok-ads` | `https://tiktok.mcp.hopkin.ai/mcp` |
| ChatGPT Ads (OpenAI Ads) | `hopkin-chatgpt-ads` | `https://chatgpt.mcp.hopkin.ai/mcp` |
| Mailchimp | `hopkin-mailchimp` | `https://mailchimp.mcp.hopkin.ai/mcp` |
| Google Search Console | `hopkin-google-search-console` | `https://gcs.mcp.hopkin.ai/mcp` |
| Google Analytics (GA4) | `hopkin-google-analytics` | `https://ga4.mcp.hopkin.ai/mcp` |

The plugin contains only Markdown skills and the MCP server configuration in [`.mcp.json`](.mcp.json). It ships no hooks, scripts, or executables, and runs nothing on your machine.

## Data and privacy

- **Where data goes.** The plugin talks only to the nine Hopkin MCP servers listed above, all on `*.mcp.hopkin.ai` over HTTPS. Those servers call each platform's official API on your behalf, using the accounts you connected at app.hopkin.ai. The skills also contain links for you to open yourself, such as app.hopkin.ai, each platform's ads manager, and the free Hopkin Tag Inspector Chrome extension; the plugin never fetches them.
- **Authentication.** Each MCP server uses OAuth with your Hopkin account. The plugin itself stores no credentials, and asks for no API keys.
- **Read-focused.** The tools retrieve and analyze reporting data. They do not create, edit, pause, or delete campaigns, ads, or budgets on any platform. The only changes they make are to your Hopkin workspace itself, such as renaming or sharing a connection, or tracking a competitor in the ad library.
- **Usage logging.** Hopkin records which tools are called and with what parameters, to operate and improve the service.

See the [Privacy Policy](https://www.hopkin.ai/privacy-policy) and [Terms of Service](https://www.hopkin.ai/terms-of-service) for details.

## Support

- Documentation: [hopkin.ai/docs](https://www.hopkin.ai/docs/getting-started)
- Issues and feature requests: [GitHub Issues](https://github.com/hopkin-ai/claude-plugin/issues)

## License

[MIT](LICENSE). The license covers the contents of this repository. Use of the Hopkin service is governed by the Hopkin Terms of Service.
