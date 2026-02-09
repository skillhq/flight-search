# Google Flights Interaction Patterns

Deep-dive cookbook for automating Google Flights with agent-browser. Covers the tricky interactions that require careful handling.

**Related**: [../SKILL.md](../SKILL.md) for the main workflow.

## Contents

- [Full Annotated Walkthrough](#full-annotated-walkthrough)
- [Airport Autocomplete Deep Dive](#airport-autocomplete-deep-dive)
- [Date Picker Calendar Navigation](#date-picker-calendar-navigation)
- [Multi-City Searches](#multi-city-searches)
- [Scrolling for More Results](#scrolling-for-more-results)
- [Price Graph and Explore Mode](#price-graph-and-explore-mode)

---

## Full Annotated Walkthrough

A complete command-by-command walkthrough for: **Round trip, BKK to NRT, March 20-27, 1 adult, Economy**.

```bash
# ── Step 1: Open Google Flights ──
agent-browser --session flights open "https://www.google.com/travel/flights"
agent-browser --session flights wait 3000
# Wait for full page load. Google Flights is a heavy SPA.

agent-browser --session flights snapshot -i
# Expected output: interactive elements including:
#   @e1 [combobox] "Where from?"      ← origin field
#   @e2 [combobox] "Where to?"        ← destination field
#   @e3 [button] "Departure"          ← departure date
#   @e4 [button] "Return"             ← return date
#   @e5 [button] "1 passenger"        ← passengers
#   @e6 [button] "Round trip"         ← trip type
#   @e7 [button] "Economy"            ← cabin class
#   @e8 [button] "Search"             ← search button
# NOTE: Actual ref numbers WILL vary. Always read the snapshot.

# ── Step 2: Check for consent banner ──
# If you see a consent/cookie dialog, handle it first:
# agent-browser --session flights click @eN  (Accept all / Reject all)
# agent-browser --session flights wait 2000
# agent-browser --session flights snapshot -i

# ── Step 3: Enter Origin ──
agent-browser --session flights click @e1
# Clicking the origin combobox focuses it and may show recent searches.

agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i
# After clicking, the field expands. Look for an input element.
# The focused input may have a different ref now.

agent-browser --session flights fill @eN "BKK"
# Use fill (clears first) with the IATA code.

agent-browser --session flights wait 2000
# CRITICAL: Must wait for autocomplete dropdown to populate.
# Google Flights queries its backend for matching airports.

agent-browser --session flights snapshot -i
# Expected: dropdown with suggestions like:
#   @e15 [listitem] "Bangkok (BKK) Suvarnabhumi Airport"
#   @e16 [listitem] "Bangkok (DMK) Don Mueang Airport"

agent-browser --session flights click @e15
# Click the correct suggestion. For BKK, pick Suvarnabhumi.

agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i
# The origin field now shows "BKK" as a chip/tag.

# ── Step 4: Enter Destination ──
agent-browser --session flights click @eN
# Click the destination combobox (ref changed from step 3).

agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

agent-browser --session flights fill @eN "NRT"
agent-browser --session flights wait 2000
agent-browser --session flights snapshot -i
# Expected: dropdown with:
#   @eN [listitem] "Tokyo (NRT) Narita International Airport"
#   @eN [listitem] "Tokyo (HND) Haneda Airport"

agent-browser --session flights click @eN
# Click the NRT suggestion.

agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# ── Step 5: Set Departure Date ──
agent-browser --session flights click @eN
# Click the departure date button/field.

agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i
# A calendar overlay appears showing 2 months.
# Look for month headers and individual day numbers.
# Expected: "February 2026" and "March 2026" side by side.

# If March isn't visible yet, navigate forward:
agent-browser --session flights click @eN   # ">" / next month arrow
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# Click day 20 in the March section
agent-browser --session flights click @eN   # The "20" element under March
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i
# Departure date is now set. Calendar stays open for return date.

# ── Step 6: Set Return Date ──
# Calendar should still be open. March 27 should be visible.
agent-browser --session flights click @eN   # The "27" element under March
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i

# Click "Done" to close the calendar
agent-browser --session flights click @eN   # "Done" button
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# ── Step 7: Search ──
# IMPORTANT: The "Done" button only closes the calendar. You MUST click "Search" separately.
agent-browser --session flights click @eN   # "Search" button
agent-browser --session flights wait 5000
# Results load asynchronously. 5 seconds is usually enough.

# ── Step 8: Extract Results ──
agent-browser --session flights get text body
# This returns all visible text on the page, including:
# - "Best departing flights" / "Cheapest" / "Best" labels
# - Airline names, times, durations, stops, prices
# - Carbon emission estimates

# ── Step 9: Close ──
agent-browser --session flights close
```

---

## Airport Autocomplete Deep Dive

The airport autocomplete is the #1 source of failures. Here's every failure mode and recovery strategy.

### How It Works

Google Flights airport fields are `combobox` elements. When focused:
1. A text input appears (sometimes replacing the combobox)
2. Typing triggers a debounced API call (~500ms)
3. A dropdown `listbox` renders with matching airports
4. Each suggestion is a `listitem` or `option` with airport name + code

### Failure Mode 1: Dropdown Doesn't Appear

**Symptom**: After typing and waiting 2s, snapshot shows no dropdown.

**Recovery**:
```bash
# Try pressing Escape and starting over
agent-browser --session flights press Escape
agent-browser --session flights wait 500
agent-browser --session flights click @eN     # Re-click the field
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# Clear and retype
agent-browser --session flights fill @eN "BKK"
agent-browser --session flights wait 3000     # Wait longer this time
agent-browser --session flights snapshot -i
```

If still no dropdown:
```bash
# Try the full city name instead of IATA code
agent-browser --session flights fill @eN "Bangkok"
agent-browser --session flights wait 3000
agent-browser --session flights snapshot -i
```

### Failure Mode 2: Wrong Airport Selected

**Symptom**: The chip shows the wrong airport (e.g., DMK instead of BKK).

**Recovery**:
```bash
# Look for an "X" or close button next to the airport chip
agent-browser --session flights snapshot -i
agent-browser --session flights click @eN     # The "X" on the chip
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i
# Now re-enter the airport code
```

### Failure Mode 3: Input Field Doesn't Accept Text

**Symptom**: `fill` command seems to work but snapshot shows field is empty.

**Recovery**:
```bash
# The combobox may need a click to expand into an input
agent-browser --session flights click @eN
agent-browser --session flights wait 500

# Try using type instead of fill
agent-browser --session flights type @eN "BKK"
agent-browser --session flights wait 2000
agent-browser --session flights snapshot -i
```

### Failure Mode 4: Multiple Airport Chips

**Symptom**: Clicking a suggestion adds it alongside an existing airport.

Google Flights supports multiple origin/destination airports. If you see multiple chips:
```bash
# Remove the unwanted one by clicking its "X"
agent-browser --session flights click @eN
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i
```

### Best Practices Summary

| Practice | Why |
|----------|-----|
| Use `fill` not `type` | Clears existing text first |
| Wait 2-3 seconds after typing | Autocomplete needs API roundtrip |
| Always click the suggestion | Enter doesn't reliably select |
| Use IATA codes (3-letter) | Most precise matching |
| Re-snapshot after every interaction | DOM changes invalidate refs |

---

## Date Picker Calendar Navigation

### Calendar Structure

The Google Flights calendar renders as a **long scrollable list of date buttons** (not a traditional grid). Each button includes:
- Full date label: `"Friday, March 20, 2026 , 10645 Thai baht"`
- Near-term dates include estimated round-trip prices
- Far-future dates show just the date (no price)

Key elements:
- `textbox "Departure"` and `textbox "Return"` at the top
- `button "Reset"` to clear selections
- `button "Done."` to confirm and close calendar
- `button "Next"` to load more distant months
- All individual dates as `button` elements

### Navigating to a Far-Future Month

The calendar loads months progressively. If your target date isn't visible:

```bash
# Click "Next" to load more months
agent-browser --session flights click @eN   # "Next" button
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i
# Repeat until target month is visible
```

**Tip**: Since dates render as buttons with full labels (e.g., "Friday, March 20, 2026"), you can search the snapshot output for your target date string to find the right ref quickly.

### Selecting Dates That Span Months

For a trip departing March 28 and returning April 5:

```bash
# 1. Navigate so March is visible
# 2. Click departure day (28) in March panel
agent-browser --session flights click @eN    # "28" in March
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i

# 3. Calendar may auto-advance. Check if April is now visible.
#    If not, click ">" to show April.
agent-browser --session flights click @eN    # ">" if needed
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i

# 4. Click return day (5) in April panel
agent-browser --session flights click @eN    # "5" in April
agent-browser --session flights wait 500
agent-browser --session flights snapshot -i

# 5. Click "Done"
agent-browser --session flights click @eN
```

### Common Calendar Issues

**Issue: Day numbers are ambiguous** — The snapshot may show "20" twice (once for each visible month).
**Solution**: Look at surrounding context in the snapshot. Day elements are grouped under their month header. Pick the one under the correct month.

**Issue: Calendar doesn't close after selecting dates** — Click the "Done" button explicitly.

**Issue: Date range highlights the wrong range** — The first click sets departure, second sets return. If reversed, click both dates again in the correct order.

---

## Multi-City Searches

### Setting Up Multi-City

```bash
# 1. Change trip type to "Multi-city"
agent-browser --session flights click @eN    # Trip type dropdown
agent-browser --session flights snapshot -i
agent-browser --session flights click @eN    # "Multi-city" option
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# The form now shows multiple flight legs (rows), each with:
# - Origin field
# - Destination field
# - Date field
```

### Adding Flight Legs

```bash
# Fill in Leg 1: BKK → NRT on March 20
# (follow standard airport autocomplete + date picker flow)

# Fill in Leg 2: NRT → ICN on March 25
# (the destination of leg 1 may auto-populate as origin of leg 2)

# Add another leg if needed:
agent-browser --session flights click @eN    # "Add flight" button
agent-browser --session flights wait 1000
agent-browser --session flights snapshot -i

# Fill in Leg 3: ICN → BKK on March 30
```

### Tips for Multi-City

- Each leg has its own origin/destination/date fields — treat them independently
- The "Add flight" button adds a new row
- The "X" next to a leg removes it
- Always re-snapshot after modifying any leg, as refs shift

---

## Scrolling for More Results

Google Flights initially shows ~10 results. To see more:

```bash
# Check if there's a "Show more flights" button or link
agent-browser --session flights snapshot -i
# Look for "Show more", "View more", or similar

# If found, click it
agent-browser --session flights click @eN    # "Show more flights"
agent-browser --session flights wait 3000
agent-browser --session flights snapshot -i

# If no button, try scrolling down
agent-browser --session flights scroll down 1000
agent-browser --session flights wait 2000
agent-browser --session flights snapshot -i
```

To extract all visible results after expanding:
```bash
agent-browser --session flights get text body
```

---

## Price Graph and Explore Mode

> **Note**: These are advanced features. The core search workflow above covers most use cases.

### Price Graph / Date Grid

Google Flights offers a "Price graph" or date grid view to see prices across multiple dates:

```bash
# After searching, look for "Price graph" or "Date grid" tab/toggle
agent-browser --session flights snapshot -i
agent-browser --session flights click @eN    # "Price graph" or "Date grid"
agent-browser --session flights wait 3000
agent-browser --session flights snapshot -i

# Extract the pricing data
agent-browser --session flights get text body
```

### Explore Mode

Google Flights "Explore" lets you search by map/region without a specific destination:

```bash
agent-browser --session flights open "https://www.google.com/travel/explore"
agent-browser --session flights wait 3000
agent-browser --session flights snapshot -i
# Shows a map with prices to various destinations from your location
```

This is useful when users ask "where can I fly cheaply from X?" but the interaction patterns are less predictable than the standard search flow. Use with caution and rely on `get text body` for data extraction.
