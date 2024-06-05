
# SAG-RRG Replanning Implementation

This repository contains the implementation of the SAG-RRG replanning algorithm for my Master's thesis project. The algorithm is based on the paper ***Semantic Abstraction-Guided Motion Planning for scLTL Missions in Unknown Environments*** by Kush Grover, Fernando S. Barbosa, Jana Tumova and Jan Kretinsky. 

The implementation was originally developed by Kush Grover which can be found at https://github.com/kushgrover/UnVeiL.git. The implementation was then modified by me to include the replanning strategy.

The implementation is written in Java. It makes use of binary decision diagrams (BDDs) package JavaBDD for storing and manipulating the product automaton, the labels of each node in the RRG, and the bias list. The undirected RRG graph is encoded as ausing the JGraphT library. The spatial indexing library RTree is used for spatial indexing and faster querying of nearest neighbours. The owl library is used for converting an LTL formula to an equivalent automaton and the jhoafparser is used for parsing the automaton file.


**Note**: the implementation of the replanning strategy is not optimized and therefore the program runs very slow in comparison to before the changes were made. There are several improvements that could be made to make the code more efficient, but with the time limit imposed by the thesis, I was not able to implement them.  Further, the way the code is set up in its current state is not optimal. I have not implemented a way to save and load the RRG graph after the exploration phase, or a better way of reading and handling the labelling files. This means that in order to apply the replanning strategy, the exploration using the original SAG-RRG always have to be runs before the replanning strategy can be applied. Also, to reexplore the environment with displaced objects, the program has to be run again, and a command line switch has to be included. This is not optimal and it is something I would like to improve.


- Requirements: Java 15.0.2 or higher
- Usage on Linux: (Tested on Ubuntu 20.04.2 and higher)




## Generating environment files

For generating a random 6-room office like environment, use:
```console
python3 random_env_gen.py $envDir numBins envSize envComplexity
```

- $envDir: Directory where the environment files will be stored.
- numBins: Number of bins [0,6] in the environment that should be displaced.
- envSize: Size of environment [small, large]. The small environment is 3m by 6m, the large one is 6m by 12m. At the moment, only large is supported.
- envComplexity: decides what labels are present in the environment. Inputs to choose from are [simple, complex]. Simple only has labeled objects *bin* and *table* and complex has additional labels *corner* and *door*.

This will generate the following files:
<ol>
<li>$envDir/file_name.env: This file defines walls and obstacles, also the initial position of the robot.</li>

<li>$envDir/file_name.lb: This file defines the areas in which the labels hold.</li>

<li>$envDir/file_name.pr: This file contains the scLTL formula.</li>

<li>$DIR/file_name_moved.lb: This file defines the areas in which the labels hold after bins have been displaced. </li>

<li>$DIR/file_name_moved_dist.txt: This file contains how far each bin has beed displaced, this is used for experiments in the Thesis and is not actually used for anythin in the implementation.</li>
</ol>

where `file_name=random_numBins_envType_envComplexity`


Example where 3 bins are displaced in a large environment with simple labelling:
```console
python3 random_env_gen.py environments/ 3 large simple
```




## Running the planner using precompiled .jar file


The project has been compiled in a .jar file. The file is located in the root directory of the project and is named `planning.jar`. It contains all the dependencies required to run the planner and can be run from the command line.

The code requires one argument, which is the path to the environment file. However, there are several command line switches available:
<ol>
<li> --output-dir [dir] : For choosing where output is stored, default directory is temp/</li>
<li> --sensing-radius [value] : Sensing radius of the robot. </li>
<li> --cell-size [value] : Size of each cell used in discretization</li> 
<li> --batch-size [value] : Size of each batch </li>
<li> --bias-prob [value] : How much biasing should be done on sampling</li>
<li> --property-file [file_name].hoa : Automaton representing the property in HOA format. If specified, .pr file will 
be ignored </li>
</ol>


There are three verisons of the planner:
<ol>
<li> Original SAG-RRG: no flag needed</li>
<li> Original SAG-RRG followed by replanning on the environment with displaced objects: --do-patching </li>
<li> Original SAG-RRG on the environment with displaced objects: --do-reexploration</li>
</ol>



To run the original SAG-RRG:
```console
java -Djava.library.path=../implementation/lib/ -jar planning.jar environments/file_name --output-dir $outputDir/ --sensing-radius 1.5
```

To run the SAG-RRG with replanning:
```console
java -Djava.library.path=../implementation/lib/ -jar planning.jar environments/file_name --output-dir $outputDir/ --sensing-radius 1.5 --do-replanning
```

To run the original SAG-RRG on the environment with displaced objects:
```console
java -Djava.library.path=../implementation/lib/ -jar planning.jar environments/file_name_moved --output-dir $outputDir/ --sensing-radius 1.5 --do-reexploration
```





## Running experiments

For running the experiments of the thesis, the script `patching_rnd_experiments.sh` can be used. The script runs the planner on a number of random environments and repeats the experiment a number of times. The results are stored in a directory named `results/patch_vs_reexpl`.

```console
./patching_rnd_experiments.sh 10 5 large simple
```
First number specifies how many random environments to be used for each number of displaced bins. Here this means that for one displaced bin there will be 10 random environments generated, for two displaced bins the same, etc.. The second number specifies how many repetitions of each environment should be run, here each environment will be run 5 times. The results will be written to a directory named `results/patch_vs_reexpl`.



The python files `compute_rnd_results.py` and `plot_rnd_results.py` are used to compute the results and plot them. The `compute_results.py` script computes the average path length and time taken for each environment and stores the results in a .csv file. The `plot_results.py` script reads the .csv file and plots the results.

```console
python3 compute_rnd_result.py $outputDir/ > $resultDir/all_results.txt
python3 plot_rnd_results.py $outputDir/ $resultDir/
```

There are also scripts for plotting the resulting paths and the environment. The `plot_paths.py` script plots the paths taken by the robot in the environment during the mission. 

```console
python plot_paths.py file_name $outputDir/ $figuresDir/
```
Where file_name is the name of the environment file, $outputDir is the directory where the output is stored and $figuresDir is the directory where the figures are stored.



