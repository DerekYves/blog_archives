---
layout: post
title:  "Sharing R code in a workgroup of Mac/Windows/Linux users"
date:   2016-05-10 18:56:05 -0400
categories: R
---

In recent years there’s been a great deal of interest in, and work toward, creating more “reproducible” statistical code. I think this is a fantastic development. Looking at code I wrote in the late 90’s and early oughts, it’s clear how much a lot of my work from that period would have benefited from the coding habits I’ve developed over the last five or so years.

Working in R, a simple way to make code portable is to locate all data, functions, and scripts within a single directory/subdirectory which gets declared at the head of script and is followed by a series of relative paths in the lines to follow. This is standard process for projects you might find on Github.

But this is not always possible. Working in a corporate environment with protected/sensitive client data, I’m often forced to separate code from scripts, specifically, to keep certain sensitive data isolated on a particular server. I’m also not allowed to have any PII containing data in my git repositories. While I could exclude certain directories or file types from Git using, e.g., a .gitignore file, I’ve found it’s easier to just keep data in one place and scripts/functions in another. One more wrinkle: I work with Windows users, so simple references like “/data” will have different meanings if the Windows user is working from, say, the “D:\” drive.

I’ve tried and discarded a number of approaches to create a workflow that allows the same master code to run on a variety of computers which may have data stored in different places. Along the way I’ve refined my approach and have, in the last year or so, arrived at a process that allows me to work with others and with different types of OS’s in a reliable manner that requires little ongoing maintenance.

An added advantage the approach I’ve developed: if I want to move ALL of my main data sets, say from /data to /var/data, I can do this by changing one line of code in one file. After this change, all of my hundreds of scripts and markdown reports gracefully adapt to the change. They adapt because they all depend upon the same file to set up “the lay of the land” before any analysis. This requires a little extra work up front but, I find, saves a lot of headaches down the road — especially when working with other data scientists.

In short, my process is basically this:

1. A specific, source controlled file is sourced at the top of each analysis script. At the point of sourcing some parameters are declared which do the correct thing depending on whether the host is Linux, Mac, or Windows. This is done in several steps. In the analysis script I include a few lines of code that allow all users to run the same exact file with the correct directory parameters loaded. Here’s an example:

### Load host-dependent directory environment
winos <- ifelse(grepl("windows", Sys.info()['sysname'], ignore.case=T), 1, 0)
if(winos==1) source("C:/data/projects/scripts/R/functions/file_dir_params.R")
if(winos==0) source("~/projects/scripts/R/functions/file_dir_params.R")
rm(winos, host)
###
Now, inside the sourced file (“file_dir_params.R”) I have a series of commands that allow the host to gracefully adapt to the exact same script regardless of the OS it’s running. Here’s an example:

# Function to standardize host OS name
get_os <- function(){
	sysinf <- Sys.info()
	if (!is.null(sysinf)){
		os <- sysinf['sysname']
		if (os == 'Darwin')
			os <- "osx"
	} else {
		os <- .Platform$OS.type
		if (grepl("^darwin", R.version$os))
			os <- "osx"
		if (grepl("linux-gnu", R.version$os))
			os <- "linux"
	}
	tolower(os)
}
fdirs$computeros <- get_os()
Since most people in my group use Mac/Linux, I then declare the key directories but overwrite this variable if the computer is Windows, e.g.:

## This is an example of a root project directory:
fdirs$prjdir <- "~/projects/"

# Now, change this variable if the computer runs Windows:
if(grepl("windows", fdirs$computeros) fdirs$prjdir <- "C:/projects/"
2. This sourced file also creates a new environment (I call mine “fdirs”) which, after creating multiple variables to describe things like directory locations and corporate color codes, is attached at the end of the script. This makes all the variables accessible within all of my scripts but doesn’t clutter the global environment panel of, say, RStudio. Here’s an example:

# Make a new environment:
fdirs <- new.env()

# Add some objects to this environment:
fdirs$incoming <- "~/projects/stats/incoming/"
fdirs$scripts <- "~/projects/scripts/"

# Define some colors for GGPlot using standard color codes in the company:

fdirs$com_ppt_orange <- "#FF6122"
fdirs$com_cr_blue <- "#30812E"
fdirs$com_cr_red <- "#D0212E"
fdirs$com_cr_green <- "#8EB126"
…
# Attach the new environment (and safely reload if already attached):
while("fdirs" %in% search())
detach("fdirs")
attach(fdirs)
3. All directory locations are created in a compound fashion. Therefore, if I move, say, my main data folder, I don’t have to rewrite 100 variable names in dozens of difference scripts. I just change the root folder (in this example, “fdirs$stat_data”). Here’s an example:

### Define the root data directory
fdirs$stat_data <- "/dataroot/"

# Now build data subdirectories "on top" off fdirs$stat_data
fdirs$dqrptsrc   <- paste0(fdirs$stat_data, "dq_reports/source/")
if(test==1) cat("\ndqrptsrc =", fdirs$dqrptsrc)
if(make==1) system(paste0("mkdir -p ", fdirs$dqrptsrc))

Notice how, in the the two lines above I also created flags to optionally build the data folder structure and pipe the output to STDOUT to alert the user to the local directory location implied by a variable.

