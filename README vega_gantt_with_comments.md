![Andel Projects Limited](perpop.png)

# INTRO TO VEGA GANTT CHART BASED ON ORIGINAL BY DAVID BACCI

## Introduction

#### Davide Bacci created a much admired Gantt chart for deployment in Power BI via Deneb which Daniel Marsh-Patrick developed for wider use in creating Vega based visuals. There has been much interest in certain parts of the LinkedIn community [ including mine :)] with Davide's chart and a number of YouTube videos have emerged. The problem until now was that Vega, like, JSON did not support comments, however Daniel provided the functionality in Deneb V 2.0. This has allowed me, as part of my wider project, to provide a guide to how the gantt actually works.

## Header Section

The header section defines the overall structure and foundational aspects of the Vega Gantt chart visualization. It specifies that we are using the Vega v5 schema, which ensures compatibility with the syntax and features of that specific version of the Vega grammar.
"$schema" : This indicates the version of the Vega specification being used (in this case, version 5). It acts as a validation reference to guide the renderer in interpreting the JSON correctly.
"description": A textual summary of the visualization’s purpose or content. This is primarily for documentation and clarification purposes.
"autosize" : Determines how the visualization should resize in response to container size changes or different layouts. The "pad" option ensures the padding around the chart content remains consistent while resizing.
"width" : Specifies the total width of the visualization in pixels, which is critical for determining the available space for displaying elements like tasks and columns in the Gantt chart.
"padding" : Defines the amount of padding around the visualization. This ensures that elements like axes and labels do not get clipped and there is some breathing room for visual elements.

```
{
"$schema"    : "https://vega.github.io/schema/vega/v5.json",
"description": "Modified Dataviz with updated colors, sample tasks, and additional fields for Status and Owner.",
"autosize"   : "pad",
"padding"    : {"left": 5, "right": 0, "top": 5, "bottom": 0},
```

## Signals Section

### Signals for Interactivity and Layout Control

Signals provide a way to make visualizations interactive and responsive by defining dynamic variables whose values can change based on user interactions or computed values.

Basic Signals : These include static values or initial states like chart height, tooltip visibility, or layout parameters. For example, setting a default height or showing tooltips by default.
Zoom and Pan : Signals enable zooming and panning functionality through mouse events like wheel movements or dragging. Expressions control the scaling and movement along the x and y domains (time and task axes).
Task Control Signals : Signals may control the visibility or collapsibility of tasks, phases, or dependencies. These enable user-driven exploration by expanding or collapsing groups or highlighting dependencies.
Dynamic Calculation Signals: These include computed values, such as "dayBandwidth" (pixels per day on the x-axis), and are used to adapt visual layouts based on data scales, user inputs, or other dynamic triggers.

