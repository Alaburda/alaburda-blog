---
title: 3 Ways to Turbocharge Your Excel Team
author: Paulius Alaburda
date: '2021-03-09'
slug: []
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-03-09T22:15:45+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
output:
  distill::distill_article:
    self_contained: false
---

A lot of companies still have Excel teams working with data. It may seem like folks working in the Microsoft garden are missing out on the bees' knees of R, Python or Julia, but Excel itself is also under active development. To be honest, the way I used Excel 2 years ago is completely different from how I use it know. What's more, Excel has become part of a larger ecosystem of tools. I think every Excel team could adopt them to supercharge their work!

# 1. PowerQuery

I like to imagine Excel standing on three columns: the usual formulas, VBA and PowerQuery. While formulas are quick and dirty, nested formulas can become a drag to maintain quite quickly. VBA does everything until the one or two VBA gurus leave your team. I feel like PowerQuery strikes a great balance between the two: it provides more functionality than formulas, the steps to transform your data are recorded in one place and maintenance is much easier. PowerQuery is, essentially, a way to write a reproducible data transformation pipeline.

PowerQuery is based on the M language. It supports function definitions that you can call and comments are rendered alongside the pipeline. It doesn't have everything you would expect from a programming language but having everything in one place forces the user to separate the input data from the way it is handled. This is incredibly useful whenever the data is replaced because your scripted logic stays intact.

Finally, Power BI runs the same M language under the hood whenever it imports data. One immediate benefit is that you can copy PowerQuery code from Excel to Power BI and __it just works__. But that also means that you can establish a common language between your Excel teams and your Business Intelligence teams. 

# 2. Adopt Power Automate to reduce manual work

Microsoft seems to be on a course to bring PowerQuery, Power BI and Power Automate into a unified ecosystem. There are quite a few differences currently but Power Automate still has a ton of advantages. If you are an Excel team, chances are you may be dealing with manual work. It may be downloading Excel attachments, copy/pasting data or downloading responses from Microsoft Forms. Power Automate is the glue that can bring data over to your other tools.

Power Automate is low-code/no-code application that can run pipelines on a schedule or based on a trigger. For example, we receive daily emails with an attached Excel file. Power Automate can download the attachment into a folder. Then PowerQuery can pull all the files from the folder into a single dataset for later use.

Power Automate expands its integrations quite fast. Chances are you will find possible integrations across your stack. I was surprised to find you can pull responses from a submission form, munge them into a JSON object and push them through the Jira API as a user story. You may need a server instance for some of the tasks but for others, Power Automate is definitely a great tool.

# 3. Adopt Tables!

Not like from Ikea, but adopt working with Excel tables. If Power Query and Power Automate are too "out there", I would encourage everyone to start formattng their datasets as tables. There are two key advantages to this.

First, a table can be thought of as a sheet in a sheet. Whenever you try to query data from an Excel file, you can either query the whole sheet, or query a table. Querying a sheet can sometimes backfire, e.g. we had a formatting issue which forced PowerQUery to import over a million rows and 300 columns. A table has a clear definition and you can be sure you will import the same structure every time.

Second, instead of refering to cell ranges in Excel, you can refer to column names in the table. Suddenly, `=SUM(A1:A5)` becomes `=SUM([sales])`. This immediately benefits formula readability and maintenance because instead of checking the referenced cell range, you can immediately grasp the intention of the formula.









