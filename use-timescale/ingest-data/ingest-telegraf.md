---
title: Ingest data using Telegraf
excerpt: Ingest data into a Timescale Cloud service using using the Telegraf plugin
products: [cloud, mst, self_hosted]
keywords: [ingest, Telegraf]
tags: [insert]
---

import ImportPrerequisites from "versionContent/_partials/_migrate_import_prerequisites.mdx";
import SetupConnectionString from "versionContent/_partials/_migrate_import_setup_connection_strings_parquet.mdx";

# Ingest data using Telegraf

Telegraf is a server-based agent that collects and sends metrics and events from databases, 
systems, and IoT sensors. Telegraf is an open source, plugin-driven tool for the collection 
and output of data. 

To view metrics gathered by Telegraf and stored in a [hypertable][about-hypertables] in a
$SERVICE_LONG.

- [Link Telegraf to your $SERVICE_LONG](#link-telegraf-to-your-service): create a Telegraf configuration
- [View the metrics collected by Telegraf](#view-the-metrics-collected-by-telegraf): connect to your $SERVICE_SHORT and
  query the metrics table

## Prerequisites

<ImportPrerequisites />

- [Install Telegraf][install-telegraf]


## Link Telegraf to your service

<Procedure>

To create a Telegraf configuration that exports data to a hypertable in your $SERVICE_SHORT:

1. **Setup your $SERVICE_SHORT connection string**

    <SetupConnectionString />

1. **Generate a Telegraf configuration file**

    In Terminal, run the following:

    ```bash
    telegraf --input-filter=cpu --output-filter=postgresql config > telegraf.conf
    ```

    `telegraf.conf` configures a CPU input plugin that samples
    various metrics about CPU usage, and the PostgreSQL output plugin. `telegraf.conf`
    also includes all available input, output, processor, and aggregator
    plugins. These are commented out by default.

1.  **Test the configuration**

    ```bash
    telegraf --config telegraf.conf --test
    ```

    You see an output similar to the following:

    ```bash
    2022-11-28T12:53:44Z I! Starting Telegraf 1.24.3
    2022-11-28T12:53:44Z I! Available plugins: 208 inputs, 9 aggregators, 26 processors, 20 parsers, 57 outputs
    2022-11-28T12:53:44Z I! Loaded inputs: cpu
    2022-11-28T12:53:44Z I! Loaded aggregators:
    2022-11-28T12:53:44Z I! Loaded processors:
    2022-11-28T12:53:44Z W! Outputs are not used in testing mode!
    2022-11-28T12:53:44Z I! Tags enabled: host=localhost
    > cpu,cpu=cpu0,host=localhost usage_guest=0,usage_guest_nice=0,usage_idle=90.00000000087311,usage_iowait=0,usage_irq=0,usage_nice=0,usage_softirq=0,usage_steal=0,usage_system=6.000000000040018,usage_user=3.999999999996362 1669640025000000000
    > cpu,cpu=cpu1,host=localhost usage_guest=0,usage_guest_nice=0,usage_idle=92.15686274495818,usage_iowait=0,usage_irq=0,usage_nice=0,usage_softirq=0,usage_steal=0,usage_system=5.882352941192206,usage_user=1.9607843136712912 1669640025000000000
    > cpu,cpu=cpu2,host=localhost usage_guest=0,usage_guest_nice=0,usage_idle=91.99999999982538,usage_iowait=0,usage_irq=0,usage_nice=0,usage_softirq=0,usage_steal=0,usage_system=3.999999999996362,usage_user=3.999999999996362 1669640025000000000
    ```

1. **Configure the PostgreSQL output plugin**

   1.  In `telegraf.conf`, in the `[[outputs.postgresql]]` section, set `connection` to
      the value of $TARGET.

      ```bash
      connection = "<VALUE OF $TARGET>"
      ```

   1. Use hypertables when Telegraf creates a new table:

      In the section that begins with the comment `## Templated statements to execute
      when creating a new table`, add the following template:

      ```bash
      ## Templated statements to execute when creating a new table.
      # create_templates = [
      #   '''CREATE TABLE {{ .table }} ({{ .columns }})''',
      # ]
      #  table_template=`CREATE TABLE IF NOT EXISTS {TABLE}({COLUMNS}); SELECT create_hypertable({TABLELITERAL},by_range('time', INTERVAL '1 week'),if_not_exists := true);`

      ```

      The `by_range` dimension builder was added to TimescaleDB 2.13.

</Procedure>


## View the metrics collected by Telegraf

This section shows you how to generate system metrics using Telegraf, then connect to your 
$SERVICE_SHORT and query the metrics [hypertable][about-hypertables].

<Procedure>

1. **Collect system metrics using Telegraf**

    Run the following command for a 30 seconds:  

    ```bash
    telegraf --config telegraf.conf
    ```
    
    Telegraf uses loaded inputs `cpu` and outputs `postgresql` along with
    `global tags`, the intervals when the agent collects data from the inputs, and 
    flushes to the outputs.

1. **View the metrics**

   1.  Connect to your $SERVICE_LONG:

        ```bash 
         psql $TARGET
       ```

   1.  View the metrics collected in the `cpu` table in `tsdb`:

       ```sql
       SELECT*FROM cpu;
       ```

       You see something like:

       ```sql
       time         |    cpu    |               host               | usage_guest | usage_guest_nice |    usage_idle     | usage_iowait | usage_irq | usage_nice | usage_softirq | usage_steal |    usage_system     |     usage_user
       ---------------------+-----------+----------------------------------+-------------+------------------+-------------------+--------------+-----------+------------+---------------+-------------+---------------------+---------------------
       2022-12-05 12:25:20 | cpu0      | hostname |           0 |                0 | 83.08605341237833 |            0 |         0 |          0 |             0 |           0 |   6.824925815961274 |  10.089020771444481
       2022-12-05 12:25:20 | cpu1      | hostname |           0 |                0 | 84.27299703278959 |            0 |         0 |          0 |             0 |           0 |   5.934718100814769 |   9.792284866395647
       2022-12-05 12:25:20 | cpu2      | hostname |           0 |                0 | 87.53709198848934 |            0 |         0 |          0 |             0 |           0 |   4.747774480755411 |   7.715133531241037
       2022-12-05 12:25:20 | cpu3      | hostname|           0 |                0 | 86.68639053296472 |            0 |         0 |          0 |             0 |           0 |    4.43786982253345 |   8.875739645039992
       2022-12-05 12:25:20 | cpu4      | hostname |           0 |                0 | 96.15384615371369 |            0 |         0 |          0 |             0 |           0 |  1.1834319526667423 |  2.6627218934917614
       ```

       To view the average usage per CPU core, use `SELECT cpu, avg(usage_user) FROM cpu GROUP BY cpu;`.

</Procedure>

For more information about the options that you can configure in Telegraf,
see the [PostgreQL output plugin][output-plugin].


[output-plugin]: https://github.com/influxdata/telegraf/blob/release-1.24/plugins/outputs/postgresql/README.md
[install-telegraf]: https://docs.influxdata.com/telegraf/v1.21/introduction/installation/
[create-service]: /getting-started/latest/
[connect-timescaledb]: /use-timescale/:currentVersion:/integrations/query-admin/about-connecting/
[grafana]: /use-timescale/:currentVersion:/integrations/observability-alerting/grafana/
[about-hypertables]: /use-timescale/:currentVersion:/hypertables/about-hypertables/
