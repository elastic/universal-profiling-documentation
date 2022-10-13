<center><img src="https://elastic.github.io/universal-profiling-documentation/img/logo.png"  width="400" height="400"></center>

## Universal Profiling features

Elastic Universal Profiling is a continuous profiling platform running on top of the Elastic stack.

For more details, see
the [Elastic Universal Profiling product page](https://www.elastic.co/observability/ebpf-continuous-code-profiling)

### Inspecting data in Kibana

When opening Kibana, find in the left menu the "Observability" section, and click on "Universal Profiling" to get
started. You are presented with the [stacktraces](#stacktraces) view.

Universal Profiling currently supports CPU profiling (via stack sampling), and it does not have memory profiling yet.

From the landing page, you can get an overview of the entirety of your data, and slice and dice into more detailed
sections of your fleet by using KQL queries in the search bar. You can apply time-based filtering and fields filtering,
inspecting more narrow portion of data and diving deeper and deeper into your system CPU usage.

See [filtering](TODO) for more details on how to slice data.

#### Stacktraces

In the landing page you will see graphs of stacktraces grouped by containers, deployments, threads, hosts and
traces.

These grouping are derived from where the stacktraces were collected, you may find an empty view in containers and
deployments if your host-agent deployment is profiling systems that do not run any containers, or container
orchestrators.

In a deployment where Universal Profiling is correctly receiving data from host-agents you should _always_ see a graph
in the threads, hosts and traces view.

![stacktraces view](./img/stacktraces-default-view.png)

Here is summary of the type of grouping in each view:

* Containers: stacktraces grouped by container name discovered by the host-agent
* Deployments: stacktraces grouped by deployment name set by the container orchestration (e.g. Kubernetes `ReplicaSet`,
  `DaemonSet`, or `SatefulSet` name)
* Threads: stacktraces grouped by process' command name (`/proc/<PID>/comm` in Linux)
* Hosts: stacktraces grouped by machine's hostname or IP
* Traces: un-grouped stacktraces

You can hover and click each of the stacked barchart sections to show details.

You can arrange the graph to display absolute values, or relative percentage values.

Below the top graph, a list of smaller graphs show the individual trend-line for each of the items.

The percentage displayed in the top-right corner of every smaller graphs is the _relative_ number of occurrences of
every time over the total of samples in the group.

The smaller graphs are ordered in decreasing order, from top to bottom, left to right.

**Do not confuse the displayed percentage with percentage of CPU usage**: Universal Profiler is not meant to show
monitoring data, rather it allows to compare which software running in your infrastructure is the most expensive.

In the "Traces" tab, clicking on "Show more" in each of the smaller graphs will bring up on the screen the full
stacktrace.

![stacktraces show more](./img/stacktraces-detailed-view.png)

_Note_: the stacktrace may be either symbolized, showing the full source code's filename and line number, or partially
symbolized, or not symbolized at all.

Notice how unsymbolized frames _do not_ show a filename and line number, but a hexadecimal number or `<unsymbolized>`.

![stacktraces unsymbolized](./img/stacktraces-unsymbolized-view.png)

Some possible use cases for the stacktraces view:

* discover which container, deployed across on a multitude of machines, is the heaviest CPU hitter
* discover how much relative overhead comes from third party software running on your machines
* detect unexpected CPU spikes across threads, and drill down into a smaller time range to investigate further with a
  flamegraph

#### Flamegraphs

A flamegraph is a visualization technique that groups hierarchical data (stacktraces) into rectangles stacked one onto
or next to each other. The size of each rectangle represents the relative weight of a children compared to the parent.

Flamegraphs provide immediate feedback on what parts of the software should be searched first for optimization
opportunities, highlighting the hottest code paths across your entire infrastructure.

![flamegraph view](./img/flamegraph-default-view.png)

You can navigate a flamegraph on two axis:

* Horizontally: every process that is recorded to be running on your CPUs will have at least a box under the `root`
  frame. In Universal Profiling, flamegraphs you will likely discover the existence of processes you don't control, but
  that are eating significant portion of your CPU resources.
* Vertically: traversing a process' call stack will allow to identify which parts of the process are executing most
  frequently. This allows pinpointing functions or methods that _should_ be negligible, and are instead found to be a
  big portion of your call sites.

Drag the graph up, down, right or left to move the visible area.

You can zoom in and out of a subset of stacktraces, by clicking on individual frames or scrolling up in the colored view.

The summary square in the bottom-left corner of the graph lets you shift the visible area of the graph.
Note how the position of summary square is adjusted when you drag flamegraph, and vice-versa: moving the summary square
will reflect the visible area in the bigger panel.

Enabling the "Show information window" on the right will display a list of entries in a panel on the right.
Clicking on each rectangle in the flamegraph will highlight the frame's detail in the right panel.

![flamegraph frame details](./img/flamegraph-detailed-view.png)

#### Functions

### Filtering

### Resource constraints

One of the key goals of Universal Profiling is to have net positive cost benefit for users: the cost of profiling and
observing applications should not be higher than the savings produced by the optimizations.

In this spirit, both the agent and the storage are engineered to be as efficient as possible in terms of resources
usage.

#### Elasticsearch storage

Universal Profiling storage budget is predictable on a per-profiled-core basis: the data we generate at the fixed
sampling frequency of 20 Hz, will be stored in Elasticsearch at the rate of 20 mega-bytes per core-day.

#### Host-agent CPU and memory

Because Universal Profiling provides whole-system continuous profiling, the resource usage of host-agent is highly
correlated with the number of processes running on the machine.

We have recorded real-world, in production host-agent's deployments to be consuming between 0.5% and 1% of CPU time,
with the process' memory being as low as 50 MB, and as high as 250 MB on busier hosts.
