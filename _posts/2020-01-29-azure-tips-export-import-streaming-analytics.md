---
layout: post
title:  "Azure Tips: Export / Import Streaming Analytics Jobs"
author: Bernard Lim
date:   2020-01-29 00:01:12 +0530
category: azure
summary: Export / Import Streaming Analytics using Visual Studio Code
thumbnail: streaminganalytics.png
---

# Export / Import Streaming Analytics Job

## Exporting

Unlike other services, there is no 'Export Template' option within the portal as shown in an example below:

![export template](/assets/img/posts/2020-01-29-azure-tips-export-import-streaming-analytics/export-template-icon.png)

One way to export a Streaming Analytics job is through **Visual Studio Code**

1. Install **Azure Stream Analytics Tools** extension through the **Extensions** tab

![sa icon](/assets/img/posts/2020-01-29-azure-tips-export-import-streaming-analytics/sa-icon.png)

2. Click on the **Azure** tab. **Stream Analytics** extension should appear.

![sa extension](/assets/img/posts/2020-01-29-azure-tips-export-import-streaming-analytics/sa-extension.PNG)

3. Sign in to your Azure account and a list of existing Streaming Analytics jobs should appear.

4. Browse over the job you would like to export and click on the 'Download' button. The job template will not be saved to your local.

![sa jobs](/assets/img/posts/2020-01-29-azure-tips-export-import-streaming-analytics/sa-jobs.PNG)

## Importing / Submitting Jobs

1. Open the downloaded / exported Streaming Analytics job folder in Visual Studio

2. Click on the **.asaql** file

3. On the top of the file, there will be a **Submit to Azure** option. On click, here you can choose the subscription you would like to deploy the job to.

<img src="/assets/img/posts/2020-01-29-azure-tips-export-import-streaming-analytics/sa-submit.PNG" width="650px" />