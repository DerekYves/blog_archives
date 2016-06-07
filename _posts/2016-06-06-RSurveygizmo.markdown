---
layout: post
title:  "Rsurveygizmo: An R package for Interacting with the SurveyGizmo API"
author: "Derek Darves"
date:   2016-06-06 18:56:05 -0400
categories: R, Survey Research
---

Several years ago our team began using SurveyGizmo for our online surveys (and, actually, a bunch of other projects as well, from polls to data entry templates). At the time, SurveyGizmo provided 
a nice balance between cost and customization when compared to similar products from, e.g., Qualtrics and SurveyMonkey.
Over the years SurveyGizmo has greatly expanded the kinds of user customization and tweaking 
that is possible, particularly in the area of API calls.
Because we mostly work in R, I decided to write a package that accesses the 
SurveyGizmo API directly so that survey and email campaign data can 
be pulled directly with a project script 
(as opposed to manually downloading the data from the webpage).

Some usage examples for this package follow. To really test it out you will need to supply
your private SurveyGizmo API key and a valid survey id. There are many more function options 
outlined in the package help files than are presented below for those who'd like to learn more.


```r        
## Download a "regular" survey (that is, with no email campaign data), keeping only complete responses:
api <- "_yourapikeyhere_"

a_survey <- pullsg(1234, api, completes_only=T)

## Download all email campaign data for a particular survey:

a_campaign <- pullsg_campaign(surveyid, api) 

## Combine the previous steps in one function, that is, download email campaign data and merge it, 
## where possible, with the survey response object (this will only work, of course, when there are 
## valid email campaigns associated with a survey project).

a_survey <- pullsg(1234, api, mergecampaign=T)
```

If you'd like to give the package a spin, Rsurveygizmo can be installed in the usual way:

```r        
library(devtools)
install_github(repo="DerekYves/rsurveygizmo")
```
     

I hope this package is helpful to somebody, and feel free to drop me an email or post to the repository if you have any questions or suggestions for improvement!
