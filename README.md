# State Machine Testing of TLS protocol implementations

This repository contains a simple setup for learning Mealy machine models of different TLS implementations using the L* algorithm.
It makes use of [LearnLib](http://learnlib.de/), a library for active model learning. On top of that, we use the learning software made by Joeri de Ruiter.

## Setup

Download and install the following before starting:
* Java SDK 1.8
* Graphviz, which contains the `dot` utility for visualizing the learned models, you need the command line tools.
  * For Linux users, install using brew or apt-get or another package manager.
```
apt-get install graphviz
```
  * For Windows users, download it from their [website](http://graphviz.org/download_windows.php). After installation, make sure to add a PATH environment variable like this:
``` 
 C:\Program Files (x86)\Graphviz2.38\bin
```