```

"signals": [
{"name": "height", "update": "500"},
{"name": "showTooltips", "value": true},
{"name": "showButtons", "value": true},
{"name": "showDomainSpanLabel", "value": false},
{
"name": "startGrain",
"value": "Months",
"description": "Days, Months, Years or All"
},
{"name": "textColour", "value": "#333333"},
{
"name": "coloursDark",
"value": ["#1f77b4", "#ff7f0e", "#2ca02c", "#d62728", "#9467bd"]
},
{
"name": "coloursLight",
"value": ["#aec7e8", "#ffbb78", "#98df8a", "#ff9896", "#c5b0d5"]
},
{"name": "yRowHeight", "value": 33, "description": "Height in pixels"},
{
"name": "yRowPadding",
"value": 0.22,
"description": "Row padding as % of yRowHeight (each side)"
},
{"name": "yPaddingInner", "update": "yRowPadding * yRowHeight"},
{"name": "taskColumnWidth", "value": 135},
{"name": "ownerColumnWidth", "value": 80}, // New column width
{"name": "startColumnWidth", "value": 45},
{"name": "endColumnWidth", "value": 45},
{"name": "statusColumnWidth", "value": 80}, // New column width
{"name": "daysColumnWidth", "value": 35},
{"name": "progressColumnWidth", "value": 55},
{"name": "columnPadding", "value": 15},
{"name": "oneDay", "update": "1000*60*60*24"},
{
"name": "dayBandwidth",
"update": "scale('x', timeOffset('day', datetime(2000,1,1),1)) - scale('x', datetime(2000,1,1))"
},
{"name": "dayBandwidthRound", "update": "(round(dayBandwidth _100)/100)"},
{"name": "minDayBandwidth", "value": 20},
{"name": "minMonthBandwidth", "value": 3},
{"name": "minYearBandwidth", "value": 0.95},
{"name": "milestoneSymbolSize", "value": 400},
{"name": "arrowSymbolSize", "value": 70},
{"name": "phaseSymbolHeight", "update": "bandwidth('y')-yPaddingInner-5"},
{"name": "phaseSymbolWidth", "value": 10},
{
"name": "columnsWidth",
"update": "taskColumnWidth + ownerColumnWidth + startColumnWidth + endColumnWidth + statusColumnWidth + daysColumnWidth + progressColumnWidth + (columnPadding _ 7)"
},
{"name": "ganttWidth", "update": "width - columnsWidth - minDayBandwidth"},
{
"name": "dayExt",
"update": "[data('xExt')[0]['s'] - oneDay, data('xExt')[0]['s'] + ((ganttWidth - minDayBandwidth) / minDayBandwidth) _ oneDay]"
},
{
"name": "monthExt",
"update": "[data('xExt')[0]['s'] - oneDay, data('xExt')[0]['s'] + ganttWidth / 2 _ oneDay]"
},
{
"name": "yearExt",
"update": "[data('xExt')[0]['s'] - oneDay, data('xExt')[0]['s'] + ganttWidth / 0.35 _ oneDay]"
},
{
"name": "allExt",
"update": "[data('xExt')[0]['s'] - oneDay, data('xExt')[0]['e'] + oneDay _ 9]"
},
{
"name": "xExt",
"update": "startGrain == 'All' ? dayExt : startGrain == 'Years' ? yearExt : startGrain == 'Months' ? monthExt : dayExt"
},
{"name": "today", "update": "utc(year(now()), month(now()), date(now()))"},
{"name": "todayRule", "update": "timeFormat(today, '%d/%m/%y')"},
{
"name": "zoom",
"value": 1,
"on": [
{
"events": "wheel!",
"force": true,
"update": "x() > columnsWidth ? pow(1.001, (event.deltaY) * pow(16, event.deltaMode)) : 1"
}
]
},
{"name": "xDomMinSpan", "update": "span(dayExt)"},
{"name": "xDomMaxSpan", "update": "round((ganttWidth / 0.13) _ oneDay)"},
{
"name": "xDom",
"update": "xExt",
"on": [
{
"events": {"signal": "xDomPre"},
"update": "span(xDomPre) < xDomMinSpan ? [anchor + (xDom[0] - anchor) _ (zoom _ (xDomMinSpan / span(xDomPre))), anchor + (xDom[1] - anchor) _ (zoom _ (xDomMinSpan / span(xDomPre)))] : span(xDomPre) > xDomMaxSpan ? [anchor + (xDom[0] - anchor) _ (zoom _ (xDomMaxSpan / span(xDomPre))), anchor + (xDom[1] - anchor) _ (zoom _ (xDomMaxSpan / span(xDomPre)))] : xDomPre"
},
{
"events": {"signal": "delta"},
"update": "[xCur[0] + span(xDom) _ delta[0] / width, xCur[1] + span(xDom) _ delta[0] / width]"
},
{"events": "dblclick", "update": "xExt"},
{
"events": "@buttonMarks:click",
"update": "datum.text == 'All' ? allExt : datum.text == 'Years' ? yearExt : datum.text == 'Months' ? monthExt : datum.text == 'Days' ? dayExt : xDom"
}
]
},
{"name": "scaledHeight", "update": "data('yScale').length _ yRowHeight"},
{
"name": "yRange",
"update": "[yRange != null ? yRange[0] : 0, yRange != null ? yRange[0] + scaledHeight : scaledHeight]",
"on": [
{
"events": [{"signal": "delta"}],
"update": "clampRange([yCur[0] + span(yCur) _ delta[1] / scaledHeight, yCur[1] + span(yCur) _ delta[1] / scaledHeight], height >= scaledHeight ? 0 : height - scaledHeight, height >= scaledHeight ? height : scaledHeight)"
},
{"events": "dblclick", "update": "[0, scaledHeight]"},
{
"events": {"signal": "closeAll"},
"update": "closeAll ? [0, scaledHeight] : yRange"
}
]
},
{
"name": "xDomPre",
"value": [0, 0],
"on": [
{
"events": {"signal": "zoom"},
"update": "[anchor + (xDom[0] - anchor) _ zoom, anchor + (xDom[1] - anchor) _ zoom]"
}
]
},
{
"name": "anchor",
"value": 0,
"on": [{"events": "wheel", "update": "+invert('x', x() - columnsWidth)"}]
},
{
"name": "xCur",
"value": [0, 0],
"on": [{"events": "pointerdown", "update": "slice(xDom)"}]
},
{
"name": "yCur",
"value": [0, 0],
"on": [{"events": "pointerdown", "update": "slice(yRange)"}]
},
{
"name": "delta",
"value": [0, 0],
"on": [
{
"events": [
{
"source": "window",
"type": "pointermove",
"consume": true,
"between": [
{"type": "pointerdown"},
{"source": "window", "type": "pointerup"}
]
}
],
"update": "down ? [down[0] - x(), y() - down[1]] : [0, 0]"
}
]
},
{
"name": "down",
"value": null,
"on": [
{"events": "pointerdown", "update": "xy()"},
{"events": "pointerup", "update": "null"}
]
},
{
"name": "phaseClicked",
"value": null,
"on": [
{
"events": "@taskSelector:click,@phaseOutline:click",
"update": "yCur[0] == yRange[0] && yCur[1] == yRange[1] && xCur[0] === xDom[0] && xCur[1] === xDom[1] && datum.phase == datum.task ? {phase: datum.phase} : null",
"force": true
},
{
"events": "@taskTooltips:click",
"update": "yCur[0] == yRange[0] && yCur[1] == yRange[1] && xCur[0] === xDom[0] && xCur[1] === xDom[1] && datum.datum.phase == datum.datum.task ? {phase: datum.datum.phase} : null",
"force": true
}
]
},
{
"name": "itemHovered",
"value": {"id": "", "dependencies": []},
"on": [
{
"events": "@taskSelector:mouseover,@phaseOutline:mouseover,@milestoneSymbols:mouseover,@taskBars:mouseover,@taskNames:mouseover,@taskLabels:mouseover",
"update": "{'id': toString(datum.id), 'dependencies': split(datum.dependencies, ',')}"
},
{
"events": "@taskTooltips:mouseover",
"update": "{'id': toString(datum.datum.id), 'dependencies': split(datum.datum.dependencies, ',')}"
},
{
"events": "@taskSelector:mouseout,@phaseOutline:mouseout,@milestoneSymbols:mouseout,@taskBars:mouseout,@taskNames:mouseout,@taskLabels:mouseout,@taskTooltips:mouseout",
"update": "{'id': '', 'dependencies': []}"
}
]
},
{
"name": "hover",
"value": "",
"on": [
{
"events": "@buttonMarks:pointerover",
"update": "datum.text ? datum.text : ''",
"force": true
},
{"events": "@buttonMarks:pointerout", "update": "''", "force": true}
]
},
{
"name": "closeAll",
"on": [
{
"events": "@buttonMarks:click",
"update": "datum.text == 'Close' ? true : false",
"force": true
}
]
},
{
"name": "openAll",
"on": [
{
"events": "@buttonMarks:click",
"update": "datum.text == 'Open' ? true : false",
"force": true
}
]
}
],
```

## Data Section

This section defines the input data used to generate the visualization, including transformations applied to derive specific properties.

Data Sources: Data objects describe the input data array, containing fields like task ID, phase, start/end dates, completion status, and dependencies. Data can be static (hardcoded values) or dynamic (loaded from an external source).
Data Transformations: These are operations applied to modify or generate new data attributes. Examples include:
Filtering: To include or exclude specific data points based on conditions (e.g., filtering milestones vs. tasks).
Aggregations: Calculating summaries, such as minimum start date for a phase, total completion percentage, etc.
Formulas: Creating derived fields, such as task duration (calculated by subtracting start from end date).
Lookups: Cross-referencing data sets for merging related information, like tasks with their phases.
Units: Date-time values often need parsing to be interpreted as dates and used in computations. Many transformations rely on units of pixels for layout calculations or units of time (days, weeks) for temporal computations.

