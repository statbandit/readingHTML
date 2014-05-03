##README

The file bookmarks.R was an excerise to read an HTML file and output a useable dataset.

For this exercise, I exported my bookmarks from Firefox into an HTML file.

* Read the file via the command readlines

* Used grep to create an expression for the lines to keep pertaining to folder names or bookmarks

* Used regexec and regmatches to extract the required information from each line

* Used a for loop to put the actual folder name for each bookmark

* Exported the file in CSV format