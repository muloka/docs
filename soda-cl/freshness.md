---
layout: default
title: Freshness checks
description: Use a SodaCL freshness check to infer data freshness according to the age of the most recently added row in a table.
parent: SodaCL
---

# Freshness checks  
<!--Linked to UI, access Shlink-->
*Last modified on {% last_modified_at %}*

Use a freshness check to determine the relative age of the data in a column in your dataset. 
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(start_date) < 3d
```

[Define freshness checks](#define-freshness-checks) <br />
&nbsp;&nbsp;&nbsp;&nbsp;[Details and limitations](#details-and-limitations)<br />
&nbsp;&nbsp;&nbsp;&nbsp;[Troubleshoot errors with freshness checks](#troubleshoot-errors-with-freshness-checks)<br />
[Freshness check results](#freshness-check-results)<br />
[Optional check configurations](#optional-check-configurations)<br />
[List of freshness thresholds](#list-of-freshness-thresholds) <br />
[List of comparison symbols and phrases](#list-of-comparison-symbols-and-phrases) <br />
[Go further](#go-further)<br />
<br />


## Define freshness checks

A freshness check measures the age of the youngest row in a table. Using this check, you can infer the freshness of the data in the dataset using the age of the most recently added row in a table. 

In the context of [SodaCL check types]({% link soda-cl/metrics-and-checks.md %}#check-types), freshness checks are unique. This check is limited in its syntax variation, with only a few mutable parts to specify column name, threshold, and, optionally, a "now" variable.

A freshness check has two or three mutable parts:

| a timestamp column name | 
| a variable to specify the value of "now" (optional)| 
| a threshold |  

<br />

The example below defines a check that measures freshness relative to "now", where "now" is the moment you run the scan that executes the freshness check. This example discovers when the last row was added to the `start_date` timestamp column, then compares that timestamp to "now". If Soda discovers that the last row was added more than three days ago, the check fails. 
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(start_date) < 3d
```

| column name | `start_date` |
| threshold |  3d |

<br />

