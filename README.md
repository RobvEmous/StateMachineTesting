# State Machine Testing of TLS protocol implementations

This repository contains a simple setup for learning Mealy machine models of different TLS implementations using the L* algorithm.
It makes use of [LearnLib](http://learnlib.de/), a library for active model learning. On top of that, we use the [StateLearner](https://github.com/jderuiter/statelearner) software made by Joeri de Ruiter.
After the install ation and setup, we will show how to run different TLS servers locally and how to test these implementations.

## Installation

Download and install the following before starting:
* Java SDK 1.8
  * For Linux users, install using apt-get or another package manager.
```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```
* For Windows users, download it from the Oracle [website](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

* Graphviz, which contains the `dot` utility for visualizing the learned models, you need the command line tools.
  * For Linux users, install using apt-get or another package manager.
```
$ apt-get install graphviz
```
  * For Windows users, download it from the [website](http://graphviz.org/download_windows.php). After installation, make sure to add a PATH environment variable pointing to the install location: `C:\Program Files (x86)\Graphviz2.38\bin`.
```
* Clone the [StateLearner](https://github.com/jderuiter/statelearner) repo and follow the build instructions.

## Setup

* Locate the example properties folder in `StateLearner\examples\configuration`


### OpenSSL (Not for Win users)

* [Download](https://www.openssl.org/source/old/) the required OpenSSL version. 
* Untar it using:
```
$ tar -xf openssl-x.x.x.tar.gz
```
* Build and install it using (requires root access):
```
$ cd openssl-x.x.x && 
$ su -i 
$ make clean && ./config zlib 
$ make && make install
```