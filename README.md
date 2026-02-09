# flight-search

Google Flights search skill for [Claude Code](https://claude.com/claude-code). Find flight prices, schedules, and availability using browser automation via [agent-browser](https://github.com/nicobailey/agent-browser).

## Install

```bash
claude skill add skillhq/flight-search
```

## What it does

Ask Claude to search for flights and it will:

1. Open Google Flights via `agent-browser`
2. Search using a direct URL (fast path) or interactive form filling (fallback)
3. Extract results and present them as a formatted table

## Triggers

- "Find flights from BKK to NRT, March 20-27"
- "How much is a flight from LAX to London?"
- "Cheapest flights from JFK to CDG in June"
- "One-way business class from SFO to Tokyo, April 15"

## Capabilities

| Feature | Method | Support |
|---------|--------|---------|
| Round trip | URL fast path | Direct results in 3 commands |
| One way | URL fast path | Direct results in 3 commands |
| Business / First class | URL fast path | Direct results in 3 commands |
| Multiple passengers | URL fast path | Direct results in 3 commands |
| Adults + children | URL fast path | Direct results in 3 commands |
| Premium economy | Interactive fallback | Form automation |
| Multi-city (3+ legs) | Interactive fallback | Form automation |

## Requirements

- [agent-browser](https://github.com/nicobailey/agent-browser) CLI installed and available in PATH

## Files

```
SKILL.md                              # Main skill (triggers, workflow, rules)
references/
  interaction-patterns.md             # Deep-dive cookbook for tricky interactions
```

## License

MIT