Instead of using the default value for "now" (the time you run the scan that executes the freshness check), you can use a variable to specify the value of "now" at scan time. For example, the following check measures freshness relative to a date that a user specifies at scan time. 
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness using end_date with NOW < 1d
```

| column name | `end_date` |
| variable to specify the value of "now" (optional)| `NOW` |
| threshold |  1d |

At scan time, you use a `-v` option to pass a value for the variable that the check expects for the value of "now". The scan command below passes a variable that the check uses. In your scan command, if you pass a variable with a timestamp, the variable must be in ISO8601 format such as `"2022-02-16 21:00:00"` or `"2022-02-16T21:00:00"`.

```shell
soda scan -d adventureworks -c configuration.yml -v NOW="2022-05-31 21:00:00" checks_test.yml
```

***Known issue:*** <br/> 
<!--CORE-449-->
When introducing a NOW variable into a freshness check, you must use the deprecated syntax that includes `using`. This syntax yields an error message in the scan output, `Syntax of freshness check has changed and is deprecated.  Use freshness(column_name) < 24h30m  See docs` but does not prevent Soda from executing the check. Workaround: ignore the deprecated syntax message.


### Details and limitations
* Out-of-the-box, freshness checks *only* work with columns that contain data types TIMESTAMP or DATE. However, you can apply a freshness check to TEXT type data using the following syntax to cast the column: 
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(createdat::datetime) < 1d
```
* Note that casting a column in a check does not work with a NOW variable.
* The only comparison symbol you can use with freshness checks is `<` *except* when you employ and alert configuration. See [Example with alert configuration](#example-with-alert-configuration) for details.
* The default value for "now" is the time you run the scan that executes the freshness check.
* If no timezone information is available in either the timestamp of the check (scan time), or in the data in the column, a freshness check uses the UTC timezone. Soda converts both timestamps to UTC to compare values.



### Troubleshoot errors with freshness checks

**Problem:** When you run a scan to execute a freshness check, the CLI returns one of the following error message.  

```shell
Invalid staleness threshold "when < 3256d"
  +-> line=2,col=5 in checks_test.yml

```

```shell
Invalid check "freshness(start_date) > 1d": no viable alternative at input ' >'
```

**Solution:** The error indicates that you are using an incorrect comparison symbol. Remember that freshness checks can only use `<` in check, unless the freshness check employs an alert configuration, in which case it can only use `>` in the check. 

<br />

**Problem:** When you run a scan to execute a freshness check that uses a NOW variable, the CLI returns an following error message for `Invalid check`. <!--CORE-449-->

```shell
Invalid check "freshness(end_date) ${NOW} < 1d": mismatched input '${NOW}' expecting {'between', 'not', '!=', '<>', '<=', '>=', '=', '<', '>'}
```

**Solution:** Until the known issue is resolved, use a deprecated syntax for freshness checks using a NOW variable, and ignore the `deprecated syntax` message in the output. For example, define a check as per the following.
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness using end_date with NOW < 1d
```

<br />

## Freshness check results  

When you run a scan that includes a freshness check, the output in the **Soda Library CLI** provides several values for measurements Soda used to calculate freshness. The value for freshness itself is displayed in days, hours, minutes, seconds, and milliseconds; see the example below. 

```shell
Soda Library 1.0.x
Soda Core 3.0.x
Scan summary:
1/1 checks FAILED: 
      Data is fresh [FAILED]
        max_column_timestamp: 2013-07-01 00:00:00
        max_column_timestamp_utc: 2013-07-01 00:00:00+00:00
        now_variable_name: NOW
        now_timestamp: 2022-09-13T16:40:39.196522+00:00
        now_timestamp_utc: 2022-09-13 16:40:39.196522+00:00
        freshness: 3361 days, 16:40:39.196522
Oops! 1 failures. 0 warnings. 0 errors. 0 pass.
```

In **Soda Cloud**, the freshness value represents age of the data in the days, hours, minutes, etc. relative to `now_timestamp`.


## Optional check configurations

| Supported | Configuration | Documentation |
| :-: | ------------|---------------|
| ✓ | Define a name for a freshness check; see [example](#example-with-check-name). |  [Customize check names]({% link soda-cl/optional-config.md %}#customize-check-names) |
| ✓ | Add an identity to a check. | [Add a check identity]({% link soda-cl/optional-config.md %}#add-a-check-identity) |
| ✓ | Define alert configurations to specify warn and fail thresholds; see [example](#example-with-alert-configuration). | [Add alert configurations]({% link soda-cl/optional-config.md %}#add-alert-configurations) |
|   | Apply an in-check filter to return results for a specific portion of the data in your dataset.| - | 
| ✓ | Use quotes when identifying dataset or column names; see [example](#example-with-quotes). <br />Note that the type of quotes you use must match that which your data source uses. For example, BigQuery uses a backtick ({% raw %}`{% endraw %}) as a quotation mark. | [Use quotes in a check]({% link soda-cl/optional-config.md %}#use-quotes-in-a-check) |
|   | Use wildcard characters ({% raw %} % {% endraw %} or {% raw %} * {% endraw %}) in values in the check. |  - |
| ✓ | Use for each to apply freshness checks to multiple datasets in one scan; see [example](#example-with-for-each-checks). | [Apply checks to multiple datasets]({% link soda-cl/optional-config.md %}#apply-checks-to-multiple-datasets) |
| ✓ | Apply a dataset filter to partition data during a scan; see [example](#example-with-dataset-filter). | [Scan a portion of your dataset]({% link soda-cl/optional-config.md %}#scan-a-portion-of-your-dataset) |

#### Example with check name
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(start_date) < 27h:
      name: Data is fresh
```

#### Example with alert configuration

The only comparison symbol that you can use with freshness checks that employ an alert configuration is `>`. 
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(start_date):
      warn: when > 3256d
      fail: when > 3258d
```

OR
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness(start_date):
      warn: 
        when > 3256d
      fail: 
        when > 3258d
```

#### Example with quotes
{% include code-header.html %}
```yaml
checks for dim_product:
  - freshness("end_date") < 3d
```

#### Example with for each
{% include code-header.html %}
```yaml
for each dataset T:
  datasets:
    - dim_prod%
  checks:
    - freshness(end_date) < 3d
```

#### Example with dataset filter
{% include code-header.html %}
```yaml
filter CUSTOMERS [daily]:
  where: TIMESTAMP '{ts_start}' <= "ts" AND "ts" < TIMESTAMP '${ts_end}'

checks for CUSTOMERS [daily]:
  - freshness(end_date) < 3d
```

<br />


## List of freshness thresholds

| Threshold | Example | Reads as |
| --------- | ------- | ----------- |
| `#d` |`3d` |  3 days |
| `#h` | `1h` | 1 hour | 
| `#m` | `30m` | 30 minutes |
| `#d#h` | `1d6h` | 1 day and 6 hours |
| `#h#m` | `1h30m` | 1 hour and 30 minutes |

## List of comparison symbols and phrases

```yaml
# If using without an alert configuration
<
# If using with an alert configuration
>
```


## Go further

* Use missing metrics in checks with alert configurations to establish [warn and fail zones]({% link soda-cl/optional-config.md %}#define-zones-using-alert-configurations)
* Use missing metrics in checks to define ranges of acceptable thresholds using [boundary thresholds]({% link soda-cl/metrics-and-checks.md %}#define-boundaries-with-fixed-thresholds).
* Need help? Join the <a href="https://community.soda.io/slack" target="_blank"> Soda community on Slack</a>.
* Reference [tips and best practices for SodaCL]({% link soda/quick-start-sodacl.md %}#tips-and-best-practices-for-sodacl).
<br />


---

Was this documentation helpful?

<!-- LikeBtn.com BEGIN -->
<span class="likebtn-wrapper" data-theme="tick" data-i18n_like="Yes" data-ef_voting="grow" data-show_dislike_label="true" data-counter_zero_show="true" data-i18n_dislike="No"></span>
<script>(function(d,e,s){if(d.getElementById("likebtn_wjs"))return;a=d.createElement(e);m=d.getElementsByTagName(e)[0];a.async=1;a.id="likebtn_wjs";a.src=s;m.parentNode.insertBefore(a, m)})(document,"script","//w.likebtn.com/js/w/widget.js");</script>
<!-- LikeBtn.com END -->

{% include docs-footer.md %}
