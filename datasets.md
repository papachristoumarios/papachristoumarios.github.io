---
layout: page
title: Datasets
permalink: /datasets/
---

Here you will find a collection of datasets available on the public domain for various tasks.
I strive to make the data I use for my research open. In case anything is not working please contact me at [papachristoumarios@cs.cornell.edu](mailto:papachristoumarios@cs.cornell.edu).

### Graph Datasets

#### Opinion Dynamics

Data in this category consist of unweighted directed networks where each node has a multi-dimenaional label (0-1 valued) regarding whether (or not) each node endorses a certain opinion. The file `X.edges` contains directed edges between the nodes of the network, and the file `X.feat` contains a dense feature matrix where the first entry corresponds to the corresponding node id.

* [pokec](https://drive.google.com/file/d/1mYF3E5adntiTbwb4EWp4SrAhYMD9yPMU/view?usp=sharing). Derived from [soc-pokec](https://snap.stanford.edu/data/soc-pokec.html). The data contains users of the pokec social network where users with private information have been filtered out. The attributes of each user are derived by looking at his/her corresponding profile interests (described in the original network).
* [github](https://drive.google.com/file/d/1tqP-1uJvmKq96XsGmXCKTaHZV-FXs5i2/view?usp=sharing). Contains data gathered from [GHTorrent](https://ghtorrent.org) with queries described in [this gist](https://gist.github.com/papachristoumarios/8bd40b4543ed44a594ee7be5c490879f) where nodes are github users and attributes are programming languages that the user has programmed at as an owner of a project.

#### Call Graphs

These datasets contain call graphs derived using the [cscout](https://github.com/dspinellis/cscout) tool. Each file represents a directed call graph where each line corresponds to a directed edge between two entities (files, functions etc.).

 * [Linux Kernel 4.21](https://zenodo.org/record/2652487#.YG0GLhNKjlw). The data was used to conduct [this study](https://dl.acm.org/doi/10.1145/3338906.3342483).
 * [Call Graphs of many open-source projects](https://github.com/papachristoumarios/call-graphs). Contains call graphs of multiple open-source projects (described in the README).

### Hypergraph Datasets

#### GHTorrent Datasets

In these datasets the nodes of a hypergraph represent users and hyperedges represent repositories, org members etc. that these users belong to. Each user comes with features (such as number of commits, number of followers etc.) used for experiments for [this paper](https://arxiv.org/abs/2206.00783). We provide the SQL queries to create the datasets based on the [GHTorrent MySQL schema](https://ghtorrent.org/relational.html). The post-processed datasets follow the convention of [these](https://www.cs.cornell.edu/~arb/data) hypergraph datasets.

 * [Datasets & MySQL Queries](https://doi.org/10.5281/zenodo.6639983).

### Other resources

 * Network Science
   * [SNAP](http://snap.stanford.edu/)
   * [Network Repository](http://networkrepository.com/index.php)
   * [GHTorrent](https://ghtorrent.org/)
   * [UCINET](https://networkdata.ics.uci.edu/resources.php)
   * [Austin Benson's Data Webpage](https://www.cs.cornell.edu/~arb/data/)
 * Computational Geometry & Computational Biology
   * [BiGG Models](http://bigg.ucsd.edu/)
   * [Netlib LPs](https://www.netlib.org/lp/data/index.html)
 * Dataset Search Engines
   * [data.world](https://data.https//data.world/)
   * [Dataset Search](https://datasetsearch.research.google.com/)  
   * [Kaggle](https://www.kaggle.com/)
