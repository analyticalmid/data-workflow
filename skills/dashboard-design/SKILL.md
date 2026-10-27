---
name: dashboard-design
description: Dashboard and report UI/UX design — audience and decisions first, KPI hierarchy, layout and grid, choosing the chart for the question, color and accessibility (WCAG contrast, color not alone), filters and drill, phone layout, and page performance, with Power BI specifics. Use when designing, critiquing, or rebuilding a dashboard, report page, or KPI view in any BI tool.
---

# Dashboard design

A dashboard is done when its main reader answers their recurring question in a few seconds, on the device they read it
on, without having to ask what a number means.

## Steps

1. **Name the reader and the decisions.** Who opens it, how often, on which device, and the three to five questions it
   answers. Done when every planned visual maps to one of those questions; a visual that maps to none is cut.
2. **Define each KPI** in one line: formula, grain, filters, and the comparison that gives it meaning (target, prior
   period, same period last year). A number without a comparison is a fact, not a KPI.
3. **Lay out by importance.** The headline KPIs top-left, trends next, breakdowns below, detail behind drillthrough or
   tooltips. Design the phone layout first when the reader uses a phone, then widen it. Done when the main page fits
   one screen without scrolling.
4. **Choose each chart** for its question with [Charts](#charts).
5. **Apply color and accessibility** with [Color and accessibility](#color-and-accessibility).
6. **Design the interaction**: the few slicers that matter, in the same place on every page; the rest in the filter
   pane; edit interactions so a slicer changes only the visuals it should.
7. **Check performance**: few visuals per page, Top N on long tables, totals only where they mean something. Done when
   the slowest visual loads in a time the reader accepts.
8. **Review** against every rule below and say which ones the design breaks and why it is acceptable.

## Charts

| Question | Chart |
| --- | --- |
| How far from a reference? (deviation) | diverging bar, bullet against target |
| How does it change over time? | line (many points), column (few periods) |
| Which is biggest, in order? (ranking) | sorted bar |
| How big compared with others? (magnitude) | bar or column |
| What is it made of? (part-to-whole) | stacked bar, treemap, waterfall; pie only below about 8 slices and only for the share of the whole |
| How is it spread? (distribution) | histogram, box plot |
| Do two measures move together? (correlation) | scatter |
| Where? (spatial) | map only when location itself matters |
| How does it move between states? (flow) | Sankey |

- Bars and columns start at zero; lines and dot plots may zoom. https://www.datawrapper.de/academy/why-our-column-and-bar-charts-start-at-zero
- No 3D, no gauges for comparison, consistent scales and sort order across visuals, three or four significant digits
  (3.4 M, not 3,412,987.22), and one time frame per axis.
  https://learn.microsoft.com/en-us/power-bi/create-reports/service-dashboards-design-tips
- Categories: Financial Times Visual Vocabulary,
  https://github.com/Financial-Times/chart-doctor/tree/main/visual-vocabulary

## Color and accessibility

- Color carries meaning (good, bad, selected, one highlighted series), with a neutral color for everything else.
- Text contrast at least 4.5:1 (3:1 for large text, 18 pt or 14 pt bold); lines, bars, and controls at least 3:1
  against what touches them. https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html ·
  https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast.html
- Color is never the only signal: add labels, markers with distinct shapes, or patterns.
  https://www.w3.org/WAI/WCAG22/Understanding/use-of-color.html
- Colorblind-safe pairs: blue with orange or red; vary lightness so the chart reads in grayscale; label lines directly
  instead of a legend. Avoid green with red, brown, blue, grey, or black, and blue with purple or grey.
  https://www.datawrapper.de/blog/colorblindness-part2 ·
  https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-accessibility-creating-reports
- Key information never lives only in a tooltip or a hover.

## Power BI

- Alt text on every visual (up to 250 characters; it can be a DAX measure), a set tab order with decorative shapes
  hidden from it, titles without acronyms; readers have keyboard navigation, the screen reader, high contrast, and Show
  data table (Alt+Shift+F11). https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-accessibility-creating-reports
- The phone layout is a separate portrait canvas built from the page's visuals, shown in the Power BI mobile apps in
  portrait; a browser always shows the desktop page.
  https://learn.microsoft.com/en-us/power-bi/create-reports/power-bi-create-mobile-optimized-report-about
- The KPI visual needs a value, a target, and a trend axis, with the direction set (high is good, or low is good).
  https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-kpi
- Sync slicers across pages; an "apply all slicers" button saves queries on DirectQuery.
  https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-slicers
- Fewer visuals per page, detail on drillthrough pages, Top N filters on large tables.
  https://learn.microsoft.com/en-us/power-bi/guidance/power-bi-optimization
