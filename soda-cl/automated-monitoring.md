---
layout: default
title: Automated monitoring checks
description: Use a SodaCL automated monitoring check to automatically check for row count anomalies and schema changes.
parent: SodaCL
---

# Automated monitoring checks 
<!--Linked to UI, access Shlink-->
*Last modified on {% last_modified_at %}*

Use automated monitoring checks to instruct Soda to automatically check for row count anomalies and schema changes in a dataset.<br />
{% include code-header.html %}
```yaml
automated monitoring:
  datasets:
    - include %
    - exclude test%
```
[About automated monitoring checks](#about-automated-monitoring-checks)<br />
[Prerequisites](#prerequisites)<br />
[Define an automated monitoring check](#define-an-automated-monitoring-check)<br />
[Optional check configurations](#optional-check-configurations) <br />
[Go further](#go-further) <br />
<br />

## About automated monitoring checks

When you add automated monitoring checks to your checks.yml file, Soda Library prepares and executes two checks on all the datasets you indicate as included in your checks YAML file. 

**Anomaly score check on row count**: This check counts the number of rows in a dataset during scan and registers anomalous counts relative to previous measurements for the row count metric. Refer to [Anomaly score checks]({% link soda-cl/anomaly-score.md %}) for details. <br />
Anomaly score checks require a minimum of four data points (four scans at stable intervals) to establish a baseline against which to gauge anomalies. If you do not see check results immediately, allow Soda Library to accumulate the necessary data points. 

**Schema checks**: This check monitors schema changes in datasets, including column addition, deletion, data type changes, and index changes. By default, this automated check results in a failure if a column is deleted, its type changes, or its index changes; it results in a warning if a column is added. Refer to [Schema checks]({% link soda-cl/schema.md %}) for details.<br />
Schema checks require a minimum of one data point to use as a baseline against which to gauge schema changes. If you do not see check results immediately, wait until after you have scanned the dataset twice.

These types of checks require a **Soda Cloud** account. Soda Library pushes check results to your account where Soda Cloud stores all the previously-measured, historic values for your checks in the Cloud Metric Store. SodaCL can then use these stored values to establish a relative state against which to evaluate future schema and anomaly score checks. 

## Prerequisites

<div class="warpper">
  <input class="radio" id="one" name="group" type="radio" checked>
  <input class="radio" id="two" name="group" type="radio">
  <div class="tabs">
  <label class="tab" id="one-tab" for="one">Configure in Soda Cloud</label>
  <label class="tab" id="two-tab" for="two">Configure using Soda Library </label>
    </div>
  <div class="panels">
  <div class="panel" id="one-panel" markdown="1">

* You have <a href="https://cloud.soda.io/signup?utm_source=docs" target="_blank">signed up for a Soda Cloud account</a>.
* You have [Administrator rights]({% link soda-cloud/roles-and-rights.md %}) within your organization's Soda Cloud account.
* You, or an Administrator in your organization's Soda Cloud account, has [deployed a Soda Agent]({% link soda-agent/deploy.md %}) which enables you to connect to a data source in Soda Cloud.


To define automated monitoring checks, follow the guided steps to [create a new data source]({% link soda-cloud/add-datasource.md %}#5-check-datasets). Reference the [section below](#define-an-automated-monitoring-check) for how to define the checks themselves. 

  </div>
  <div class="panel" id="two-panel" markdown="1">


* You have installed a [Soda Library package]({% link soda-library/install.md %}) in your environment.
* You have [configured Soda Library]({% link soda-library/configure.md %}) to connect to a data source using a `configuration.yml` file. 
* You have created and [connected a Soda Cloud account]({% link soda-library/configure.md %}) to Soda Library.
* You have [installed Soda Scientific](#install-soda-scientific) in the same directory or virtual environment in which you installed Soda Library; see instructions below.

## Install Soda Scientific

To use automated monitoring, you must install Soda Scientific in the same directory or virtual environment in which you installed Soda Library.

{% include install-soda-scientific.md %}

Refer to [Troubleshoot Soda Scientific installation](#troubleshoot-soda-scientific-installation) for help with issues during installation.


## Troubleshoot Soda Scientific installation

While installing Soda Scientific works on Linux, you may encounter issues if you install Soda Scientific on Mac OS (particularly, machines with the M1 ARM-based processor) or any other operating system. If that is the case, consider using one of the following alternative installation procedures.

* [Install Soda Scientific locally](#install-soda-scientific-locally)
* [Use Docker to run Soda Library](#use-docker-to-run-soda-library)

Need help? Ask the team in the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.

### Install Soda Scientific Locally 

{% include install-soda-scientific.md %}

  </div>
  </div>
</div>

### Use Docker to run Soda Library

{% include docker-soda-library.md %}


## Define an automated monitoring check

In the context of [SodaCL check types]({% link soda-cl/metrics-and-checks.md %}#check-types), automated monitoring checks are unique. This check employs the `anomaly score` and `schema` checks, but is limited in its syntax variation, with only a couple of mutable parts to specify which datasets to automatically apply the anomaly and schema checks.

The example check below uses a wildcard character (`%`) to specify that Soda Library executes automated monitoring checks against all datasets with names that begin with `prod`, and *not* to execute the checks against any dataset with a name that begins with `test`.
{% include code-header.html %}
```yaml
automated monitoring:
  datasets:
    - include prod%
    - exclude test%
```

<br />

You can also specify individual datasets to include or exclude, as in the following example.
{% include code-header.html %}
```yaml
automated monitoring:
  datasets:
    - include orders
```

### Scan results in Soda Cloud

To review the check results for automated monitoring checks in Soda Cloud, navigate to the **Checks** dashboard to see the automated monitoring check results with an INSIGHT tag. 


## Optional check configurations

| Supported | Configuration | Documentation |
| :-: | ------------|---------------|
|   | Define a name for an automated monitoring check. |  - |
|   | Define alert configurations to specify warn and fail thresholds. | - |
|   | Apply an in-check filter to return results for a specific portion of the data in your dataset.| - | 
|   | Use quotes when identifying dataset names. | - |
| ✓ | Use wildcard characters ({% raw %} % {% endraw %} with dataset names in the check; see [example](#example-with-wildcards). | - |
|   | Use for each to apply anomaly score checks to multiple datasets in one scan. | - |
|   | Apply a dataset filter to partition data during a scan. |  -  |


#### Example with wildcards 
{% include code-header.html %}
```yaml
automated monitoring:
  datasets:
    - include prod%
    - exclude test%
```

## Go further

* Need help? Join the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
* Reference [tips and best practices for SodaCL]({% link soda/quick-start-sodacl.md %}#tips-and-best-practices-for-sodacl).
* Use a [freshness check]({% link soda-cl/freshness.md %}) to gauge how recently your data was captured.
* Use [reference checks]({% link soda-cl/reference.md %}) to compare the values of one column to another.

---

Was this documentation helpful?

<!-- LikeBtn.com BEGIN -->
<span class="likebtn-wrapper" data-theme="tick" data-i18n_like="Yes" data-ef_voting="grow" data-show_dislike_label="true" data-counter_zero_show="true" data-i18n_dislike="No"></span>
<script>(function(d,e,s){if(d.getElementById("likebtn_wjs"))return;a=d.createElement(e);m=d.getElementsByTagName(e)[0];a.async=1;a.id="likebtn_wjs";a.src=s;m.parentNode.insertBefore(a, m)})(document,"script","//w.likebtn.com/js/w/widget.js");</script>
<!-- LikeBtn.com END -->

{% include docs-footer.md %}
