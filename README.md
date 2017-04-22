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
  * For Windows users, download it from the Oracle [website](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

* Graphviz, which contains the `dot` utility for visualizing the learned models, you need the command line tools.
  * For Linux users, install using apt-get or another package manager.
```
$ sudo apt-get install graphviz
```
  * For Windows users, download it from the [website](http://graphviz.org/download_windows.php). After installation, make sure to add a PATH environment variable pointing to the install location: `C:\Program Files (x86)\Graphviz2.38\bin`.

* Clone the [StateLearner](https://github.com/jderuiter/statelearner) repo and either follow the build instructions on that repo, or use an IDE like Eclipse:
  * `Import > Existing Maven Projects > [location of the StateLearner] > Finish`
  * `Run As > Maven Build`
  * `Run As > Maven Install (requires the use of a __JDK__ instead of a JRE)`


## Setup
You will first setup the state learner and then you can choose (one of) the different TLS implementation servers to use the learner on.

* Locate the example properties file in `StateLearner\examples\configuration` and overwrite it with the provided properties file. You probably want to change a number of lines:
  * keystore_filename (line 5): The path to the example keystore
  * output_dir (line 10): The output folder name
  * port (line 64): The port the current TLS implementation-under-test listens on
  
## Run the tester
This can be done using the command line, or via an IDE like Eclipse:
  * `Run As > Run Configurations > Arguments (tab) > Program arguments (field) = examples/configuration/config.properties`
  * `Run As > Run Configurations > Main class (field) = nl.cypherpunk.statelearner.Learner`
  * `Run As > Java Application`
  
It will not work yet, because there is no TLS server to connect to. You can setup one or more TLS servers using the following steps. As these servers often only work on Linux, Windows users should set them up in a virtual machine and forward the required TCP ports to the Windows host. For VBox users, port forwarding can be done like [this](http://stackoverflow.com/questions/9537751/virtualbox-port-forward-from-guest-to-host).
  
### BearSSL (Not for Win users)

### OpenSSL (Not for Win users)

* [Download](https://www.openssl.org/source/old/) the required OpenSSL version. 
* Untar, build and install the server (requires root access):
```
$ tar -xf openssl-x.x.x.tar.gz
$ cd openssl-x.x.x && 
$ su -i 
$ make clean && ./config zlib 
$ make && make install
```
* Finally, setup and start the server on port 10000 (for instance).
```
& openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
$ openssl s_server -key key.pem -cert cert.pem -accept 10000 -www
```
* You can test it by going to https://localhost:10000



