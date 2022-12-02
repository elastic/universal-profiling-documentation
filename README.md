<p align="center">
  <img src="https://github.com/elastic/universal-profiling-documentation/raw/65188a30b6e7877298a7b4f67fc1bcef3c3324ed/img/logo.png" height="400">
</p>

Welcome to the **private beta** documentation for Universal Profiling.

This documentation will help you configure and use Universal Profiling. We would appreciate feedback on your experience
with this product and any other profiling pain points you may have. See the **[Send Feedback](#send-feedback)** section
at the end of this documentation for more information.

Continue to the next section to get started with Universal Profiling.

If you've completed the initial setup, continue to [Universal Profiling features](./features.md).

## Get started with Universal Profiling

At the moment, Universal Profiling is **only available on [Elastic Cloud](http://cloud.elastic.co)**.

Enabling Universal Profiling on a deployment currently requires some manual actions. Many of these actions will be
automated in upcoming releases.

### Prerequisites

Before setting up Universal Profiling, check the following prerequisites:

- an Elastic stack deployment on [Elastic Cloud](http://cloud.elastic.co) at version 8.5.0 or higher (you can either provision a new one or upgrade an existing one).
- the Integrations Server must be enabled in the deployment.
- credentials (either an API key or username/password) for the `superuser` Elasticsearch role (typically,
  the `elastic` user).
- a x86_64 Linux machine with a terminal to run commands.

#### Interpeters

Univeral Profiling is a system-wide profiling solution with additional support for PHP, Python, Java (or any JVM language), Go, Rust, C/C++, Node.js/V8, Ruby, and Perl.

The minimum supported versions of interpreters are: 

- JVM/JDK: >= 7
- Python: >= 3.6 
- V8: >= 8.1.0
- Perl: >= 5.28 
- PHP: >= 7.3 
- Ruby: >= 2.5 

### Deployment configuration example

Before creating a new cluster or upgrading an existing one, review the suggested configuration tier for each Elastic
stack component.

The deployment configuration example in the following table was tested to support profiling data from a fleet of up to
500 hosts, each with 8 or 16 CPU cores, for a total of roughly 6000 cores.

| Component           | Size per zone (memory) | Zones | 
|---------------------|------------------------|-------|
| Elasticsearch       | 64 GB                  | 2     |
| Kibana              | 8 GB                   | 1     |
| Integrations Server | 8 GB                   | 1     |

Even if you're profiling a smaller fleet, we recommend configuring at least 2 zones for Elasticsearch and 4 GB of memory
each for the Integrations Server and Kibana.

### <a name="setup"></a> Setting up Universal Profiling on a Cloud deployment

Follow these steps to enable the Universal Profiling app in Kibana:

1. Log in to [Elastic Cloud](https://cloud.elastic.co) and locate your deployment.
2. Click the **Manage deployment** icon next to your deployment.
3. Click **Edit** in the navigation menu.
4. Scroll down to the Kibana section, and click **Edit user settings**.
   ![edit Kibana user settings](./img/kibana-edit-user-settings.png)
5. Add this line to the user settings:
   ```yaml
   xpack.profiling.enabled: true
   ```
   ![edit Kibana user settings](./img/kibana-edit-user-settings-popup.png)
6. Click **Back** at the bottom of the **User settings** pane.
7. Scroll to the bottom of the **Edit** page, and click **Save** to save your settings.

Once Kibana has the Universal Profiling app enabled, it's located under **Observability** in the navigation menu.

Next, follow these steps to configure data ingestion:

1. Copy your deployment's Cloud ID from the deployment overview page.
   ![cloud ID](./img/cloud-id.png)
1. On your x86_64 Linux machine, open a terminal to execute the following steps.
1. Download and extract the `elastic-profiling` CLI by replacing the `<PROVIDED_URL>` placeholder in the following example with the URL you received in your private beta onboarding call:
   ```bash
   wget -O- <PROVIDED_URL> | tar xz
   chmod +x elastic-profiling
   ```
    If you haven't signed up for the private beta, sign up [here](https://docs.google.com/forms/d/e/1FAIpQLSd-SWVgvhO7Z_jAfaV9_bFGa0dUZPuX0JORzPGS8SDP7G-dVQ/viewform).
  
1. Use the Cloud ID and the `superuser` Elasticsearch credentials to set up Universal Profiling in your deployment.
   In the following example, replace the placeholders in `<>` with the proper values for your deployment.
   ```
   ./elastic-profiling setup cloud --reset --cloud-id=<CLOUD_ID> --es-user=<ES_USERNAME> --es-password=<ES_PASSWORD>
   ```
1. Confirm that this is the first time setting up Universal Profiling in the terminal prompt.
   If Universal Profiling has already been installed, confirm with your cluster administrator that it's ok to erase all
   existing profiling data.

### Installing the host-agent

The host-agent is the component that profiles your fleet and needs to be installed and configured on every machine that
you want to profile. It needs elevated privileges in order to run (currently: `root` / `CAP_SYS_ADMIN`). The following
instructions guide you through the basic setup of a host-agent on your Linux machine.

If everything is working correctly, you can deploy the host-agent across your fleet in the final step of the following instructions:

1. Copy your deployment's APM Cluster ID from the deployment overview page.
   ![apm cluster ID](./img/apm-cluster-id.png)
1. Copy your deployment's Cloud ID from the deployment overview page (like in the first step of the previous section).
1. Use `elastic-profiling` to print the host-agent installation and configuration instructions for various package formats.
   The default package format is binary but there are also instructions for DEB, RPM, Docker and Kubernetes. The DEB and
   RPM package formats contain systemd service definitions that will ensure the host-agent stays running as a service.

   You can list all available package formats by running:
   ```bash
   ./elastic-profiling help config
   ```

1. Print the `binary` configuration to test it on your current Linux machine:
   ```bash
   ./elastic-profiling config --binary --apm-cluster-id=<APM_CLUSTER_ID> --cloud-id=<CLOUD_ID> \
     --es-user=<ES_USERNAME> --es-password=<ES_PASSWORD>
   ```
1. Run the host-agent with the provided steps, testing that your Universal Profiling deployment is working as expected.
   The host-agent will print out logs that will notify you if the connection to Elastic Cloud is not working.
   In this case, see [troubleshooting and support](#troubleshooting-and-support).
1. After a few minutes, open Kibana and confirm you can see stacktraces data coming from your host.
   Navigate to the **Threads** tab in **Observability > Universal Profiling > Stacktraces**. You should see a graph and a
   list of processes.
1. You can now print more configurations to deploy the host-agents on your fleet.

**Notes on the host-agent configuration**

Note that the printed configuration uses a `stable` version.
This is good for testing environments, but for production we recommend to use immutable, versioned artifacts.

The host-agent versioning scheme is **not aligned with the Elastic stack version scheme**.

The OS packages downloaded from `releases.prodfiler.com` have a version in their file name.

For container images, you can find a list of versions in the
[Elastic container library repository](https://container-library.elastic.co/r/observability/profiling-agent).

For Kubernetes deployments, the Helm chart version is already used to configure the same container image, unless
overwritten with the `version` parameter in values.

### Splitting/filtering data from multiple workloads

`elastic-profiling config` prints a default configuration that allows ingesting data into a Cloud deployment.
There is only one config knob that you can change: `project-id` (default value is `1`).

The `-project-id` flag, or the `project-id` key in the host-agent configuration file, is a parameter to split
profiling data into logical groups that you control.

You are free to assign any non-zero, unsigned integer to a host-agent deployment you control. In Kibana,
the KQL field `service.name` is mapped to `project-id` and can be used to slice/filter data.

You may want to set a per-environment project ID (i.e. dev=3, staging=2, production=1), a per-datacenter project ID (
i.e. DC1=1, DC2=2), or even a per-k8s-cluster project ID (i.e. us-west2-production=100, eu-west1-production=101).

### Adding symbols for native frames

To see function names and line numbers in traces of applications written in programming languages that 
compile to native code (C, C++, Rust, Go, ...), the user must first push symbols to the cluster. This
is done using the `elastic-profiling push-symbols` command. All variants of this command require the
same authentication parameters as the `config` command discussed in previous sections of this document:

```
   --apm-cluster-id=<cluster id>
   --cloud-id=<cloud id>
   --es-user=<elasticsearch user>
   --es-password=<elasticsearch password>
```

These four parameters are referred to as `<auth args>` in the following sub-sections.

#### Go applications

The meta-information present in Go binaries allows them to be symbolized even if they were stripped.
No additional parameters need to be passed during the build. To push symbols for a Go binary, simply 
invoke the `elastic-profiling` tool like this:

```
./elastic-profiling push-symbols executable <auth args> -e ./my-go-app 
```

#### C, C++ and Rust applications

C/C++ applications must be built with debug symbols (`-g`) for symbolization to work. Rust applications
must be built with [`debug = 1`][cargo-debug] (or higher). The debug info doesn't have to be deployed to 
production, but needs to be present temporarily to push them to the Elastic cluster. 

[cargo-debug]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

If you don't mind deploying your applications with debug symbols, you can simply do:

```
./elastic-profiling push-symbols executable <auth args> -e ./my-c-app 
```

If you don't want debug symbols in production, simply copy the executable and strip the copy.
You can then use the `-d` argument to instruct the tool to read the symbols from the original
unstripped binary while still calculating the file hash from the final stripped binary. After
the symbols have been pushed, the unstripped binary can be removed.

```
cp ./my-app ./my-stripped-app
strip ./my-stripped-app
./elastic-profiling push-symbols executable <auth args> -e ./my-stripped-app -d ./my-app
rm ./my-app
```

> **Warning**
>
> Simply pushing debug information and then stripping the binary later **will not work**:
> the executable passed via the `-e` argument is used to calculate the file hash that
> associates stack traces with their symbols and stripping the binary later will change that hash.

#### Linux distribution packages

For Debian, Ubuntu, Fedora and Arch Linux, the `elastic-profiling` tool supports pushing symbols
for an entire package at once. The required debug symbols are automatically retrieved from
the distribution's [debuginfod] server.

[debuginfod]: https://wiki.debian.org/Debuginfod

For this to work, please make sure that the debuginfod client is installed on your machine.

| Distro | Install command                               |
|--------|-----------------------------------------------|
| Debian | `sudo apt install debuginfod`                 |
| Ubuntu | `sudo apt install debuginfod`                 |
| Fedora | `sudo dnf install elfutils-debuginfod-client` |
| Arch   | `sudo pacman -S debuginfod`                   |

Then, invoke `elastic-profiling` as follows:

```
./elastic-profiling push-symbols package <auth args> -p package-name
```

For example, to push symbols for libc on Debian:

```
./elastic-profiling push-symbols package <auth args> -p libc6
```

Note that debuginfod servers for many distributions tend to be rather unreliable: if the tool
ends up printing `No debug info found for executable, skipping.` for all executables in a
package, this is likely the reason. This is more likely to take place if you're on a faster
moving release-channel of your distribution (e.g. Debian Testing).

#### Native symbolization limitations

In the current beta release of Elastic Universal Profiling, native symbolization is still limited
in a few important ways:

- No virtual frames for inline functions
- No symbols for leaf frames (top-most frame in a trace)
- No automatic insertion of debug symbols for OS packages

We're aiming that these limitations be improved on and eventually lifted in later versions.

> **Note**
>
> If symbols are not displayed in Kibana after ingesting them, try restarting Kibana in your deployment (Cloud -> Deployments -> <Deployment> -> Kibana --> Force Restart). 
> This is a known issue in 8.5 and will be fixed in later versions. 



### Troubleshooting and support

We mentioned earlier that you can spot errors in the host-agent logs.

The following is an example of a _healthy_ host-agent output:

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

You can confirm that a host-agent deployment is working if you run the following command and it has an empty output:

```bash
head host-agent.log -n 15 | grep "level=error"
```

If the output of the previous command contains error level logs, the following are possible causes:

- The host-agent is running on an unsupported version of the Linux kernel, or it can't perform its operations because
   of missing kernel features.

   In case of an outdated kernel version, this message will be logged:
   ```text
   Host Agent requires kernel version 4.15 or newer but got 3.10.0
   ```

   In case of eBPF features not being available in the kernel, the host-agent will fail starting up, and the logs will
   contain:
   ```text
   Failed to probe eBPF syscall
   ```
   or
   ```text
   Failed to probe tracepoint
   ```

- The host-agent is not able to connect to Elastic Cloud.
   In this case, you should see a message like the following:
   ```text
   Failed to setup gRPC connection (retrying...): context deadline exceeded
   ```

   Verify the `collection-agent` configuration value is set and is equal to what was printed
   by `elastic-profiling config`.

- The secret token is not valid, or it has been changed.
   In this case, the host-agent will shut down, logging an error message like the following:
   ```text
   rpc error: code = Unauthenticated desc = authentication failed
   ```

- The host-agent is unable to reach the collectionagent service.
   In this case, you should see a message like the following:
   ```text
   Failed to report hostinfo (retrying...): rpc error: code = Unimplemented desc = unknown service collectionagent.CollectionAgent"
   ```

   This typically means that your Cloud cluster has not been configured for Universal Profiling.
   Follow the steps under the [setup](#setup) section.

- The APM server (part of the backend in Cloud that receives data from the host-agent) died
   (e.g. ran out of memory):
   ```text
   Error: failed to invoke XXX(): Unavailable rpc error: code = Unavailable desc = unexpected HTTP status code received from server: 502 (Bad Gateway); transport: received unexpected content-type "application/json; charset=UTF-8"
   ```

   Verify that the APM server is running, by browsing to Cloud->Deployments->"Deployment"->Integrations Server
   in the [Cloud](https://cloud.elastic.co/home) UI. If the "Copy endpoint" link next to APM is grayed out,
   the APM server is not operational and needs to be restarted. This can take place by clicking "Force Restart".
   ![restart-integrations-server](./img/restart-integrations-server.png)

If you are unable to find a solution to the host-agent failure, you can raise a support request
indicating `Universal Profiling` and `host-agent` as source of the problem.

#### Enabling verbose logging in host-agent

During the support process, you may be asked to provide debug logs from one of the host-agent installations from your
deployment.

**We recommend only enabling debug logs on a single instance of host-agent**, rather than an entire deployment because
of the amount of logs produced.

To enable debug logs, add the `-verbose` command-line flag or the `verbose true` setting in the configuration file.

#### Troubleshoot host-agent K8s deployments

When the Helm chart installation completes, the output has instructions on how to check the host-agent pod status and
read logs. A series of scenarios that could manifest when the host-agent installation **is not healthy** follows.

##### Taints

K8s clusters often include in their
setup [taints and tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/).
In such cases, a host-agent installation may display no pods or very few pods running, even for a large cluster.

The reason for that: a _taint_ precludes the execution of pods on a node, unless the workload has been _tolerated_.
The Helm chart `tolerations` key in `values.yaml` sets the toleration of taints, using the official K8s scheduling API
format.

For some example use cases, we provide a `tolerations` config to be added in the Helm chart `values.yaml`

* To deploy the host-agent on all nodes with taint `workload=python:NoExecute`: add
  in `values.yaml`
  
  ```
  tolerations:
    - key: "workload"
      value: "python"
      effect: "NoExecute"
  ```

* To deploy the host-agent on all nodes tainted with _key_ `production` and effect `NoSchedule` (no value
  provided): add in `values.yaml`
  
  ```
  tolerations:
    - key: "production"
      effect: "NoSchedule"
      operator: Exists
  ```

* To deploy the host-agent on all nodes, tolerating all taints: add in `values.yaml`
  
  ```
  tolerations:
    - effect: NoSchedule
      operator: Exists
    - effect: NoExecute
      operator: Exists
  ```

##### Security policy enforcement

Some K8s clusters are configured with hardened security add-ons, to limit the blast radius of application
vulnerabilities being exploited. Different hardening methodologies can impair host-agent operations and
may for example result in pods continuously restarting after displaying a `CrashLoopBackoff` status.

###### K8s PodSecurityPolicy ([deprecated](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/))

This K8s API has been deprecated, but some still use it. A PodSecurityPolicy (PSP) may explicitly prevent the execution
of `privileged` containers across the entire cluster.

Since host-agent _needs_ privileges in most kernels/CRI, you need to build a PSP to allow the host-agent
DaemonSet to run.

###### K8s policy engines

Read more about these in
the [SIG-Security doc](https://github.com/kubernetes/sig-security/blob/main/sig-security-docs/papers/policy/kubernetes-policy-management.md)
.

Tools like the following _may_ prevent the execution of host-agent pods, as the Helm chart builds a cluster role and
binds it into the host-agent service account (we use it for container metadata):

* Open Policy Agent Gatekeeper
* Kyverno
* Fairwinds Polaris

If you have a policy engine in place, you should configure it to allow the host-agent execution and RBAC configs.

###### Network configurations

Even if host-agent pods are running fine, they may not be able to connect to the remote data collector gRPC interface,
and stay in the startup phase, periodically trying to connect.

Possible causes for it:

* K8s [`NetworkPolicies`](https://kubernetes.io/docs/concepts/services-networking/network-policies/) define connectivity
  rules that prevent all outgoing traffic unless explicitly allow-listed
* Cloud/Datacenter provider network rules are restricting egress traffic to allowed destinations only (ACLs)

###### OS-level security

These settings _are not part of Kubernetes_ and may have been included in the setup of the nodes. They may prevent host-agent
from working properly, as they'd be intercepting syscalls from HA to the Kernel, modifying or blocking them.

If you have implemented security hardening (some providers listed below), you should be aware of the privileges needed by HA.

* gVisor on GKE
* seccomp filters
* AppArmor LSM


#### Submit a support request

Reach the [support request page](https://cloud.elastic.co/support) in the Cloud console.

Depending on what type of problem you faced during the setup or operation of Universal Profiling, please specify in the
request if the problem is in the host-agent or the Kibana app.

#### Send feedback

If troubleshooting and support are not working for you, or you have any other feedback that you want to share about the
product, send the Profiling team an email at `profiling-feedback@elastic.co`.
