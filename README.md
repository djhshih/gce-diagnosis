# Diagnostic Notes for Google Cloud Engine

When a job is running much longer than expected or spontaneously stop, one or more of the following causes could explain the symptoms. Identiying the cause would help resolve the problem and allow the job to run more efficiently.

To diagnosis such an issue, it is important to monitor CPU and disk usage via the Google Cloud console. Memory usage may be monitored by `ssh` login to the compute node. Alternatively, one may also run progams such as `conky` as background process on the compute node to print usage statistics to `stderr` (see [conkymon](https://github.com/djhshih/conkymon)). Standard Linux utilities such as `df` and `free` may also be launched in the background on the node to monitor specific disk usage and memory usage respectively.

The causes of unexpected long-running jobs may include:

## Writing to boot disk

Some programs can write massive amount of data to disk unbeknownst to the user at locations such as `/tmp`. This can cause the program to slow to a crawl because the sustained read/write throughput limit of the boot disk is very low (1.2 MB/s). In contrast, additional persistent disks have much higher throughput limit (up to 120 MB/s), and persistent solid state disk have even higher throughput.

### Symptoms

- Output files in such locations as `/tmp`
- Unexpected, sustained Low CPU usage
- Unexpected, sustained Low disk write

### Treatment

Ensure the program writes output to a mounted disk instead; either redirect the output to another directory on a persistent disk or mount a persistent disk at the location where the program is writing, e.g. `/tmp`.


## Out of memory

Google Cloud console does not track memory usage. When a compute node runs out of memory, it may exhibit unpredicted behaviour. Sometimes the node will be automatically killed; other times, the node will linger on for an extended period of time.

### Symptoms

- Serial port console output contains error: `[Errno 12] Cannot allocate memory`.
- Sustained disk read for an extended period of time (if the program's heavy memory consumption is primarily due to data input)
- CPU usage and disk usage may not appear abnormal (usage varies through time)
- Job is killed unexpected (but not preempted)

### Treatment

Reduce memory usage of the program by tweaking its command line parameters. Modify the program or script to reduce memory footprint. Process data in smaller batches. Request compute engines with more memory.