```
    "data": [
    {
      "name": "collapsedPhases",
      "on": [
        {"trigger": "phaseClicked", "toggle": "phaseClicked"},
        {"trigger": "closeAll", "remove": true},
        {"trigger": "closeAll", "insert": "data('phases')"},
        {"trigger": "openAll", "remove": true}
      ]
    },
    {
      "name": "input",
      "values": [
        {
          "id": 1,
          "phase": "Planning",
          "task": "Project Kickoff",
          "milestone": null,
          "start": "01/06/2024",
          "end": "03/06/2024",
          "completion": 100,
          "status": "Complete",
          "owner": "John Doe",
          "dependencies": null,
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 2,
          "phase": "Planning",
          "task": "Define Scope",
          "milestone": null,
          "start": "04/06/2024",
          "end": "10/06/2024",
          "completion": 60,
          "status": "On Schedule",
          "owner": "Alice Smith",
          "dependencies": null,
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 3,
          "phase": "Design",
          "task": "Create Wireframes",
          "milestone": null,
          "start": "11/06/2024",
          "end": "20/06/2024",
          "completion": 40,
          "status": "On Schedule",
          "owner": "Bob Johnson",
          "dependencies": "2",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 4,
          "phase": "Design",
          "task": "Design Review",
          "milestone": true,
          "start": "21/06/2024",
          "end": "21/06/2024",
          "completion": 0,
          "status": "Future Task",
          "owner": "Carol Williams",
          "dependencies": "3",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 5,
          "phase": "Development",
          "task": "Backend Development",
          "milestone": null,
          "start": "22/06/2024",
          "end": "15/07/2024",
          "completion": 25,
          "status": "On Schedule",
          "owner": "David Brown",
          "dependencies": "4",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 6,
          "phase": "Development",
          "task": "Frontend Development",
          "milestone": null,
          "start": "22/06/2024",
          "end": "15/07/2024",
          "completion": 30,
          "status": "On Schedule",
          "owner": "Eve Davis",
          "dependencies": "4",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 7,
          "phase": "Testing",
          "task": "Unit Testing",
          "milestone": null,
          "start": "16/07/2024",
          "end": "25/07/2024",
          "completion": 10,
          "status": "Future Task",
          "owner": "Frank Miller",
          "dependencies": "5,6",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 8,
          "phase": "Testing",
          "task": "User Acceptance Testing",
          "milestone": null,
          "start": "26/07/2024",
          "end": "05/08/2024",
          "completion": 0,
          "status": "Future Task",
          "owner": "Grace Lee",
          "dependencies": "7",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 9,
          "phase": "Deployment",
          "task": "Deploy to Production",
          "milestone": true,
          "start": "06/08/2024",
          "end": "06/08/2024",
          "completion": 0,
          "status": "Future Task",
          "owner": "Henry Wilson",
          "dependencies": "8",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 10,
          "phase": "Maintenance",
          "task": "Monitor Systems",
          "milestone": null,
          "start": "07/08/2024",
          "end": "30/08/2024",
          "completion": 0,
          "status": "Future Task",
          "owner": "Ivy Martinez",
          "dependencies": "9",
          "hyperlink": "http://www.example.com"
        },
        {
          "id": 11,
          "phase": "Maintenance",
          "task": "Bug Fixes",
          "milestone": null,
          "start": "07/08/2024",
          "end": "30/08/2024",
          "completion": 0,
          "status": "Future Task",
          "owner": "Jack Taylor",
          "dependencies": "10",
          "hyperlink": "http://www.example.com"
        }
      ],
      "format": {"parse": {"start": "utc:'%d/%m/%Y'", "end": "utc:'%d/%m/%Y'"}},
      "transform": [
        {"type": "formula", "as": "labelEnd", "expr": "datum.end"},
        {"type": "formula", "as": "end", "expr": "datetime(+datum.end + oneDay)"},
        {
          "type": "formula",
          "as": "days",
          "expr": "(datum.end - datum.start) / oneDay"
        },
        {
          "type": "formula",
          "as": "completionLabel",
          "expr": "datum.completion + '%'"
        },
        {
          "type": "window",
          "sort": {"field": "start", "order": "ascending"},
          "ops": ["rank"],
          "as": ["taskSort"],
          "groupby": ["phase"]
        },
        {"type": "formula", "as": "start", "expr": "+datum.start"},
        {"type": "formula", "as": "end", "expr": "+datum.end"}
      ]
    },
    {
      "name": "phases",
      "source": "input",
      "transform": [
        {
          "type": "aggregate",
          "fields": [
            "start",
            "end",
            "completion",
            "task",
            "completion",
            "labelEnd"
          ],
          "ops": ["min", "max", "sum", "count", "mean", "max"],
          "as": ["start", "end", "sum", "count", "completion", "labelEnd"],
          "groupby": ["phase"]
        },
        {"type": "formula", "as": "task", "expr": "datum.phase"},
        {"type": "formula", "as": "taskSort", "expr": "0"},
        {
          "type": "formula",
          "as": "completion",
          "expr": "round(datum.completion)"
        },
        {
          "type": "formula",
          "as": "days",
          "expr": "(datum.end - datum.start) / oneDay"
        },
        {
          "type": "window",
          "sort": {"field": "start", "order": "ascending"},
          "ops": ["row_number", "row_number"],
          "as": ["phaseSort", "id"]
        },
        {
          "type": "formula",
          "as": "id",
          "expr": "length(data('input')) + datum.id + '^^^^^'"
        }
      ]
    },
    {
      "name": "phasePaths",
      "source": "phases",
      "transform": [
        {
          "type": "formula",
          "as": "phasePath",
          "expr": "'M ' + scale('x', datum.start) + ' ' + (scale('y', datum.id) + yPaddingInner) + ' H ' + scale('x', datum.end) + ' ' + ' v ' + phaseSymbolHeight + ' L ' + (scale('x', datum.end) - phaseSymbolWidth) + ' ' + (scale('y', datum.id) + yPaddingInner + phaseSymbolHeight / 2) + ' L ' + (scale('x', datum.start) + phaseSymbolWidth) + ' ' + (scale('y', datum.id) + yPaddingInner + phaseSymbolHeight / 2) + ' L ' + scale('x', datum.start) + ' ' + (scale('y', datum.id) + yPaddingInner + phaseSymbolHeight) + ' z'"
        }
      ]
    },
    {
      "name": "tasks",
      "source": "input",
      "transform": [
        {"type": "filter", "expr": "datum.milestone != true"},
        {
          "type": "filter",
          "expr": "!indata('collapsedPhases', 'phase', datum.phase)"
        }
      ]
    },
    {
      "name": "milestones",
      "source": "input",
      "transform": [
        {"type": "filter", "expr": "datum.milestone == true"},
        {
          "type": "filter",
          "expr": "!indata('collapsedPhases', 'phase', datum.phase)"
        }
      ]
    },
    {
      "name": "yScale",
      "source": ["tasks", "phases", "milestones"],
      "transform": [
        {
          "type": "lookup",
          "from": "phases",
          "key": "phase",
          "values": ["phaseSort"],
          "fields": ["phase"],
          "as": ["phaseSort"]
        },
        {
          "type": "window",
          "sort": {
            "field": ["phaseSort", "taskSort"],
            "order": ["ascending", "ascending"]
          },
          "ops": ["row_number"],
          "as": ["finalSort"]
        }
      ]
    },
    {
      "name": "xExt",
      "source": "input",
      "transform": [
        {
          "type": "aggregate",
          "fields": ["start", "end"],
          "ops": ["min", "max"],
          "as": ["s", "e"]
        },
        {"type": "formula", "as": "days", "expr": "(datum.e - datum.s) / oneDay"}
      ]
    },
    {
      "name": "weekends",
      "transform": [
        {
          "type": "sequence",
          "start": 0,
          "stop": {
            "signal": "dayBandwidthRound >= minMonthBandwidth ? span(xDom) / oneDay : 0"
          },
          "as": "sequence"
        },
        {
          "type": "formula",
          "as": "start",
          "expr": "datetime(utc(year(xDom[0]), month(xDom[0]), date(xDom[0])) + (oneDay * datum.sequence))"
        },
        {
          "type": "filter",
          "expr": "day(datum.start) == 6 || day(datum.start) == 0"
        },
        {
          "type": "formula",
          "as": "end",
          "expr": "datetime(+datum.start + (oneDay))"
        }
      ]
    },
    {
      "name": "taskDependencyArrows",
      "source": "yScale",
      "transform": [
        {
          "type": "filter",
          "expr": "isValid(datum.dependencies) && datum.dependencies != ''"
        }
      ]
    },
    {
      "name": "phaseDependencyArrows",
      "source": "input",
      "transform": [
        {
          "type": "filter",
          "expr": "indata('collapsedPhases', 'phase', datum.phase)"
        },
        {
          "type": "joinaggregate",
          "fields": ["id", "start"],
          "ops": ["values", "min"],
          "as": ["allPhaseIds", "start"],
          "groupby": ["phase"]
        },
        {"type": "formula", "as": "id", "expr": "toString(datum.id)"},
        {
          "type": "formula",
          "as": "allPhaseIds",
          "expr": "pluck(datum.allPhaseIds, 'id')"
        },
        {
          "type": "formula",
          "as": "dependencies",
          "expr": "split(datum.dependencies, ',')"
        },
        {"type": "flatten", "fields": ["dependencies"]},
        {
          "type": "formula",
          "as": "internalDependenciesIndex",
          "expr": "indexof(datum.allPhaseIds, datum.dependencies)"
        },
        {"type": "formula", "as": "milestone", "expr": "null"},
        {
          "type": "filter",
          "expr": "datum.dependencies != 'null' && datum.dependencies != '' && datum.internalDependenciesIndex == -1"
        },
        {
          "type": "lookup",
          "from": "phases",
          "key": "phase",
          "values": ["id"],
          "fields": ["phase"],
          "as": ["id"]
        }
      ]
    },
    {
      "name": "dependencyArrows",
      "source": ["taskDependencyArrows", "phaseDependencyArrows"]
    },
    {
      "name": "dependencyLines",
      "source": ["yScale", "phaseDependencyArrows"],
      "transform": [
        {
          "type": "filter",
          "expr": "isValid(datum.dependencies) && datum.dependencies != ''"
        },
        {
          "type": "formula",
          "as": "dependencies",
          "expr": "split(datum.dependencies, ',')"
        },
        {"type": "flatten", "fields": ["dependencies"]},
        {
          "type": "lookup",
          "from": "input",
          "key": "id",
          "values": ["id", "end", "phase"],
          "fields": ["dependencies"],
          "as": ["sourceId", "sourceEnd", "sourcePhase"]
        },
        {
          "type": "lookup",
          "from": "phases",
          "key": "phase",
          "values": ["id", "end"],
          "fields": ["sourcePhase"],
          "as": ["sourcePhaseId", "sourcePhaseEnd"]
        },
        {
          "type": "formula",
          "as": "sourceId",
          "expr": "indata('collapsedPhases', 'phase', datum.sourcePhase) == true ? datum.sourcePhaseId : datum.sourceId"
        },
        {
          "type": "formula",
          "as": "sourceEnd",
          "expr": "indata('collapsedPhases', 'phase', datum.sourcePhase) == true ? datum.sourcePhaseEnd : datum.sourceEnd"
        },
        {
          "type": "formula",
          "as": "plottedStart",
          "expr": "datum.milestone == null || datum.milestone == false ? scale('x', datum.start) - sqrt(arrowSymbolSize) - 1 : (scale('x', datum.start) + (dayBandwidth / 2) - (sqrt(milestoneSymbolSize) / 2) - sqrt(arrowSymbolSize)) - 1"
        },
        {
          "type": "formula",
          "as": "plottedSourceEnd",
          "expr": "scale('x', datum.sourceEnd) - (dayBandwidth / 2)"
        },
        {
          "type": "formula",
          "as": "a",
          "expr": "[datum.milestone == null || datum.milestone == false ? scale('x', datum.start) : scale('x', datum.start) + (dayBandwidth / 2) - (sqrt(milestoneSymbolSize) / 2), scale('y', datum.id) + bandwidth('y') / 2]"
        },
        {
          "type": "formula",
          "as": "b",
          "expr": "[datum.plottedStart >= datum.plottedSourceEnd ? datum.plottedSourceEnd : datum.plottedStart, scale('y', datum.id) + bandwidth('y') / 2]"
        },
        {
          "type": "formula",
          "as": "c",
          "expr": "[datum.plottedSourceEnd, scale('y', datum.sourceId) + bandwidth('y') / 2]"
        },
        {
          "type": "formula",
          "as": "d",
          "expr": "[datum.plottedStart > datum.plottedSourceEnd ? null : datum.plottedStart, datum.plottedStart > datum.plottedSourceEnd ? null : scale('y', datum.sourceId) + (bandwidth('y'))]"
        },
        {
          "type": "formula",
          "as": "e",
          "expr": "[datum.plottedStart > datum.plottedSourceEnd ? null : datum.plottedSourceEnd, datum.plottedStart > datum.plottedSourceEnd ? null : scale('y', datum.sourceId) + (bandwidth('y'))]"
        },
        {"type": "fold", "fields": ["a", "b", "d", "e", "c"]},
        {"type": "filter", "expr": "datum.value[0] != null"},
        {"type": "formula", "as": "value0", "expr": "datum.value[0]"},
        {"type": "formula", "as": "value1", "expr": "datum.value[1]"},
        {
          "type": "window",
          "ops": ["row_number"],
          "as": ["duplicates"],
          "groupby": ["id", "sourceId", "value0", "value1"]
        },
        {"type": "filter", "expr": "datum.duplicates == 1"}
      ]
    },
    {
      "name": "buttons",
      "values": [
        {"side": "left", "text": "Close", "x": 15, "leftRadius": 4},
        {"side": "left", "text": "Open", "x": 65, "rightRadius": 4},
        {"side": "right", "text": "All", "x": 250, "rightRadius": 4},
        {"side": "right", "text": "Years", "x": 300},
        {"side": "right", "text": "Months", "x": 350},
        {"side": "right", "text": "Days", "x": 400, "leftRadius": 4}
      ]
    }

],
"marks": [
{
"name": "buttonMarks",
"description": "All buttons",
"type": "group",
"from": {"data": "buttons"},
"clip": {"signal": "!showButtons"},
"encode": {
"update": {
"x": {
"signal": "datum.side == 'left' ? datum.x : columnsWidth + ganttWidth - datum.x"
},
"width": {"value": 50},
"y": {"value": -60},
"height": {"signal": "18"},
"stroke": {"signal": "'#7f7f7f'"},
"strokeWidth": {"value": 1},
"cornerRadiusTopLeft": {"field": "leftRadius"},
"cornerRadiusBottomLeft": {"field": "leftRadius"},
"cornerRadiusTopRight": {"field": "rightRadius"},
"cornerRadiusBottomRight": {"field": "rightRadius"},
"cursor": {"value": "pointer"},
"fill": [
{"test": "indexof(hover, datum.text) > -1", "value": "#1f77b4"},
{
"test": "datum.text == 'Close' && data('collapsedPhases').length == data('phases').length",
"value": "#1f77b4"
},
{
"test": "datum.text == 'Open' && data('collapsedPhases').length == 0",
"value": "#1f77b4"
},
{
"test": "datum.text == 'Days' && dayBandwidthRound == minDayBandwidth",
"value": "#1f77b4"
},
{
"test": "datum.text == 'Months' && dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minDayBandwidth",
"value": "#1f77b4"
},
{
"test": "datum.text == 'Years' && dayBandwidthRound < minYearBandwidth",
"value": "#1f77b4"
},
{"value": "white"}
]
}
},
"marks": [
{
"name": "buttonText",
"interactive": false,
"type": "text",
"encode": {
"update": {
"text": {"signal": "parent.text"},
"baseline": {"value": "middle"},
"align": {"value": "center"},
"x": {"signal": "item.mark.group.width / 2"},
"y": {"signal": "10"},
"fill": [
{"test": "indexof(hover, parent.text) > -1", "value": "white"},
{
"test": "parent.text == 'Close' && data('collapsedPhases').length == data('phases').length",
"value": "white"
},
{
"test": "parent.text == 'Open' && data('collapsedPhases').length == 0",
"value": "white"
},
{
"test": "parent.text == 'Days' && dayBandwidthRound == minDayBandwidth",
"value": "white"
},
{
"test": "parent.text == 'Months' && dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minDayBandwidth",
"value": "white"
},
{
"test": "parent.text == 'Years' && dayBandwidthRound < minYearBandwidth",
"value": "white"
},
{"value": "#7f7f7f"}
]
}
}
}
]
},
{
"name": "xDomainText",
"interactive": false,
"type": "text",
"encode": {
"update": {
"text": {
"signal": "showDomainSpanLabel ? timeFormat(xDom[0], '%d/%m/%y') + ' - ' + timeFormat(xDom[1], '%d/%m/%y') : null"
},
"baseline": {"value": "top"},
"align": {"value": "right"},
"x": {"signal": "columnsWidth + ganttWidth"},
"y": {"signal": "showDomainSpanLabel ? height + 15 : 0"},
"fill": {"signal": "textColour"}
}
}
},
{
"name": "phaseBackgrounds",
"description": "Background rect for phases",
"type": "rect",
"clip": true,
"zindex": 0,
"from": {"data": "phases"},
"encode": {
"update": {
"x": {"value": 0},
"x2": {"signal": "columnsWidth"},
"y": {"signal": "scale('y', datum.id)"},
"height": {"signal": "bandwidth('y')"},
"fill": {"value": "#e0f7fa"},
"opacity": {"value": 0.3}
}
}
},
{
"name": "taskLabelSizes",
"description": "Hidden label sizes to support tooltips when the task name doesn't completely fit",
"type": "text",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"enter": {
"x": {"value": -100},
"y": {"value": -100},
"fill": {"value": "transparent"},
"text": {"signal": "datum.task"},
"fontSize": {"value": 11}
}
}
},
{
"type": "rect",
"name": "taskTooltips",
"description": "Hidden rect to support tooltips when the task name doesn't completely fit",
"from": {"data": "taskLabelSizes"},
"clip": true,
"zindex": 101,
"encode": {
"update": {
"x": {"value": -15},
"x2": {"signal": "taskColumnWidth"},
"y": {"signal": "scale('y', datum.datum.id)"},
"height": {"signal": "bandwidth('y')"},
"fill": {"value": "transparent"},
"tooltip": {
"signal": "datum.bounds.x2 - datum.bounds.x1 >= taskColumnWidth - 16 ? datum.datum.task : null"
},
"cursor": {
"signal": "datum.datum.phase == datum.datum.task ? 'pointer' : 'auto'"
},
"href": {"field": "datum.hyperlink"}
}
}
},
{
"type": "group",
"name": "columnHolder",
"style": "cell",
"layout": {
"padding": {"signal": "columnPadding"},
"bounds": "flush",
"align": "each"
},
"encode": {
"enter": {
"x": {"signal": "0"},
"stroke": {"value": "transparent"},
"width": {"signal": "columnsWidth"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "group",
"name": "taskColumnWidth",
"style": "cell",
"title": {
"text": "Task",
"anchor": "start",
"frame": "group",
"align": "left",
"dx": 16
},
"encode": {
"enter": {
"stroke": {"value": "transparent"},
"width": {"signal": "taskColumnWidth"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "left"},
"dx": {"value": 16},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {
"signal": "datum.phase == datum.task ? upper(datum.task) : datum.task"
},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"limit": {"signal": "taskColumnWidth - 16"},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
}
}
}
},
{
"type": "symbol",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"fill": {
"signal": "toString(datum.id) == itemHovered.id && datum.phase == datum.task ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : datum.phase == datum.task ? scale('cDark', datum.phase) : 'transparent'"
},
"x": {"value": 10},
"size": {"value": 90},
"yc": {"signal": "(scale('y', datum.id) + bandwidth('y') / 2) - 1"},
"shape": {
"signal": "datum.phase == datum.task && !indata('collapsedPhases', 'phase', datum.phase) ? 'triangle-down' : datum.phase == datum.task && indata('collapsedPhases', 'phase', datum.phase) ? 'triangle-right' : ''"
}
}
}
}
]
},
{
"type": "group",
"name": "ownerColumnWidth",
"style": "cell",
"title": {
"text": "Owner",
"anchor": "start",
"frame": "group",
"align": "left",
"dx": 16
},
"encode": {
"enter": {
"stroke": {"value": "transparent"},
"width": {"signal": "ownerColumnWidth"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "left"},
"dx": {"value": 16},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"field": "owner"},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"limit": {"signal": "ownerColumnWidth - 16"},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
}
}
}
}
]
},
{
"type": "group",
"name": "startColumnWidth",
"style": "cell",
"title": {
"text": "Start",
"anchor": "end",
"frame": "group",
"align": "right"
},
"encode": {
"update": {
"width": {"signal": "startColumnWidth"},
"height": {"signal": "height"},
"stroke": {"value": "transparent"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "right"},
"x": {"signal": "startColumnWidth"},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"signal": "timeFormat(datum.start, ' %d/%m/%y')"},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
}
}
}
}
]
},
{
"type": "group",
"name": "endColumnWidth",
"style": "cell",
"title": {
"text": "End",
"anchor": "end",
"frame": "group",
"align": "right"
},
"encode": {
"update": {
"width": {"signal": "endColumnWidth"},
"stroke": {"value": "transparent"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "right"},
"x": {"signal": "endColumnWidth"},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"signal": "timeFormat(datum.labelEnd, ' %d/%m/%y')"},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
}
}
}
}
]
},
{
"type": "group",
"name": "statusColumnWidth",
"style": "cell",
"title": {
"text": "Status",
"anchor": "end",
"frame": "group",
"align": "right"
},
"encode": {
"update": {
"width": {"signal": "statusColumnWidth"},
"stroke": {"value": "transparent"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "right"},
"x": {"signal": "statusColumnWidth"},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"field": "status"},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"fill": {
"signal": "datum.status == 'Complete' ? '#2ca02c' : datum.status == 'Late' ? '#d62728' : datum.status == 'On Schedule' ? '#ff7f0e' : '#9467bd'"
}
}
}
}
]
},
{
"type": "group",
"name": "daysColumnWidth",
"style": "cell",
"title": {
"text": "Days",
"anchor": "end",
"frame": "group",
"align": "right"
},
"encode": {
"update": {
"width": {"signal": "daysColumnWidth"},
"stroke": {"value": "transparent"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "text",
"style": "col",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "right"},
"x": {"signal": "daysColumnWidth"},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"signal": "datum.days + ' d'"},
"fontWeight": {
"signal": "datum.phase == datum.task ? 'bold' : 'normal'"
},
"font": {
"signal": "datum.phase == datum.task ? 'Arial' : 'Segoe UI'"
},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
}
}
}
}
]
},
{
"type": "group",
"name": "completionColumn",
"style": "cell",
"title": {"text": "Progress", "anchor": "start", "frame": "group"},
"encode": {
"update": {
"width": {"signal": "progressColumnWidth"},
"stroke": {"value": "transparent"},
"height": {"signal": "height"}
}
},
"marks": [
{
"type": "rect",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"x": {"signal": "1"},
"width": {"signal": "item.mark.group.width - 2"},
"stroke": {"signal": "'#a0d786'"},
"yc": {"signal": "(scale('y', datum.id) + bandwidth('y') / 2)"},
"fill": {"value": "white"},
"height": {"signal": "bandwidth('y') - yPaddingInner * 2"},
"strokeWidth": {"signal": "datum.id == itemHovered.id ? 2 : 1"}
}
}
},
{
"type": "rect",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"x": {"signal": "1"},
"width": {
"signal": "(item.mark.group.width / 100) * datum.completion"
},
"fill": {"signal": "'#81c784'"},
"yc": {"signal": "(scale('y', datum.id) + bandwidth('y') / 2)"},
"strokeWidth": {"value": 1},
"height": {"signal": "bandwidth('y') - yPaddingInner * 2"}
}
}
},
{
"type": "text",
"clip": true,
"from": {"data": "yScale"},
"encode": {
"update": {
"align": {"value": "left"},
"dx": {"value": 3},
"fill": {"signal": "textColour"},
"y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"text": {"signal": "datum.completion + '%'"}
}
}
}
]
}
]
},
{
"type": "group",
"name": "weekendContainer",
"encode": {
"update": {
"x": {"signal": "columnsWidth"},
"y": {"signal": "-15"},
"clip": {"signal": "true"},
"height": {"signal": "height + 15"},
"width": {"signal": "ganttWidth"},
"fill": {"value": "transparent"}
}
},
"marks": [
{
"type": "rect",
"description": "Weekend shading",
"name": "weekendShading",
"from": {"data": "weekends"},
"encode": {
"update": {
"x": {"signal": "scale('x', datum.start)"},
"x2": {"signal": "scale('x', datum.end)"},
"y": {"signal": "dayBandwidthRound >= minDayBandwidth ? 0 : 15"},
"y2": {"signal": "scaledHeight < height ? yRange[1] + 15 : height + 15"},
"strokeWidth": {"signal": "1"},
"stroke": {"value": "#f0f0f0"},
"fill": {"value": "#f0f0f0"}
}
}
},
{
"name": "todayHighlight",
"description": "Today highlight",
"type": "rect",
"data": [{}],
"encode": {
"update": {
"x": {"signal": "scale('x', today)"},
"width": {"signal": "dayBandwidth"},
"y": {"value": 0},
"height": {"signal": "15"},
"fill": {"value": "#ffcc80"}
}
}
}
]
},
{
"type": "group",
"name": "ganttContainer",
"encode": {
"update": {
"x": {"signal": "columnsWidth"},
"y": {"signal": "0"},
"clip": {"signal": "true"},
"height": {"signal": "height"},
"width": {"signal": "ganttWidth"},
"fill": {"value": "transparent"}
}
},
"marks": [
{
"name": "completionLabelSizes",
"type": "text",
"from": {"data": "tasks"},
"encode": {
"enter": {
"fill": {"value": "transparent"},
"text": {"signal": "datum.completionLabel"}
}
}
},
{
"name": "taskLabels",
"description": "Task, milestone and phase names",
"from": {"data": "yScale"},
"type": "text",
"encode": {
"update": {
"x": {"scale": "x", "field": "end"},
"align": {"value": "left"},
"dx": {
"signal": "datum.milestone ? sqrt(milestoneSymbolSize) / 2 - dayBandwidth / 2 + 5 : 5"
},
"y": {
"signal": "datum.phase == datum.task ? scale('y', datum.id) - 2 : scale('y', datum.id)"
},
"dy": {"signal": "bandwidth('y') / 2"},
"fill": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : textColour"
},
"text": {
"signal": "datum.milestone ? datum.task : datum.task && datum.owner ? datum.task + ' (' + datum.days + ' d' + ') - ' + datum.owner : datum.task + ' (' + datum.days + ' d' + ')'"
}
}
}
},
{
"type": "group",
"from": {
"facet": {
"name": "dependencyLinesFacet",
"data": "dependencyLines",
"groupby": ["id", "sourceId"]
}
},
"marks": [
{
"type": "line",
"from": {"data": "dependencyLinesFacet"},
"encode": {
"enter": {
"x": {"signal": "datum.value[0]"},
"y": {"signal": "datum.value[1]"},
"stroke": {"value": "#b0bec5"},
"strokeWidth": {"value": 1},
"interpolate": {"value": "linear"},
"strokeJoin": {"value": "bevel"},
"strokeCap": {"value": "round"},
"defined": {"value": true}
},
"update": {
"stroke": {
"signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : '#b0bec5'"
},
"strokeWidth": {"signal": "datum.id == itemHovered.id ? 1.5 : 1"}
}
}
}
]
},
{
"name": "todayRule",
"description": "Today rule",
"type": "rule",
"data": [{}],
"encode": {
"update": {
"x": {"signal": "scale('x', today + oneDay / 2)"},
"y2": {"signal": "scaledHeight < height ? yRange[1] : height"},
"strokeWidth": {"value": 1},
"stroke": {"value": "#1f77b4"},
"strokeDash": {"value": [2, 2]},
"opacity": {"value": 0.8}
}
}
},
{
"name": "todayText",
"description": "Today text",
"type": "text",
"data": [{}],
"encode": {
"update": {
"x": {"signal": "scale('x', today + oneDay / 2)"},
"fill": {"value": "#1f77b4"},
"text": {"value": "Today"},
"angle": {"signal": "90"},
"baseline": {"value": "bottom"},
"dx": {"value": 10},
"dy": {"value": -4},
"opacity": {"value": 0.7}
}
}
},
{
"name": "taskBars",
"description": "The task bars (serve as an outline for percent complete)",
"type": "group",
"from": {"data": "tasks"},
"encode": {
"update": {
"clip": {"signal": "true"},
"x": {"scale": "x", "field": "start"},
"x2": {"scale": "x", "field": "end"},
"yc": {"signal": "(scale('y', datum.id) + bandwidth('y') / 2)"},
"height": {"signal": "bandwidth('y') - yPaddingInner _ 2"},
"tooltip": {
"signal": "showTooltips && down == null ? {'Phase': datum.phase, 'Task': datum.task, 'Owner': datum.owner, 'Status': datum.status, 'Start': timeFormat(datum.start, '%a, %d %B %Y'), 'End': timeFormat(datum.labelEnd, '%a, %d %B %Y'), 'Days': datum.days, 'Progress': datum.completionLabel} : null"
},
"fill": {
"signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cLight', datum.phase)), {l:0.65}) : scale('cLight', datum.phase)"
},
"stroke": {
"signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : scale('cDark', datum.phase)"
},
"cornerRadius": {"value": 5},
"zindex": {"value": 101},
"strokeWidth": {
"signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? 1.5 : 1"
},
"href": {"field": "hyperlink"}
}
},
"transform": [
{
"type": "lookup",
"from": "completionLabelSizes",
"key": "datum.id",
"fields": ["datum.id"],
"values": ["bounds.x1", "bounds.x2"],
"as": ["a", "b"]
}
],
"marks": [
{
"name": "taskFills",
"description": "Percent complete for each task",
"type": "rect",
"interactive": false,
"encode": {
"update": {
"x": {"signal": "0"},
"y": {"signal": "0"},
"height": {"signal": "item.mark.group.height"},
"width": {
"signal": "(item.mark.group.width / 100) _ item.mark.group.datum.completion"
},
"fill": {
"signal": "toString(item.mark.group.datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(item.mark.group.datum.id)) > -1 ? merge(hsl(scale('cDark', item.mark.group.datum.phase)), {l:0.40}) : scale('cDark', item.mark.group.datum.phase)"
},
"strokeWidth": {"value": 0},
"cornerRadiusBottomLeft": {"value": 5},
"cornerRadiusTopLeft": {"value": 5}
}
}
},
{
"name": "completeText",
"description": "Completion Text",
"type": "text",
"interactive": false,
"encode": {
"update": {
"x": {
"signal": "(item.mark.group.width / 100) _ parent.completion"
},
"align": {"value": "right"},
"dx": {"signal": "-2"},
"baseline": {"value": "middle"},
"y": {"signal": "item.mark.group.height / 2 + 1"},
"text": {
"signal": "round(((item.mark.group.width / 100) _ parent.completion)) >= (item.mark.group.b + 4) && parent.completion > 0 ? parent.completionLabel : ''"
},
"fill": {
"signal": "luminance(item.mark.group.stroke) >= 0.45 ? 'black' : 'white'"
}
}
}
}
]
},
{
"name": "phaseOutline",
"description": "The phase bar outlines",
"type": "path",
"from": {"data": "phasePaths"},
"encode": {
"update": {
"path": {"signal": "datum.phasePath"},
"fill": {
"signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cLight', datum.phase)), {l:0.65}) : scale('cLight', datum.phase)"
},
"stroke": {
"signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : scale('cDark', datum.phase)"
},
"strokeWidth": {"signal": "datum.id == itemHovered.id ? 1.5 : 1"},
"tooltip": {
"signal": "showTooltips && down == null ? {'Phase': datum.phase, 'Start': timeFormat(datum.start, '%a, %d %B %Y'), 'End': timeFormat(datum.labelEnd, '%a, %d %B %Y'), 'Days': datum.days, 'Progress': datum.completion + '%'} : null"
},
"cursor": {"value": "pointer"}
}
}
},
{
"name": "phaseGroup",
"description": "Group to hold the x y coordinates for the SVG clipping fills",
"type": "group",
"clip": true,
"from": {"data": "phasePaths"},
"encode": {
"update": {
"strokeWidth": {"value": 0},
"stroke": {"value": "red"},
"x": {"scale": "x", "field": "start", "offset": 0},
"x2": {"scale": "x", "field": "end", "offset": 0},
"yc": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
"height": {"signal": "bandwidth('y') - yPaddingInner \* 2"}
}
},
```
## Marks for Visual Representation

