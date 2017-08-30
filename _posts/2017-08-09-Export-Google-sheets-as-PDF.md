---
layout: post
title:  Export Google sheets as PDF 
date:   2017-08-09 20:55:12 +0800
categories: 'Google Apps Script'
---

You can find the code [here](https://github.com/ycy88/exportGSheet).

The difficult part is not exporting to PDF. It is just one line: 

```javascript 
 var theBlob = theSpreadsheet.getBlob().getAs('application/pdf').setName(pdfName);
 ```

The difficulty is that .getBlob() is callable only on files, which will save them as PDFs. But this means that you export the **entire workbook**, and not the active spreadsheet which we want. 

The script gets around creating duplicate workbooks and deleting the extra sheets that you don't need. 

Limitations: As we use Google Sheets API, there are some functions that we can't customize. You will notice that exported PDFs have grid lines. They cannot be disabled through the API natively. A work around is to fetch the particular sheet using HTTP requests, as some Stackoverflow-ers have mentioned. 

Till next time!