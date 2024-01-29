---
title: Mood & Habits Tracker on Obsidian
tags:
  - analytics
  - mood
  - habits
  - obsidian
date: 2024-01-29
description: How to track your daily mood and habits using obsidian.
icon: 📅
---
> [!warning] Work in progress

---
## Intro

With the new year, I wondered if it could be interesting to have data on my mood or activities throughout the year, whether it's related to sports, nutrition or other aspects of life. A tracking tool would offer several benefits, such as:
- **Self-awareness:** Regular tracking could foster a better self-awareness, identifying factors that positively or negatively contribute to my overall well-being.
- **Analyzing trends:** Monitoring data over time could help identify trends in my mood or habits, providing a better understanding of my daily life.
- **Motivation:** Through regular tracking, I could stay motivated and committed to pursuing long-term goals, and so on.


The application I desired had to be user-friendly, prioritize my privacy, and present data in a visually appealing manner. I have tested several applications but didn't find anything convincing.

Then I thought, why not gather my notes and tracking information together?

In this post, I'll show you how to setup [Obsidian](https://obsidian.md/) - my daily note tool - to create a simple mood and habit tracker. The idea is to create the simplest and most efficient method for data input and then display the data graphically with a heat-map.

![[11-demo.png]]

---
## Prerequisite

- [Obsidian]() with add-ons
	- [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
	- [Heatmap Calendar](https://github.com/Richardsl/heatmap-calendar-obsidian)

---
## Setup

### Plugins

Install the two necessary plugins: [Dataview](https://github.com/blacksmithgu/obsidian-dataview) and [Heatmap Calendar](https://github.com/Richardsl/heatmap-calendar-obsidian).

In addition to the default configuration, make sure to enable the `Dataview` -> `Enable Javascript Queries` option. This is required for Heatmap to fetch data accurately.

### Template

Create a folder `templates` in your vault and add a file `tracker-template` with the content:

```md
# {{date}}

---

## Mood

- [ ] 😢
- [ ] 😕
- [ ] 😐
- [ ] 😊
- [ ] 😁

```

This template can be used to create a new entry and will be consumed by the plugins to create the heat-map graph.

> [!note] Note
> To utilize the template, ensure that the Obsidian setting `Template` -> `Template folder location` corresponds to the correct folder.

### Create the Graph

In another file, such as `index`, create the initial graph as illustrated below.


````md
```dataviewjs
// title
dv.span("**🙌 Mood Tracker**")

// map the different emoji with a integer value
const moodMapping = {"😢": 1, "😕": 2, "😐": 3, "😊": 4, "😁": 5}

// calendar configuration
const calendarData = {

	// scaling mood is from 1 to 5
	intensityScaleStart: 1,
	intensityScaleEnd: 5,

	// colors that will be used to render the heatmap
	colors: {
		redToGreen: ["#FD6C6E","#ED998F","#999999","#93C765","#4AB275"]
	},

	// entries that will be populated by the for loop below
	entries: []
}

// for loop to populate entries
// we will look for all the pages of the folder "tracking"
// that have checked tasks in the "Mood" section
for(let page of dv.pages('"tracking"').file.tasks.where(p=>p.checked).where(p=>String(p.section).includes("Mood"))){

	// get the date from the filename
	const date = page.path.split("/")[1].replace(".md", "")

	// push the datas to the entries array
    calendarData.entries.push({
        date,
		intensity: moodMapping[page.text],
		content: await dv.span(`[](${date})`)
    })
}

// render the heatmap
renderHeatmapCalendar(this.container, calendarData)
```
````

At this stage, you should see an empty heatmap.

>[!note] Note
>This script will scan all the files in the `tracking` folder, and it assumes that the filename corresponds to the date. Check that the Obsidian configuration `Daily notes` -> `New file location` is set to the `tracking` folder. This ensures that creating daily notes with the Obsidian shortcut will automatically integrate them into the graph.

### Create a Daily Entry

Run the `Open today's daily note` command or click on the ribbon to create the daily note.

![[11-open-ribbon.png]]

Then, apply the `tracker-template` to the file running the command `Templates: Insert template`.

![[11-insert-template.png|500]]

Now you can select your mood by selecting the corresponding emoji.

![[11-select-mood.png]]

And tadaa the heatmap should now display the mood of the day.

![[11-first-mood.png]]

---
## Styling

You have the flexibility to adjust the style of the heatmap for a more compact rendering. To do this, create a snippet `.obsidian/snippets/custom-heatmap.css` with the code below.

```css
.heatmap-calendar-boxes {
    padding: 5px !important;
    margin: 0 !important;
}

.heatmap-calendar-months {
    padding-inline-start: 1em !important;
    margin-block-start: 0 !important;
    margin-block-end: 0 !important;
}

.heatmap-calendar-days, .heatmap-calendar-boxes {
    grid-template-rows: repeat(7, minmax(0, 1em));
}

.heatmap-calendar-days {
    padding-inline-start: 0 !important;
    margin-block-start: 0 !important;
}

.heatmap-calendar-boxes > li {
    min-width: 10px;
    min-height: 10px;
}

.heatmap-calendar-graph {
    margin-top: 2em;
}

```

Next, activate the snippet in the Obsidian settings under `Appearance` -> `CSS snippet`.

![[11-heatmap-compact.png]]

Your heatmap should now showcase a distinct style! 🚀

---
## Ressources

- Photo by [Prophsee Journals](https://unsplash.com/@prophsee?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/white-book-WI30grRfBnE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)