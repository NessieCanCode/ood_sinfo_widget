# OnDemand System Status Widget

**Reusable, transferable SLURM partition-level status widget for OnDemand dashboards.**  
Pulls raw `sinfo` output, summarizes per-partition usage, offline nodes, and shows utilization with visual bars and badges.

## About
![System Status Widget](Screenshot.gif)

This widget displays current system status by parsing the output of `sinfo` (SLURM) and rendering a compact, human-friendly overview of each partition: total vs used nodes, offline nodes, and percent utilization, with classification coloring. It is intended to be dropped into existing OnDemand-style dashboards with minimal changes so other sites/institutions can adopt the same status surface.

## Features

- Fetches live SLURM `sinfo` output embedded server-side.
- Aggregates per-partition totals, used, and offline node counts.
- Heuristic subtitle assignment based on partition naming conventions.
- Visual utilization bar with thresholds (green/yellow/red).
- Shows offline node count and relative usage.
- “Last updated” timestamp and manual refresh.
- Graceful fallback when `sinfo` output is missing or malformed.

## Installation

Clone the repository:

```bash
git clone https://github.com/NessieCanCode/ood_sinfo_widget.git
cd ood_system_status_widget
```

Place or customize the widget file (`_sinfo.html.erb`) with any local adjustments (e.g., `sinfo` format string, subtitle logic, or styling).

Create the symlink so the dashboard picks it up (adjust path if your OnDemand installation differs):

```bash
ln -s _sinfo.html.erb /etc/ood/config/apps/dashboard/views/widgets/_sinfo.html.erb
```

## Usage

On page load, the script will:
- Parse the embedded raw `sinfo` output from the `<pre>` block.
- Aggregate node states per partition.
- Replace the raw text with styled status sections.
- Update the “Last updated” timestamp.
- Allow manual refresh via the refresh button.

## Configuration & Heuristics

### Data Source
The widget expects `sinfo` output in this format:
```erb
<pre>
  <%= `sinfo -o "%P %D %t %C %m %G %N" 2>/dev/null` %>
</pre>
```
Ensure the output includes column names such as `PARTITION`, `NODES`, and `STATE`. Adjust the format string if your SLURM environment differs.

### Subtitle Logic
Partition subtitles are inferred from naming patterns:
- `dept.*` → "department, condo"
- `pi.*` → "pi, condo"
- `grp.*` → "group, condo"
- Known public partitions like `short`, `medium`, `long`, `gpu`, `bigmem`, `test` → "public, shared"
- `cenvalarc.*` → "cenval-arc, shared"
- Default fallback → "standard, wide"

This logic lives in the `renderStatus` function and can be extended.

## CSS: Defaults & Overrides

Below are the actual rules the widget uses by default; you can override any of these to fit your branding or preferences.

```css
/* Header / title / action button */
.widget-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.25rem;
  font-weight: 600;
  color: #222;
}
.widget-title {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.widget-action {
  font-size: 1rem;
  padding: 0.2rem 0.8rem;
  border: none;
  background: white;
  border-radius: 999px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

/* Container for all partition status sections */
.sinfo-container {
  font-family: system-ui, sans-serif;
  color: #111;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  padding: 1rem;
  background: #f9fafa;
  display: grid;
  grid-auto-flow: column;
  grid-template-rows: repeat(3, 1fr);
  grid-auto-columns: 300px;
  gap: 1rem;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: thin;
  scrollbar-color: #888 #eee;
}
.sinfo-container::-webkit-scrollbar {
  height: 8px;
}
.sinfo-container::-webkit-scrollbar-track {
  background: #eee;
  border-radius: 4px;
}
.sinfo-container::-webkit-scrollbar-thumb {
  background-color: #888;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}
.sinfo-container::-webkit-scrollbar-thumb:hover {
  background-color: #555;
}

/* Individual partition card */
.sinfo-section {
  background: #fff;
  border-radius: 10px;
  padding: 1rem 1.5rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06);
  position: relative;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.sinfo-title {
  font-size: 1.1rem;
  font-weight: 700;
  margin-bottom: 2px;
  text-transform: lowercase;
}
.sinfo-subtitle {
  font-size: 0.85rem;
  color: #666;
  margin-bottom: 0.5rem;
}
.sinfo-status {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.95rem;
  margin-bottom: 0.25rem;
}

/* Utilization bar */
.sinfo-bar-wrapper {
  height: 6px;
  background: #eee;
  border-radius: 4px;
  overflow: hidden;
  margin: 4px 0 6px;
}
.sinfo-bar-fill {
  height: 100%;
  transition: width 0.3s ease;
}
.red { background-color: #f44336; }
.yellow { background-color: #ffeb3b; color: #000; }
.green { background-color: #4caf50; }

/* Chips and offline indicator */
.status-chip {
  background-color: #d4f6da;
  color: #0f8029;
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 999px;
  font-weight: 600;
}
.offline {
  font-size: 0.85rem;
  color: #444;
  text-align: right;
}

/* Timestamp line */
.last-updated {
  font-size: 0.75rem;
  color: #666;
  margin-top: -0.5rem;
}

/* Fallback / error state */
.no-data-msg {
  padding: 1rem;
  background: #fff;
  border-radius: 10px;
  font-size: 0.95rem;
  text-align: center;
  color: #555;
}
```

### Example Overrides

```css
/* Stronger offline emphasis */
.offline {
  font-weight: 700;
  color: #b71c1c;
}

/* Custom utilization thresholds */
.sinfo-bar-fill.green {
  background-color: #28a745;
}
.sinfo-bar-fill.yellow {
  background-color: #ffb300;
}
.sinfo-bar-fill.red {
  background-color: #e53935;
}
```

## Error Handling

- If `sinfo` output is missing or malformed, shows:
  ```html
  <div class="no-data-msg">Unable to retrieve system status. SLURM may be unreachable.</div>
  ```
- Refresh reloads the page content with `cache: "no-store"` to attempt fresh data.

## Extensibility Ideas

- Link partition names to deeper dashboards.
- Make subtitle mappings and thresholds configurable via `data-` attributes or external config.
- Add sorting, collapsing, or alerting based on usage.
- Background polling instead of full reload for refresh.
- Localize timestamp formatting or add relative “X minutes ago”.

## Troubleshooting

- **Empty/invalid output**: Verify `sinfo` is runnable and that the ERB interpolation executes.  
- **Parsing issues**: Confirm header labels match expectations (`PARTITION`, `NODES`, `STATE`).  
- **Missing styles**: Ensure the CSS for the widget’s classes is loaded.  
- **Refresh no-op**: Check that caching isn’t preventing updated HTML and that the button wiring is intact.

## Contributing

1. Fork the repo.  
2. Improve heuristics, add configuration abstraction, or enhance accessibility.  
3. Submit a PR including:
   - Sample SLURM output used for testing.
   - Description of any new subtitle rules or thresholds.
   - Optional before/after screenshots.

## License

MIT License — see the `LICENSE` file for details.
