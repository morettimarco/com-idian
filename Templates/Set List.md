<%*
// Prompt the user to enter the name of the venue where they are performing
const venueName = await tp.system.prompt("Where are you performing?", "", false, false);
// Store the file creation date in a variable
const fileCreationDate = tp.file.creation_date("YYYY-MM-DD");
// Sanitize the venue name by removing special characters if the venue name is null sets it to NoVenue
let cleanVenueName = venueName==null ? "NoVenue" :venueName.replace(/[^a-zA-Z0-9 ]/g, '');
// Initialize a flag to track if the file has been successfully created
let fileCreated = false;
// Initialize a counter to append to the filename if necessary
let counter = "";

// Loop until the file is successfully created
while (!fileCreated){
    try {
        // Attempt to move the file to the specified directory with the constructed name
        await tp.file.move("/Set Lists/" + cleanVenueName + counter + " - " + fileCreationDate);
        // If the move operation is successful, set fileCreated to true to exit the loop
        fileCreated = true;
    } catch (error) {
        // Log the error message to the console
        console.log(error);
        // Increment the counter to ensure a unique filename in the next attempt
        counter++;
        continue;
    }
}

// Function to retrieve tags from the file metadata
function getTags(file) { 
    const metadata = app.metadataCache.getFileCache(file); 
    return metadata.tags ? metadata.tags.map(tag => tag.tag) : []; 
}

// Function to retrieve the duration from the file metadata, if available
function getDuration(file) { 
    const metadata = app.metadataCache.getFileCache(file);
    if (metadata && metadata.frontmatter && metadata.frontmatter.duration)
        return parseFloat(metadata.frontmatter.duration); 
    else
        return 0;
}

// Filter and sort markdown files in the 'Jokes' folder
let allFiles = app.vault.getMarkdownFiles().filter(file => file.path.startsWith("Material")).sort((a, b) => { 
    // Custom sorting logic based on filename
    if (a.basename.toLowerCase().startsWith("intro")) return -1;
    if (b.basename.toLowerCase().startsWith("intro")) return 1;
    if (a.basename.toLowerCase().startsWith("closer")) return 1;
    if (b.basename.toLowerCase().startsWith("closer")) return -1;
    return a.basename.localeCompare(b.basename);
});

// Initialize variables for user selection process
let selectMore = true;
let selectedFiles = [];
let totalDuration = 0;
let jokesList = "";

// Loop to allow user to select files until they decide to stop
while (selectMore) {
    // Use the suggester to prompt the user to select a file based on certain properties
    let choice = await tp.system.suggester((item) => `${item.basename} (Tags: ${getTags(item).join(' ')}) (${getDuration(item)}min)`, allFiles, false, "[Select files (ESC when finished)] - " + totalDuration.toFixed(1)+"min");
    
    if (!choice) {
        // If the user presses ESC or closes the suggester, exit the loop
        selectMore = false;
    } else {
        // Add the selected file to the list of selected files
        selectedFiles.push(choice.basename);
        let cache = app.metadataCache.getFileCache(choice);
        if (cache && cache.frontmatter && cache.frontmatter.duration) {
            // Update the total duration with the duration of the selected file
            totalDuration += parseFloat(cache.frontmatter.duration);
        }
        // Filter out the chosen file from the list of all files for the next iteration
        allFiles = allFiles.filter(file => !file.path.match(choice.basename));
    } 
}

// Create the list of links based on the files the user selected
selectedFiles.forEach((selectedFile) => {
    jokesList += '\n - [[' + selectedFile + ']]' + ' ![[' + selectedFile + '#^Body]]';
});
_%>
---
date: 
time: 
venue: <% venueName %>
creation: <% tp.file.creation_date("YYYY-MM-DD") %>
duration: <% totalDuration.toFixed(1) %>
---
### Used jokes
<% jokesList %>

### Unused Jokes 
```dataview
list rows.file.link from !outgoing([[]])
where status = "New"
sort file.cdate
group by type
```