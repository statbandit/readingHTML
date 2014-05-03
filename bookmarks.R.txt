#####################
#Reading an HTML file
# In this case my firefox bookmarks export file
#
# Set the working directory
setwd("C:/to a local directory where file is located")
getwd()
list.files()

############################################
#This solution demonstrates a line by line approach with grep and regexpressions
htmlDoc <- readLines("bookmarks043014.html")
head(htmlDoc,15)

#Eliminate html lines that don't pertain to the bookmark folder name or the bookmark itself
#Use the value=TRUE to return the actual value otherwise it returns row number
bookmarkhtmlDoc<-grep("<DT><H3 ADD_DATE=(.*?) LAST_MODIFIED=(.*?)>(.*?)</H3>|*.<DT><A HREF=\"http(.*)>(.*)</A>",
                      htmlDoc,value=TRUE)


#Here's the required grep commands to extract the lines with folder names
#Bookmark folder names
folderline<-"<DT><H3 ADD_DATE=(.*?) LAST_MODIFIED=(.*?)>(.*?)</H3>"
bookmarkfolderlines<-grep(folderline,bookmarkhtmlDoc,value=TRUE)

#Here's the required grep commands to extract the lines with bookmark information
#Bookmark links and description
bookmarkline<-"*.<DT><A HREF=\"http(.*)>(.*)</A>"
bookmarklines<-grep(bookmarkline,bookmarkhtmlDoc,value=TRUE)


###########################################
# Now the lines are separated let's extract the actual values we're after
# The Bookmark Folder Name
# The Bookmark HTML Address
# The Bookmark Description

#The following is the commands to extract just the value for bookmark folder names
#Use the regexec command to return the location of the text in the ()'s for each line of bookmarkfolders
foldernamelocation<-regexec("*.?<DT><H3 ADD_DATE=.*? LAST_MODIFIED=.*?>(.*)<",bookmarkfolderlines)
#Use the regmatches to return the location and value
bookmarkfoldermatches<-regmatches(bookmarkfolderlines, foldernamelocation)
#Put the values into their own list
bookmarkfoldernames <- sapply(bookmarkfoldermatches, function(x) x[2])

#Grab the HTML Address and Description
bookmarkexp<-regexec("*.?<DT><A HREF=\"(.*?)\" ADD.*?>(.*?)</A>",bookmarklines)
bookmarklocation<-regmatches(bookmarklines, bookmarkexp)
bookmarklinks <- sapply(bookmarklocation, function(x) x[2])
bookmarkdescriptions <- sapply(bookmarklocation, function(x) x[3])

#Now convert to table
bookmarks<-as.data.frame(cbind(bookmarklinks,bookmarkdescriptions))
names(bookmarks)<-c("BookMark","BookMarkDescription")


###########################################
#Now for the finishing touch
#Attach the folder name to each of the bookmark and description line

#Get the line numbers for each line with a folder name on it
#use the folderline expression from above but do not use the value=TRUE flag
folderlinenumbers<-grep(folderline,bookmarkhtmlDoc)

#concatenate the overal length of bookmarkhtmlDoc + 1 to the end of the list for ease of use in
#the for loop.
folderlinenumbers<-c(folderlinenumbers,length(bookmarkhtmlDoc)+1)

#Initilize folderlist then loop for length of bookmarkfolders
folderlist<-as.data.frame(cbind(list(),list()))
for (i in 1:length(bookmarkfoldernames)){
  templist<-cbind((folderlinenumbers[i]+1):(folderlinenumbers[i+1]-1),bookmarkfoldernames[i])
  folderlist<-rbind(folderlist,templist)
  }

#Rename the column headers
names(folderlist)<-c("LineNumber","BookmarkFolder")

#Merge the two tables into one complete table
bookmarktable<-cbind(folderlist$BookmarkFolder,bookmarks)

#Write the finished table out as CSV delimited file
write.table(bookmarktable,file="bookmarktable.csv",sep=",",row.names=FALSE)


