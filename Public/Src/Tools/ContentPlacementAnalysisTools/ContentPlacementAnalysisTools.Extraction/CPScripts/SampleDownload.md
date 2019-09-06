## Downloading and Sampling

To download and create a week sample, you need to run two scripts:

1. weeklySampleDownload.cmd with the following arguments:

a. Configuration file: A valid path to a json configuration file following this spec:

####################################################################
## Sample configuration file:
####################################################################
{
    "AnalyzerConfig":{
        "Exe": "<path to bxlanalyzer.exe>",
        "SampleProportion": "<the proportion of non epty artifacts that the sample will contain, 0.1 <= x <= 1.0>",
        "SampleCountHardLimit": "<the maximum number of artifacts that a sample could contain, e.g. 1000000>""
    },
    "KustoConfig":{
        "Cluster": "https://cbuild.kusto.windows.net",
          "User": "<a valid id (email)>",
          "AuthorityId": "<the corresponding authorithy id>",
          "DefaultDB": "CloudBuildProd"
    },
    "ConcurrencyConfig":{  
        "MaxDownloadTasks": <Max. number of threads used to download builds. Usually 1, since the server might not allow concurrent downloads>,
        "MaxAnalysisTasks": <Max. number of threads used to analyze builds. Each thread will run a bxlanalyzer.exe instance>,
        "MaxMachineMapTasks": <Max. number of threads for machine map creation. One per processor is fine, since this task is light> 
    }
}
####################################################################

b. The year (int)
c. The month (int, 1-12)
d. The start day (int, 1-31)
e. The number of per-day builds to download (int)
f. The output directory (has to exists before)

The span is fixed in this case to 7 days. This means that a certain number of builds (e) will be downloaded from each day starting at month(c)/day(d)/year(b) and ending
in  month(c)/day(d) + 7/year(b). The output json files are created at (f)/Results and are the input for the next phase.

2. queueData.cmd with the following arguments:

a. Configuration file: A valid path to a json configuration file (same as above)
b. year (int)
c. month (int, 1-12)
d. The output directory (has to exists before)

This will download (to (d)/Results) the queue similarity training data to a csv file + 2 directories named QueueMap and MachineMap which are necessary to load the content placement classifier.
As you probably noticed, this process is sampling a month of data for the queues.