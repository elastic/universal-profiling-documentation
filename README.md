![logo](./img/logo.png)

Welcome to Universal Profiling documentation.

Universal Profiling is in **private beta**, which means that this documentation exists as guidance during the process of
trying out the product, with some limitations and sharp corners.

Continue to the next section to get started with Universal Profiling.

## Getting started with Universal Profiling

At the moment, Universal Profiling is **available only on [Elastic Cloud](http://cloud.elastic.co)**.
Eventually, it may also be available in self-managed or open-source distributions of the Elastic stack.

### Elastic Cloud

Enabling Universal Profiling on a deployment requires some manual actions.
Many of these manual actions will be replaced by automated configurations in Elastic Cloud in the upcoming releases.

Before you dive into the setup, you will need:

- a Cloud account with a Platinum subscription
- a deployment at version 8.5.0 or higher (you can either provision a new one or upgrade an existing one)
- the Integrations Server should be enabled in the deployment
- credentials (either an API key or username/password) granted with the `superuser` Elasticsearch role (typically,
  the `elastic` user)
- a Linux machine with a terminal to run commands

With the above points sorted out, you are ready to get started!

The following diagram summarize the steps we are going to detail below.

![user-diagram](./img/user-diagram.png)

#### Deployment reference

Before creating a new cluster, or upgrading an existing one to 8.5, please review the suggested configuration tier
for each Elastic stack component.

The reference deployment configuration in the table was tested to support profiling data from a fleet up to 500 hosts,
each with 8 or 16 CPU cores, for a total of roughly 6000 cores.

| Component           | Size per zone (memory) | Zones | 
|---------------------|------------------------|-------|
| Elasticsearch       | 64 GB                  | 2     |
| Kibana              | 8 GB                   | 1     |
| Integrations Server | 8 GB                   | 1     |

Even if you have a smaller fleet to be profiled, for an optimal user experience we 
recommend to configure at least 4 GB of memory for Integrations Server as well as Kibana.

#### Setting up Universal Profiling on a Cloud deployment

Follow these steps to enable the Universal Profiling app in Kibana:

1. Login to `cloud.elastic.co` and locate your deployment.
2. Edit the deployment.
3. Scroll down to the Kibana section, and click on "Edit user settings".
   ![edit Kibana user settings](./img/kibana-edit-user-settings.png)
4. Add a line in the user settings:
   ```yaml
   xpack.profiling.enabled: true
   ```
   ![edit Kibana user settings](./img/kibana-edit-user-settings-popup.png)
5. Go back and scroll to the bottom of the page.
6. Save the settings by clicking on Save.
7. If you encounter an error during the configuration change, [submit a support request](#submit-a-support-request)
   to add the previous line in Kibana user settings.

Once Kibana has the Universal Profiling app enabled, it's visible under Observability in the left menu.
You can now go ahead and configure Elasticsearch, Fleet and APM to allow ingesting data.

These steps will guide you in configuring Elasticsearch, Fleet and APM to enable the Universal Profiling storage:

1. Fetch the Cloud ID of your deployment from the Cloud console.
   ![cloud ID](./img/cloud-id.png)
1. Within your Linux machine, open a terminal to run the next steps.
1. Download and extract the `elastic-profiling` CLI:
   ```bash
   wget -O- https://releases.prodfiler.com/stable/elastic-profiling.tgz | tar xz
   chmod +x elastic-profiling
   ```
1. Use the Cloud ID and the `superuser` Elasticsearch credentials to set up Universal Profiling in your deployment.
   Replace the placeholders in `<>` with the proper values for your deployment.
   ```
   ./elastic-profiling setup --reset --cloud-id=<CLOUD_ID> --es-user=<ES_USERNAME> --es-password=<ES_PASSWORD>
   ```
1. Confirm that this is the first time setting up Universal Profiling in the terminal prompt.
   If Universal Profiling has already been installed, confirm with your cluster administrator that you are willing to
   erase all existing profiling data.

#### Installing the host-agent

1. Fetch the APM Cluster ID of your deployment from the Cloud console.
   ![apm cluster ID](./img/apm-cluster-id.png)
1. Use `elastic-profiling` to print the host-agent configuration for various platforms.
   You can list all available platforms by running
   ```bash
   ./elastic-profiling help config
   ```
1. Print the `binary` configuration to test it out on the VM you are using:
   ```bash
   ./elastic-profiling config --binary --apm-cluster-id=<APM_CLUSTER_ID> --es-user=<ES_USERNAME> --es-password=<ES_PASSWORD>
   ```
1. Run the host-agent with the provided steps, testing your Universal Profiling deployment is working as expected.
   The host-agent will print out logs that will notify if the connection to the collector is not working.
   In such case, see [troubleshooting and support](#troubleshooting-and-support).
1. In a few minutes, open Kibana and confirm you can see stacktraces data coming from your host.
   Move to the `Threads` tab in Observability > Universal Profiling > Stacktraces, you should see a graph and a list of
   processes.
1. You can now print more configurations to deploy the host-agents on your fleet.

**Note on the host-agent configuration**

`elastic-profiling config` prints a default configuration that allows ingesting data into a Cloud deployment.
There is only a config knob that you can change: `project-id` (default value is `1`).

The `-project-id` flag, or the `project-id` key in the host-agent configuration file, is a parameter to split
profiling data into logical groups.

You are free to assign any non-zero, unsigned integer to a host-agent deployment you control: this will be helpful to
slice profiling data in Kibana.
You may want to set a per-environment project ID (i.e. dev=3, staging=2, production=1), a per-datacenter project ID (
i.e. DC1=1, DC2=2), or even a per-k8s-cluster project ID (i.e. us-west2-production=100, eu-west1-production=101).

#### Adding symbols for native frames

TODO

#### Troubleshooting and support

We briefly mentioned earlier that you can spot errors in the host-agent logs.

First off, how does a _healthy_ host-agent output look like? Like this

```
time="..." level=info msg="Starting Prodfiler Host Agent v2.4.0 (revision develop-5cce978a, build timestamp 12345678910)"
time="..." level=info msg="Interpreter tracers: perl,php,python,hotspot,ruby,v8"
time="..." level=info msg="Automatically determining environment and machine ID ..."
time="..." level=warning msg="Environment tester (gcp) failed: failed to get GCP metadata: Get \"http://169.254.169.254/computeMetadata/v1/instance/id\": dial tcp 169.254.169.254:80: i/o timeout"
time="..." level=warning msg="Environment tester (azure) failed: failed to get azure metadata: Get \"http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01&format=json\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
time="..." level=warning msg="Environment tester (aws) failed: failed to get aws metadata: EC2MetadataRequestError: failed to get EC2 instance identity document\ncaused by: RequestError: send request failed\ncaused by: Get \"http://169.254.169.254/latest/dynamic/instance-identity/document\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
time="..." level=info msg="Environment: hardware, machine ID: 0xdeadbeefdeadbeef"
time="..." level=info msg="Assigned ProjectID: 5"
time="..." level=info msg="Start CPU metrics"
time="..." level=info msg="Start I/O metrics"
time="..." level=info msg="Found tpbase offset: 9320 (via x86_fsbase_write_task)"
time="..." level=info msg="Environment variable KUBERNETES_SERVICE_HOST not set"
time="..." level=info msg="Supports eBPF map batch operations"
time="..." level=info msg="eBPF tracer loaded"
time="..." level=info msg="Attached tracer program"
time="..." level=info msg="Attached sched monitor"
```

In essence, you can validate a host-agent deployment is working as intended, if the following command has an empty
output:

```bash
head host-agent.log -n 15 | grep "level=error"
```

If the output of the above command is not empty and there are error level logs, we list below some possible causes of
malfunction:

1. The host-agent is running on an unsupported version of Linux kernel, or it can't perform its operations because of
   missing kernel features.

   In case of outdated kernel version, this message will be logged:
   ```text
   Host Agent requires kernel version 4.15 or newer but got 3.10.0
   ```

   In case of eBPF features not being available in the kernel, host-agent will fail starting up, and logs will contain
   ```text
   Failed to probe eBPF syscall
   ```
   or
   ```text
   Failed to probe tracepoint
   ```

1. The host-agent is not able to connect to the collector.
   In this case you should see a message like the following:
   ```text
   Failed to setup gRPC connection (retrying...): context deadline exceeded
   ```

   Verify the `collection-agent` configuration value is set and is equal to what was printed
   by `elastic-profiling config`.

1. The secret token is not valid, or it has been changed.
   In this case, the host-agent will shut down, logging an error message like the following:
   ```text
   rpc error: code = Unauthenticated desc = authentication failed
   ```

If you are unable to find a solution to the host-agent failure, you can raise a support request,
indicating `Universal Profiling` and `host-agent` as source of the problem.

##### Enabling verbose logging in host-agent

During the support process, you may be asked to provide debug logs from one of the host-agent installations from
your deployment.

**We recommend to enable debug logs only on a single instance of host-agent**, rather than an entire deployment, because
of the amount of logs produced.

To enable debug logs, add the `-verbose` command-line flag or the `verbose true` setting in the configuration file.

##### Submit a support request

Reach the [support request page](https://cloud.elastic.co/support) in the Cloud console.

Depending on what type of problem you faced during the setup or operation of Universal Profiling, please specify in the
request if the problem is in the host-agent, or the Kibana app.

##### Send feedback

If troubleshooting and support are not working for you, or you have any other feedback that you want to share about
the product, send the Profiling team an email to `profiling-feedback@elastic.co`.
