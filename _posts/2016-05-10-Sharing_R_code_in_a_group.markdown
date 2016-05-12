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

1. A specific (hopefully source controlled) file is sourced at the top of each analysis script. At the point of sourcing some parameters are declared which do the correct thing depending on whether the host is Linux, Mac, or Windows. This is done in several steps. In the analysis script I include a few lines of code that allow all users to run the same exact file with the correct directory parameters loaded. Here’s an example:
        
        ## Load host-dependent directory environment
        winos <- ifelse(grepl("windows", Sys.info()['sysname'], ignore.case=T), 1, 0)
        if(winos==1) source("C:/data/projects/scripts/R/functions/file_dir_params.R")
        if(winos==0) source("~/projects/scripts/R/functions/file_dir_params.R")
        rm(winos, host)
     
2. The next step is to build your version of the file/directory parameter file that was sourced in step 1 by a script (in this post I'll call this file “file_dir_params.R”). 

        
        #--begin file_dir_params.R script--#
        
        # Make a new environment:
        fdirs <- new.env()
        
3. Run add this function to "file_dir_params.R", which is used to save a simple string indicating the host's OS type:

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
        

4. With our new environment loaded and knowledge of the computer's OS, we're ready to build platform agnostic variables that point to key shared directories (directories that are probably saved through pulling a remote GIT repository onto the local computer):
        
        ## Declare the root project and data directory:
        
        show  <- 1
        build <- 1
        
        if(grepl("windows", fdirs$computeros)==F){
        	fdirs$prjdir <- "~/projects/"
        	fdirs$prjdta <- "/your_data/"
          }else{
        	fdirs$prjdir <- "C:/projects/"
        	fdirs$prjdta <- "D:/your_data/"
        }
        
5. Depending on your setup you may have multiple project or data root folders you want to declare. 
To keep my life simple on my development machine, I try and make all data a sub-directory of prjdta 
and all scripts a subdirectory of prjdir. Once you have these set using the platform agnostic pattern
outlined in step 4, you're ready to build out all your subdirectories. Because every sub-directory
is a child of the root directories, make sure to always build new variables using the either 
(in my example) either prjdta or prjdir. This is the key to making code work across different platforms.
One added very useful bonus: if I move, say, my main data folder, I don’t have to rewrite 100 variable names 
in dozens of scripts. I just change the root data folder (in this example, “fdirs$prjdta”) once
and everything else takes care of itself. Here’s some examples:
        
        # Add some child objects to the fdirs environment:
        fdirs$dqrptsrc   <- paste0(fdirs$prjdta, "data_quality/source/")
        if(show==1) cat("\ndqrptsrc =", fdirs$dqrptsrc)
        if(make==1) system(paste0("mkdir -p ", fdirs$dqrptsrc))
        
        # Define some colors for (say) GGPlot using your organization's color codes:
        fdirs$com_ppt_orange <- "#FF6122"
        fdirs$com_cr_blue    <- "#30812E"
        fdirs$com_cr_red     <- "#D0212E"
        fdirs$com_cr_green   <- "#8EB126"
        

6. Notice how, in the lines above, I reference the "show" and "build" flags we declared near the top of the "file_dir_params.R" script.
What these do, respectively, is print the file location of the variable to standard output and build the folder if it does not exist. This can be
useful but is not strictly necessary.

7. Our last step is to attach the "fdirs" environment (or safely reload it if it's already attached):

        
        # Attach the new environment (and safely reload if already attached):
        while("fdirs" %in% search())
        detach("fdirs")
        attach(fdirs)
        

<br>

**Final Thoughts**

Attaching the environment as we did in step 7 is a great time saver because your can omit the "fdirs$" prefix in your scripts. For example, to load a file in my data
folder I now just write:

        x <- readRDS(paste0(dqrptsrc, "somefile.Rds"))

As opposed to:

        x <- readRDS(paste0(fdirs$dqrptsrc, "somefile.Rds"))

But as with everything, there is a draw back: you'll want to make sure the names of your variables in fdirs don't overlap with functions
or other items you declare in a script. For this reason I use expressions like "prjdta" rather than "data", and I avoid overly concise
constructions that are used in a lot of example code, e.g. "x", "y", or "z".

I hope some or all of the above is helpful to somebody, and feel free to drop me an email if you have any questions!
