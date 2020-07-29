# Phenomiser

This is new implementation of the Phenomiser algorithm published in https://www.sciencedirect.com/science/article/pii/S0002929709003991

You can make a query programmatically using the Phenomiser class. Alternatively, you can build a jar file with all dependencies and run from the command line.


##Quick start

1. Precompute 
Phenomiser need to simulate query with sets of HPO terms to compute an empirical similarity score distributions for each disease. 
We use **-debug** mode to compute score distributions for 50 randomly selected diseases. 
You should omit this argument if you want to precompute data for all diseases. Be aware: it will take a long time complete. 
Read below for details. 
- Download the latest hp.obo file from http://purl.obolibrary.org/obo/hp.obo
- Download the latest annotation file from http://compbio.charite.de/jenkins/job/hpo.annotations.2018/lastSuccessfulBuild/artifact/misc_2018/phenotype.hpoa

Use the debug flag only to do the sampling for only 50 diseases for testing purposes.

```
java -jar phenomiser-cli.jar precompute
-hpo ${path to}hp.obo
-da ${path to}phenotype.hpoa
--sampling 1 10
[--debug]
```

To do the full precomputation, reserve sufficient memory, e.g., ``-Xmx32g``

You should be able to find a **Phenomiser_data** created for you under your home directory. The folder contains precomputed data that will be used in the following steps.

2. Query with HPO terms. This allows you to rank diseases based on their similarities with the query terms that you provided.

```
query
-hpo ${path to}hp.obo
-da ${path to}phenotype.hpoa
-query HP:0003074,HP:0010645,HP:0001943
```

3. Grid search. This command allows you to simulate queries with a i signal (HPO terms belonging to a certain disease) and j noise (random HPO terms not belonging to the same disease), where i and j can be specified by setting the **-signal** (max of i) and **-noise** (max of j).

```
grid
-hpo ${path to}hp.obo
-da ${path to}phenotype.hpoa
-signal 2
-noise 1
```

4. Phenopacket analysis

Run Phenomiser analysis across a collection of Phenopackets. 

```
  $ java -jar PhenomiserApp.jar phenopacket \\
    -hpo /home/whoever/wherever/data/hp.obo \\
    -da /home/whoever/wherever/data/phenotype.hpoa \\
    -cachePath /home/whoever/wherever/data/phenomiser/ \\
    -db OMIM \\
    --phenopacket /home/whoever/wherever/ppacket
```

This will output a file called phenomiser-results.txt with the rankings of the correct diagnoses. 


The output is a matrix where the rows are the number of i and columns the number of j. The value is the percentage (range 0 to 1) of simulations where Phenomiser corrected ranked the disease of target number 1. 

### Help

Run the app with "-h" to print out a list of all arguments:

```
Usage: java -jar PhenomiserApp.jar [options] [command] [command options]
  Options:
    -h, --help
      display this help message
  Commands:
    precompute      Precompute similarity score distributions
      Usage: precompute [options]
        Options:
          -cachePath, --cachePath
            specify the path to save precomputed data
         (...)
```

### How to set precompute parameters 

We recommend running this command on a cluster, and download the cached folder to your personal computer for query (you still need a large RAM to read in the serialized data).

Two parameters are critical: 

**numThreads** Increase the number if you can request more CPU. We tested 40 CPU and can finish the job in about 20 hours.

**sampling** We recommend using the default value of 1 - 10. There is no need to increase the max value above 10, as the score distributions will not change much above 10. If you change the numbers, you have to make sure that the number of query terms fall within the range.

### How to access cached data

Default caching data is under HOME/Phenomiser_data. You can use "-cachePath" if you want to overwrite the default setting.
