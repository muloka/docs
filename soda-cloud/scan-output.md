---
layout: default
title: Scan output in Soda Cloud
description: Learn how to access data quality scan results in Soda Cloud.
parent: Soda Cloud
---

# Scan output in Soda Cloud
*Last modified on {% last_modified_at %}*

Soda Cloud displays the latest status of all of your checks in the **Checks** dashboard.

There two methods through which a check and its latest result appears on the dashboard.
* When you define checks in a [checks YAML file]({% link soda-library/how-library-works.md %}) and use Soda Library to run a scan, the checks and their latest results manifest in the **Checks** dashboard in Soda Cloud. 
* Any time Soda Cloud runs a scheduled scan of your data as part of an [agreement]({% link soda-cloud/agreements.md %}), it displays the checks and their latest results in the **Checks** dashboard.

Log in to view the **Checks** dashboard; each row in the table represents a check, and the icon indicates whether the result from the latest scan passed, warned, or failed.

![check-dashboard](/assets/images/check-dashboard.png){:height="700px" width="700px"}

[Check results](#check-results)<br />
[Scan failed](#scan-failed)<br />
[Sending scan results to Soda Cloud](#sending-scan-results-to-soda-cloud)<br />
&nbsp;&nbsp;&nbsp;&nbsp;[Troubleshoot](#troubleshoot)<br />
[Do not send scan results to Soda Cloud](#do-not-send-scan-results-to-soda-cloud)<br />
[Run an ad hoc scan](#run-an-ad-hoc-scan)<br />
[Go further](#go-further)<br />
<br />

## Check results

{% include scan-output.md %}

## Scan failed

Check results indicate whether check passed, warned, or failed during the scan. However, if a scan itself failed to complete successfully, Soda Cloud displays a warning message in the **Datasets** dashboard under the dataset for which scans have failed. Soda Cloud does not send an email or Slack notification when a scan fails.

![scan-failed](/assets/images/scan-failed.png){:height="550px" width="550px"}

<br />

## Sending scan results to Soda Cloud

Soda Library use a secure API to connect to Soda Cloud. When it completes a scan, Soda Library:
1. securely gains access to Soda Cloud,
2. pushes the results of any checks you configured in the checks YAML file to Soda Cloud.

![scan-with-cloud](/assets/images/scan-with-cloud.png){:height="350px" width="350px"}


#### Troubleshoot

**Problem:** When running a programmatic scan or a scan from the command-line, I get an error that reads `Error while executing Soda Cloud command response code: 400`.

**Solution:** While there may be several reasons Soda returns a 400 error, you can address the following which may resolve the issue:
* Upgrade to the [latest version]({% link soda-library/install.md %}#upgrade) of Soda Library.
* Confirm that all the checks in your checks YAML file identify a dataset against which to execute. For example, the following syntax yields a 400 error because the `checks:` does not identify a dataset.

```yaml
checks:
    - schema:
        warn:
            when schema changes: any
```

<br />

## Do not send scan results to Soda Cloud

When you execute a scan from the command-line using Soda Library, you can add `--local` option to the scan command to prevent Soda Library from sending check results and any other metadata to Soda Cloud. Learn more about [scan options]({% link soda-library/run-a-scan.md %}#add-scan-options)
{% include code-header.html %}
```shell
soda scan -d my_datasource -c configuration.yml checks.yml --local
```

## Run an ad hoc scan in Soda Cloud

{% include ad-hoc-scan.md %}

## Go further

* Learn more about [Soda products in general]({% link soda/product-overview.md %}) and how the work together to establish and maintain data reliability.
* Learn [How Soda Library works]({% link soda-library/how-library-works.md %}).
* Questions? Join the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
<br />

---

Was this documentation helpful?

<!-- LikeBtn.com BEGIN -->
<span class="likebtn-wrapper" data-theme="tick" data-i18n_like="Yes" data-ef_voting="grow" data-show_dislike_label="true" data-counter_zero_show="true" data-i18n_dislike="No"></span>
<script>(function(d,e,s){if(d.getElementById("likebtn_wjs"))return;a=d.createElement(e);m=d.getElementsByTagName(e)[0];a.async=1;a.id="likebtn_wjs";a.src=s;m.parentNode.insertBefore(a, m)})(document,"script","//w.likebtn.com/js/w/widget.js");</script>
<!-- LikeBtn.com END -->

{% include docs-footer.md %}