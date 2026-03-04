# ✈️ flight-search

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/Agent-Skill-blueviolet)](https://skills.sh)

Google Flights search skill for any coding agent — Claude Code, Codex, and more. Find flight prices, compare cabins, and get booking links from your terminal.

## Install

```bash
npx skills add https://github.com/skillhq/flight-search --skill google-flights
```

## Triggers

- "Find flights from BKK to NRT, March 20-27"
- "How much is a flight from LAX to London?"
- "Cheapest flights from JFK to CDG in June"
- "One-way business class from SFO to Tokyo, April 15"

## Example Output

```
✈️ Flights: BKK → NRT (Mar 20–27, Round Trip)

| # | Airline  | Dep      | Arr      | Duration | Stops   | Economy    | Business   | Biz Delta            |
|---|----------|----------|----------|----------|---------|------------|------------|----------------------|
| 1 | JAL      | 8:05 AM  | 4:00 PM  | 5h 55m   | Nonstop | THB 23,255 | THB 65,915 | +THB 42,660 (+183%) |
| 2 | THAI     | 10:30 PM | 6:20 AM  | 5h 50m   | Nonstop | THB 28,165 | THB 75,000 | +THB 46,835 (+166%) |
| 3 | Air Japan| 12:10 AM | 8:15 AM  | 6h 05m   | Nonstop | THB 20,515 | —          | —                    |
| 4 | ZIPAIR   | 11:45 PM | 7:30 AM  | 5h 45m   | Nonstop | THB 21,425 | —          | —                    |

Want booking links for any of these? Just say which one.
```

Economy and business results are fetched in parallel and merged with a delta column so you can see the upgrade cost at a glance.

## Booking Options

After picking a flight, the skill extracts booking providers with prices and direct links:

```
📋 Booking Options for JAL BKK→NRT (5h 55m, Nonstop)

| Provider    | Price      | Book                          |
|-------------|------------|-------------------------------|
| Emirates    | THB 28,960 | [Continue](https://...)       |
| Booking.com | THB 29,512 | [Continue](https://...)       |
| Teaflight   | THB 28,171 | [Continue](https://...)       |
```

## Capabilities

| Feature | Method | Support |
|---------|--------|---------|
| Round trip | URL fast path | Direct results in 3 commands |
| One way | URL fast path | Direct results in 3 commands |
| Business / First class | URL fast path | Direct results in 3 commands |
| Multiple passengers | URL fast path | Direct results in 3 commands |
| Adults + children | URL fast path | Direct results in 3 commands |
| Booking links extraction | Click → snapshot | Provider prices + direct URLs |
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