Marks are the visual elements (e.g., rectangles, lines, symbols) that represent data in the Gantt chart.

Task Bars: Rectangles representing tasks, with properties like start/end positions determined by the x scale. The width of each bar corresponds to the task duration. Additional properties like fill color or stroke can reflect task status or dependencies.
Milestone Symbols: Milestones are typically represented as symbols (e.g., diamonds) at specific points in the timeline. They may include hover tooltips for detailed information.
Phase Outlines: Marks that visually group related tasks within a phase. These may be outlined rectangles or paths that represent the entire span of a phase.
Dependency Arrows: Lines or paths connecting tasks to indicate dependencies, allowing users to visualize task relationships. \*/

          "marks": [
            {
              "name": "phaseFills",
              "description": "Percent complete for each phase. Clipping path signal has to be here as it fails to update on zoom when coming from a dataset. The only value available in the clipping path signal is parent!",
              "type": "rect",
              "interactive": false,
              "clip": {
                "path": {
                  "signal": "'M 0 0' + ' L ' + (scale('x', parent.end) - scale('x', parent.start)) + ' 0' + ' v ' + phaseSymbolHeight + ' L ' + (scale('x', parent.end) - phaseSymbolWidth) + ' ' + (phaseSymbolHeight / 2) + ' H ' + phaseSymbolWidth + ' L 0 ' + phaseSymbolHeight + ' z'"
                }
              },
              "encode": {
                "update": {
                  "height": {"signal": "phaseSymbolHeight"},
                  "width": {
                    "signal": "(item.mark.group.width / 100) * item.mark.group.datum.completion"
                  },
                  "fill": {
                    "signal": "toString(item.mark.group.datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(item.mark.group.datum.id)) > -1 ? merge(hsl(scale('cDark', item.mark.group.datum.phase)), {l:0.40}) : scale('cDark', item.mark.group.datum.phase)"
                  },
                  "strokeWidth": {"value": 0},
                  "stroke": {"value": "red"}
                }
              }
            }
          ]
        },
        {
          "name": "milestoneSymbols",
          "description": "Milestones",
          "type": "symbol",
          "from": {"data": "milestones"},
          "encode": {
            "update": {
              "x": {"signal": "scale('x', datum.start) + dayBandwidth / 2"},
              "y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
              "size": {"signal": "milestoneSymbolSize"},
              "shape": {"value": "diamond"},
              "fill": {
                "signal": "(toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1) && datum.completion > 0 ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cLight', datum.phase)), {l:0.65}) : datum.completion > 0 ? scale('cDark', datum.phase) : scale('cLight', datum.phase)"
              },
              "stroke": {
                "signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : scale('cDark', datum.phase)"
              },
              "strokeWidth": {
                "signal": "toString(datum.id) == itemHovered.id || indexof(itemHovered.dependencies, toString(datum.id)) > -1 ? 1.5 : 1"
              },
              "tooltip": {
                "signal": "showTooltips && down == null ? {'Phase': datum.phase, 'Task': datum.task, 'Owner': datum.owner, 'Status': datum.status, 'Start': timeFormat(datum.start, '%a, %d %B %Y'), 'End': timeFormat(datum.labelEnd, '%a, %d %B %Y'), 'Days': datum.days, 'Progress': datum.completionLabel} : null"
              }
            }
          }
        },
        {
          "name": "taskDependencyArrowsymbol",
          "description": "Dependency arrows",
          "type": "symbol",
          "from": {"data": "dependencyArrows"},
          "encode": {
            "update": {
              "shape": {"value": "triangle-right"},
              "x": {
                "signal": "scale('x', datum.start)",
                "offset": {
                  "signal": "datum.milestone != null && datum.milestone != false ? -(sqrt(arrowSymbolSize)) / 2 + dayBandwidth / 2 - (sqrt(milestoneSymbolSize)) / 2 + 1 : -(sqrt(arrowSymbolSize) / 2) + 1"
                }
              },
              "y": {"signal": "scale('y', datum.id) + bandwidth('y') / 2"},
              "fill": {
                "signal": "toString(datum.id) == itemHovered.id ? merge(hsl(scale('cDark', datum.phase)), {l:0.40}) : '#6a6a6a'"
              },
              "size": {"signal": "arrowSymbolSize"}
            }
          }
        }
      ]
    },
    {
      "name": "taskSelector",
      "description": "Hidden rect to support phase expand and collapse",
      "type": "rect",
      "clip": true,
      "zindex": 99,
      "from": {"data": "yScale"},
      "encode": {
        "update": {
          "x": {"value": -15},
          "x2": {"signal": "columnsWidth"},
          "y": {"signal": "scale('y', datum.id)"},
          "height": {"signal": "bandwidth('y')"},
          "fill": {"value": "transparent"},
          "cursor": {"signal": "datum.phase == datum.task ? 'pointer' : 'auto'"},
          "href": {"field": "hyperlink"}
        }
      }
    },
    {
      "type": "group",
      "name": "axisClipper",
      "style": "cell",
      "clip": true,
      "encode": {
        "enter": {
          "width": {"signal": "columnsWidth"},
          "stroke": {"value": "transparent"},
          "height": {"signal": "height"}
        }
      },
```

## Axes Section

Axes provide visual guides that help interpret the scales and data within the visualization.

X-Axis (Time): Displays dates or periods along the horizontal axis, with tick marks and labels. Depending on the zoom level or time granularity, labels may show days, months, or years.
Y-Axis (Tasks/Phases): Provides a label or guide for each task or phase, displayed vertically. Y-axis labels may be hidden or simplified depending on the chart's interactivity or layout constraints.
Grid Lines: Optionally displayed for guidance, typically aligned with ticks to help interpret data positions relative to the axes. \*/

      "axes": [
        {
          "scale": "y",
          "orient": "right",
          "encode": {"ticks": {"update": {"x2": {"signal": "-columnsWidth"}}}},
          "tickColor": "#f1f1f1",
          "labels": false,
          "title": "",
          "grid": false,
          "ticks": true,
          "bandPosition": {"signal": "0"}
        }
      ]
    }

],
"axes": [
{
"description": "Bottom date axis",
"ticks": true,
"labelPadding": -12,
"scale": "x",
"position": {"signal": "columnsWidth"},
"orient": "top",
"tickSize": 15,
"grid": false,
"zindex": 1,
"labelOverlap": false,
"formatType": "time",
"tickCount": {
"signal": "dayBandwidthRound >= minYearBandwidth ? 'day' : 'month'"
},
"encode": {
"ticks": {
"update": {
"strokeWidth": [
{"test": "dayBandwidthRound >= minDayBandwidth", "value": 1},
{
"test": "dayBandwidthRound >= minMonthBandwidth && dayBandwidthRound < minDayBandwidth && date(datum.value) == 1",
"value": 1
},
{
"test": "dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minMonthBandwidth && date(datum.value) == 1",
"value": 1
},
{
"test": "dayBandwidthRound < minYearBandwidth && dayofyear(datum.value) == 1",
"value": 1
},
{"value": 0}
]
}
},
"labels": {
"update": {
"text": [
{
"test": "dayBandwidthRound >= minDayBandwidth",
"signal": "timeFormat(datum.value, '%d')"
},
{
"test": "dayBandwidthRound >= minMonthBandwidth && dayBandwidthRound < minDayBandwidth && date(datum.value) == 15",
"signal": "timeFormat(datum.value, '%B %y')"
},
{
"test": "dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minMonthBandwidth && date(datum.value) == 15",
"signal": "timeFormat(datum.value, '%b')"
},
{
"test": "dayBandwidthRound < minYearBandwidth && month(datum.value) == 6",
"signal": "timeFormat(datum.value, '%Y')"
},
{"value": ""}
],
"dx": {"signal": "dayBandwidthRound / 2"}
}
}
}
},
{
"description": "Top date axis",
"scale": "x",
"position": {"signal": "columnsWidth"},
"domain": false,
"orient": "top",
"offset": 0,
"tickSize": 22,
"labelBaseline": "middle",
"grid": false,
"zindex": 0,
"tickCount": {
"signal": "dayBandwidthRound >= minYearBandwidth ? 'day' : 'month'"
},
"encode": {
"ticks": {
"update": {
"strokeWidth": [
{
"test": "dayBandwidthRound >= minDayBandwidth && date(datum.value) == 1",
"value": 1
},
{
"test": "dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minDayBandwidth && dayofyear(datum.value) == 1",
"value": 1
},
{"value": 0}
]
}
},
"labels": {
"update": {
"text": [
{
"test": "dayBandwidthRound >= minDayBandwidth && date(datum.value) == 15",
"signal": "timeFormat(datum.value, '%B %y')"
},
{
"test": "dayBandwidthRound >= minYearBandwidth && dayBandwidthRound < minMonthBandwidth && month(datum.value) == 5 && date(datum.value) == 15",
"signal": "timeFormat(datum.value, '%Y')"
},
{"value": ""}
],
"dx": {"signal": "dayBandwidthRound / 2"}
}
}
}
},
{
"description": "Month grid lines",
"scale": "x",
"position": {"signal": "columnsWidth"},
"domain": false,
"orient": "top",
"labels": false,
"grid": true,
"tickSize": 0,
"zindex": 0,
"tickCount": {
"signal": "dayBandwidthRound >= minMonthBandwidth || dayBandwidthRound <= 0.35 ? 0 : 'month'"
}
}
],```

## Scales Section

Scales map data values to visual properties like position, size, or color. In a Gantt chart, scales play a pivotal role in converting data (e.g., dates, task categories) to screen positions.

X Scale (Time): Maps the start and end dates of tasks to horizontal positions in the chart. It is typically a time scale and defines the domain (range of input dates) and the output range (pixels).
Y Scale (Band): Maps tasks or phases to vertical positions in a banded layout. This scale ensures that tasks are stacked in rows, with each task or phase occupying a specific vertical band.
Color Scales: These map phases or statuses to colors, enhancing visual distinction and readability. Color mappings can vary between "light" and "dark" ranges for aesthetic differentiation.

```

"scales": [
{
"name": "x",
"type": "time",
"domain": {"signal": "xDom"},
"range": {"signal": "[0, ganttWidth]"}
},
{
"name": "y",
"type": "band",
"domain": {
"fields": [{"data": "yScale", "field": "id"}],
"sort": {"op": "min", "field": "finalSort", "order": "ascending"}
},
"range": {"signal": "yRange"}
},
{
"name": "cDark",
"type": "ordinal",
"range": {"signal": "coloursDark"},
"domain": {"data": "input", "field": "phase"}
},
{
"name": "cLight",
"type": "ordinal",
"range": {"signal": "coloursLight"},
"domain": {"data": "input", "field": "phase"}
}
],
```

## Configuration Section

The configuration section defines global styles and settings for elements within the chart.

View Settings: Controls properties like background color or border strokes for the overall visualization container.
Font and Text Styles: Defines default font families, sizes, and text alignments for labels, tooltips, and other text elements.
Custom Styles: Named styles that can be applied to groups of marks or specific elements for consistent appearance across the visualization. \*/
"config": {
"view": {"stroke": "transparent"},
"style": {"col": {"fontSize": 11}, "cell": {"strokeWidth": {"value": "0"}}},
"font": "Segoe UI",
"text": {"font": "Segoe UI", "fontSize": 10, "baseline": "middle"},
"axis": {"labelColor": {"signal": "textColour"}, "labelFontSize": 10},
"title": {"color": {"signal": "textColour"}}
}
}
