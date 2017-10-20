# Diagnostic Notes for Google Compute Engine

When a job runs much longer than expected or spontaneously stops, it may be adversely affected by an undiagnosed performance bottleneck. Identiying the underlying cause would help resolve the problem and allow the job to run more efficiently.

To diagnosis such an issue, it is important to monitor CPU and disk usage, as well as Serial port console output, via the Google Cloud Console. Outputs from `gsutil` may also be helpful. Memory usage may be monitored by `ssh` login to the compute node. Alternatively, one may also run progams such as `conky` as background process on the compute node to print usage statistics to `stderr` (see [conkymon](https://github.com/djhshih/conkymon)). Standard Linux utilities such as `df` and `free` may also be launched in the background on the node to monitor specific disk usage and memory usage respectively.

The causes of unnecessarily long running jobs may include:


## Excessive writing to boot disk

Some programs can write massive amount of data to disk unbeknownst to the user at locations such as `/tmp` or `/var/run`. This can cause the program to slow to a crawl because the sustained read/write throughput limit of the standard boot disk is very low (1.2 MB/s). In contrast, additional persistent disks have much higher throughput limit (up to 120 MB/s), and persistent solid state disk may have an even higher throughput (> 240 MB/s).

### Symptoms

- Output files appear in such locations as `/tmp`.
- Unexpected, sustained low CPU usage
- Unexpected, sustained low disk write

### Intervention

Ensure the program writes output to an additional persistent disk instead; either redirect the output to another directory on a persistent disk or mount a persistent disk at the location where the program is writing, e.g. `/tmp`.


## Running out of persistent disk space

If a program is writing more (temporary) data to the persistent disk than the latter can store. Depending on the program's quality, it may crash unceremoniously or exit more gracefully with a non-zero status code and a helpful error message. This overflowing problem is distinct from the writing to boot disk problem because it is not limited by low write throughput.

### Symptoms

- Program may output: `Cannot write to file`.
- Unexpected program crash

### Intervention

Track the size of the output (temporary) data in real time by running `df` as a background process, and allocate sufficient persistent disk space. (At the time of writing this document, `conky` can correctly report usage of any mount point within a `docker` container; however, this functionality does not work within the Google compute engine environment due to an undiagnosed cause.)


## Running out of memory

Google Cloud Console does not track memory usage. When a compute node runs out of memory, it may exhibit unpredicted behaviour. Sometimes the node will be automatically killed; other times, the node will linger on for an extended period of time.

### Symptoms

- Serial port console output contains error: `[Errno 12] Cannot allocate memory`.
- Sustained disk read for an extended period of time (if the program's heavy memory consumption is primarily due to data input).
- If CPU and disk usage are normally cyclical, they may now oscillate without predictable pattern.
- Job is killed unexpectedly (but it was not preempted).

### Intervention

Request compute engines with more memory. Reduce memory usage of the program by tweaking its command line parameters. (Depending on the program's quality, it may not adhere to the requested memory limit.) Process data in smaller batches. Modify the program or switch to another program to reduce memory footprint.


## Underutilizing CPU

Since the cost per CPU is fixed across the preconfigured compute engines, one may reduce job run time by using more cores with negligible increase in cost (due to mandatory single-threaded tasks within a job workflow). If the program supports native multithreading, it may be configured to use more parallel threads (with possibly higher memory consumption that may exceed the alloted memory). Alternatively, the input data may split into batches, and the batches may be processed on parallel.


## Mismatched memory to CPU ratio

The preconfigured compute engines come with a fixed RAM memory to CPU ratio. A memory intensive program may underutilized CPU, or a CPU intensive program may underutilized memory. It may be possible to reduce cost with no performance impact by configuring a custom virtual machine.


## Concluding remarks

Indeed, the first step to maximize performance and minimize cost is to diagnose performance bottlenecks. Unexpected disk input-output as well as memory overconsumption may be difficult to diagnosis without prior awareness. Cloud computing provides unprecedent access to massive computing resources, albeit at considerable cost if misused.
