---
layout: default
title: Run a Soda Library scan
description:  Soda Library uses the input in the checks and configuration YAML file to prepare a scan that it runs against the data in a dataset.
parent: Soda Library
redirect_from:
- /soda-core/first-scan.html
- /soda-core/scan-reference.html
- /soda-core/scan-core.html
---

# Run a Soda Library scan 
*Last modified on {% last_modified_at %}*
{% include banner-core.md %}

A **scan** is a command that executes checks to extract information about data in a dataset. 
{% include code-header.html %}
```shell
soda scan -d postgres_retail_db  -c configuration.yml checks.yml
```

A **check** is a test that Soda Library performs when it scans a dataset in your data source. Soda Library uses the checks you define in the **checks YAML** file to prepare SQL queries that it runs against the data in a table. Soda Library can execute multiple checks against one or more datasets in a single scan. 

[Anatomy of a scan command](#anatomy-of-a-scan-command)<br />
[Variables](#variables)<br />
[Scan output](#scan-output)<br />
[Examine a scan's SQL queries](#examine-a-scans-sql-queries)<br />
[Programmatically use scan output](#programmatically-use-scan-output)<br />
[Configure the same scan to run in multiple environments](#configure-the-same-scan-to-run-in-multiple-environments)<br />
[Add scan options](#add-scan-options)<br />
[Troubleshoot](#troubleshoot)<br />
[Go further](#go-further) <br />
<br />

## Anatomy of a scan command

Each scan requires the following as input:

* the name of the data source that contains the dataset you wish to scan, identified using the `-d` option
* a `configuration.yml` file, which contains details about how Soda Library can connect to your data source, identified using the `-c` option
* a `checks.yml` file which contains the checks you write using SodaCL

Scan command:
{% include code-header.html %}
```shell
soda scan -d postgres_retail -c configuration.yml checks.yml
```
<br />

Note that you can use the `-c` option to include **multiple configuration YAML files** in one scan execution. Include the filepath of each YAML file if you stored them in a directory other than the one in which you installed Soda Library.
{% include code-header.html %}
```shell
soda scan -d postgres_retail -c other-directory/configuration.yml other-directory/checks.yml
```

<br />

You can also include **multiple checks YAML files** in one scan execution. Use multiple checks YAML files to execute different sets of checks during a single scan. 
{% include code-header.html %}
```shell
soda scan -d postgres_retail -c configuration.yml checks_stats1.yml checks_stats2.yml
```

<br />
Use the soda `soda scan --help` command to review options you can include to customize the scan. See also: [Add scan options](#add-scan-options).

## Variables

There are several ways you can use variables in checks, filters, and in your data source configuration to pass values at scan time; a few examples follow. 

Refer to the comprehensive [Filters and variables]({% link soda-cl/filters.md %}) documentation for details.
{% include code-header.html %}
```yaml
# In-check filter
checks for dim_employee:
  - max(vacation_hours) < 80:
      name: Too many vacation hours for US Sales
      filter: sales_territory_key = 11

# Dataset filter with variables
filter CUSTOMERS [daily]:
  where: TIMESTAMP '${ts_start}' <= "ts" AND "ts" < TIMESTAMP '${ts_end}'

checks for CUSTOMERS [daily]:
  - row_count = 6
  - missing(cat) = 2

# In-check variable 
checks for ${DATASET}:
  - invalid_count(last_name) = 0:
      valid length: 10 
```


## Scan output

{% include scan-output.md %}

Example output with a check that triggered a warning:
```shell
Soda Library 1.0.x
Scan summary:
1/1 check WARNED: 
    CUSTOMERS in postgres_retail
      schema [WARNED]
        missing_column_names = [sombrero]
        schema_measured = [geography_key, customer_alternate_key, title, first_name, last_name ...]
Only 1 warning. 0 failure. 0 errors. 0 pass.
```

Example output with a check that failed:
```shell
Soda Library 1.0.x
Scan summary:
1/1 check FAILED: 
    CUSTOMERS in postgres_retail
      freshness(full_date_alternate_key) < 3d [FAILED]
        max_column_timestamp: 2020-06-24 00:04:10+00:00
        max_column_timestamp_utc: 2020-06-24 00:04:10+00:00
        now_variable_name: NOW
        now_timestamp: 2022-03-10T16:30:12.608845
        now_timestamp_utc: 2022-03-10 16:30:12.608845+00:00
        freshness: 624 days, 16:26:02.608845
Oops! 1 failures. 0 warnings. 0 errors. 0 pass.

```

### Examine a scan's SQL queries

To examine the SQL queries that Soda Library prepares and executes as part of a scan, you can add the `-V` option to your `soda scan` command. This option prints the queries as part of the scan results.
{% include code-header.html %}
```shell
soda scan -d postgres_retail -c configuration.yml -V checks.yml
``` 

## Programmatically use scan output

Optionally, you can insert the output of Soda Library scans into your data orchestration tool such as Dagster, or Apache Airflow. 

You can save Soda Library scan results anywhere in your system; the `scan_result` object contains all the scan result information. To import the Soda Library library in Python so you can utilize the `Scan()` object, [install a Soda Library package]({% link soda-library/install.md %}), then use `from soda.scan import Scan`. Refer to [Define programmatic scans]({% link soda-library/programmatic.md %}) and [Test data in a pipeline]({% link soda/quick-start-prod.md %}) for details.


## Configure the same scan to run in multiple environments

{% include scan-multiple-envs.md %}

## Add scan options

When you run a scan in Soda Library, you can specify some options that modify the scan actions or output. Add one or more of the following options to a `soda scan` command.

| Option | Required | Description and examples |
| ------ | :------: | ---------------------- |
| `-c TEXT` or<br /> `--configuration TEXT` | ✓ | Use this option to specify the file path and file name for the configuration YAML file.|
| `-d TEXT` or<br /> `--data-source TEXT` |  ✓ | Use this option to specify the data source that contains the datasets you wish to scan.|
| `-l` or<br /> `--local` |  | Use this option to prevent Soda Library from pushing check results or any other metadata to Soda Cloud. |
| `-s TEXT` or<br /> `--scan-definition TEXT` |  | Use this option to provide a [scan definition name]({% link soda/glossary.md %}#scan-definition-name) so that Soda Cloud keeps check results from different environments (dev, prod, staging) separate. See [Configure a single scan to run in multiple environments]({% link soda-library/configure.md %}#configure-the-same-scan-to-run-in-multiple-environments).|
| `-srf` or <br /> `--scan-results-file TEXT` |  | Specify the file name and file path to which Soda Library sends a JSON file of the scan results. You can use this in addition to, or instead of, sending results to Soda Cloud. <br /> `soda scan -d adventureworks -c configuration.yml -srf test.json checks.yml`|
| `-t TEXT` or<br /> `--data-timestamp TEXT` |  | Placeholder, only. |
| `-T TEXT` or<br /> `--template TEXT` | Use this option to specify the file path and file name for a [templates YAML]({% link soda-cl/check-template.md %}) file.|
| `-v TEXT` or<br /> `--variable TEXT` |  | Replace `TEXT` with variables you wish to apply to the scan, such as a [filter for a date]({% link soda-cl/filters.md %}). Put single or double quotes around any value with spaces. <br />  `soda scan -d my_datasource -v start=2020-04-12 -c configuration.yml checks.yml` |
| `V` or <br /> `--verbose` |  | Return scan output in verbose mode to review query details. |


## Troubleshoot

**Problem:** When you run a scan, you get an error that reads, `Exception while exporting Span batch.`

**Solution:** Without an internet connection, Soda Library is unable to communicate with `soda.connect.io` to transmit anonymous usage statistics about the software. <br /> If you are using Soda Library offline, you can resolve the issue by setting `send_anonymous_usage_stats: false` in your `configuration.yml` file. Refer to [Soda Library usage statistics]({% link soda-library/usage-stats.md %}) for further details.

<br />

**Problem:** In a Windows environment, you see an error that reads `[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (ssl_c:997)`. 

**Short-term solution:** Use `pip install pip-system-certs` to temporarily resolve the issue. This install works to resolve the issue only on Windows machines where the Ops team installs all the certificates needed through Group Policy Objects, or similar. However, the fix is short-term because when you try to run this in a pipeline on another machine, the error will reappear.

**Short-term solution:** Contact your Operations or System Admin team to obtain the proxy certificate.

## Go further

* Consider completing the [Quick start for SodaCL]({% link soda/quick-start-sodacl.md %}) to learn how to write more checks for data quality.
* Need help? Join the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
<br />

---

Was this documentation helpful?

<!-- LikeBtn.com BEGIN -->
<span class="likebtn-wrapper" data-theme="tick" data-i18n_like="Yes" data-ef_voting="grow" data-show_dislike_label="true" data-counter_zero_show="true" data-i18n_dislike="No"></span>
<script>(function(d,e,s){if(d.getElementById("likebtn_wjs"))return;a=d.createElement(e);m=d.getElementsByTagName(e)[0];a.async=1;a.id="likebtn_wjs";a.src=s;m.parentNode.insertBefore(a, m)})(document,"script","//w.likebtn.com/js/w/widget.js");</script>
<!-- LikeBtn.com END -->

{% include docs-footer.md %}