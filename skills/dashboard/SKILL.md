---
name: dashboard
description: Take a dashboard from the business need to a buildable spec — reader, questions, KPIs, layout, charts, measures.
disable-model-invocation: true
argument-hint: "[who it is for and what it must answer, or a report to critique]"
---

Dashboard: $ARGUMENTS

1. **Load the method.** Invoke the `data-workflow:dashboard-design` skill; when the tool is Power BI or Fabric, also
   `data-workflow:power-bi`.
2. **New or existing.** A request for a new dashboard runs the `dashboard-design` steps 1 to 7. An existing report
   (screenshots, a `.pbix`/`.pbip` project, a link) is read page by page and reviewed against the same steps and rules.
3. **Ask only what changes the design.** When the reader, their device, or their decisions are unknown, ask those in one
   message; decide everything else and state the assumption.
4. **Deliver the spec**, in this order:
   - Reader, device, frequency, and the questions, numbered.
   - KPI dictionary: each KPI with its formula, grain, filters, comparison, and the question it answers; in Power BI,
     the DAX measure written per the `power-bi` skill.
   - Layout: an ASCII wireframe of the phone layout when the reader uses a phone, then the desktop page, with each
     visual's position, chart type, and the question number it answers.
   - Interaction: slicers, filter pane, drillthrough, tooltips, and the interactions turned off.
   - Color and accessibility: the palette with its meaning, the contrast checks, alt text per visual.
   - Data needs: the tables and grain the model must provide, and anything missing from it.
5. **Critique** (for an existing report): each finding with the rule it breaks, the visual, and the fix, ranked by how much
   it slows the reader. Done when every visual has a verdict and every planned visual answers a numbered question.
