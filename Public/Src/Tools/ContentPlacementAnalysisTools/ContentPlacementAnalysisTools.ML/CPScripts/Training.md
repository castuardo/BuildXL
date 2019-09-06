## Training

To train a set of random forests you need to run the following scripts:

1. createDatabase.cmd with the following arguments:

a. Configuration file: A valid path to a json configuration file following this spec:

####################################################################
## Sample configuration file:
####################################################################
{
    "ConcurrencyConfig":{  
        "MaxBuildParsingTasks": <Max threads to read consolidated samples (output of analyzer)>,
        "MaxArtifactStoreTasks": <Max threads to organize those consolidated samples (per hash)>,
        "MaxArtifactLinearizationTasks": <Max threads to linearize (vectorzing) artifacst>,
        "MaxForestCreationTasks": <Max threads to creatre/train random forests>
    },
    "WekaConfig":{
        "WekaJar":"CPResources\\weka.jar",
        "MemoryGB": <Memory for weka processes (in GB)>,
        "WekaRunArffCommand": "weka.core.converters.CSVLoader {0}",
        "WekaRunRTCommand":"weka.classifiers.meta.FilteredClassifier -F \"weka.filters.unsupervised.attribute.Remove -R {0}\" -c 1 -t {1} -S 1 -W weka.classifiers.trees.RandomForest -- -P {2} -print -attribute-importance -I {3} -num-slots {4} -K 0 -M 1.0 -V 0.001 -S 0 -num-decimal-places 10",
        "RandomTreeConfig":{
            "RemovedColumns":"2,3,4",
            "RandomTreeCount": 100,
            "BagSizePercentage": 100,
            "MaxCreationParallelism": 4,
            "Classes": "Shared,NonShared"
        }
    }
}
####################################################################

b. Input dirrectory, which is a valid path containing json consolidated files
c. Output directory, which is a valid path (existing) directory on which to write the results.

The output its a set of directories, one for each hash, containing a mapping between hash and queues (one mapping per instance).
Creating a database might be time consuming but it needs to be done one per sample.

2. linearizeDatabase.cmd with the following arguments:

a. Configuration file: A valid path to a json configuration file like specified above.
b. Input directory, whcih is a valid path to an existing database (created using createDatabase.cmd)
c. Num samples (int default=1), indicating how many samples will be created
d. Sample size (int, default=10000) indicating the number of instances in each sample

This will create (c) + 1 csv files in the input directory, containg (c) samples and one csv for all the artifacts. The last csv file should be used to evaluate 
classifiers created with samples.

You can linearize as many times as you want, notice that the output files will have different names (so the older ones wont be deleted).

3. buildClassifiers.cmd with the following arguments:

a. Configuration file: A valid path to a json configuration file like specified above.
b. Input directory, whcih is a valid path to an existing directory with linearized database (created using linearizeDatabase.cmd)

This will take each "-sample" csv file present in the input directory and will create a random forest for each one (stored with extension wtree in the same directory). 
After these files are created, each wtree classifier will be evaluated against the main csv file. You will see the precision, recall and performance metrics for each evaluation.

With this, you can select your best classifier (wtree file), take your queue and machine map and create a ContentPlacementClassifier


