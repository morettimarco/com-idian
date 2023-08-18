---
type: Joke
status: <% tp.system.suggester(["To Do", "New", "Ready"], ["To Do", "New", "Ready"],false,"Status") %>
rating: 0/5
creation: <% tp.file.creation_date("YYYY-MM-DD") %> 
duration:
---
<%*
const fileName = await tp.system.prompt("Name your joke","",false,false)
let cleanFileName = fileName.replace(/[^a-zA-Z0-9 ]/g, '')
let fileCreated = false;
let counter = "";

while (!fileCreated){
try {
	await tp.file.move("/Material/Jokes/Joke - "+cleanFileName+counter)
	fileCreated=true;
} catch (error) {
	counter++
	continue;
	}
}
%>
>[!Joke]+ <% fileName %>
><% tp.file.cursor(1) %>
>^Body


-----
### Precursors
```dataview 
list from [[]]
and !outgoing([[]])
where type = "Joke" or type = "Bridge" or type = "Intro" or type = "Closer" 
```

### Follow ups
- <% tp.file.cursor(2) %>

<%*
const dv = app.plugins.plugins.dataview.api
let allTags = Object.entries(app.metadataCache.getTags() )
   .sort( (a, b) => a[0].localeCompare(b[0]) ) // Sorted alphabetically
   // .sort( (a, b) => b[1] - a[1], "desc" ) // Sorted related to frequency

let selectMore = true
let selectedTags = []
while (selectMore) {
  let choice = await tp.system.suggester((t) => t[0] + "(" + t[1] + ")", allTags, false, "[Select more tags (ESC when finished)] - " + selectedTags.join(", "))
  if (!choice) {
    selectMore = false
  } else {
    selectedTags.push(choice[0])
    allTags = allTags.filter(tag => !tag[0].match(choice[0]))
  }
}

tR += selectedTags.join(" ") 
%>