# State Machine Testing of TLS protocol implementations

This repository contains a simple setup for learning Mealy machine models of different TLS server implementations using the L* algorithm.
It makes use of [LearnLib](http://learnlib.de/), a library for active model learning. On top of that, we use the [StateLearner](https://github.com/jderuiter/statelearner) software made by Joeri de Ruiter.
After the installation and setup, we will show how to run different TLS servers locally and how to test these implementations.

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
  * For Windows users, download it from the [website](http://graphviz.org/download_windows.php). After installation, make sure to add a PATH environment variable pointing to the install location: `C:/Program Files (x86)/Graphviz2.38/bin`.

* Clone the [StateLearner](https://github.com/jderuiter/statelearner) repo and either follow the build instructions of that repo, or use an IDE like Eclipse:
  * `Import > Existing Maven Projects > [location of the StateLearner] > Finish`
  * `Run As > Maven Build`
  * `Run As > Maven Install` (requires the use of a __JDK__ instead of a JRE)

## Setup
You will first setup the state learner and then you can choose (one of) the different TLS implementation servers to use the learner on.

* Locate the example properties file in `StateLearner/examples/configuration` and overwrite it with the properties file of this repo. You probably want to change a number of lines:
  * keystore_filename (line 5): The path to the example keystore
  * output_dir (line 10): The output folder name
  * alphabet (line 60): All message types the State Learner is allowed to send to the server. On line 36, you can see all supported messages. The larger the used alphabeth, the longer the learning phase will take (about exponentially).
  * port (line 64): The port the current TLS implementation-under-test listens on
  
## Run the State Learner
This can be done using the command line, or via an IDE like Eclipse:
  * `Run As > Run Configurations > Arguments (tab) > Program arguments (field) = examples/configuration/config.properties`
  * `Run As > Run Configurations > Main class (field) = nl.cypherpunk.statelearner.Learner`
  * `Run As > Java Application`
  
It will not work yet, because there is no TLS server to connect to. You can setup one or more TLS servers using the following steps. As these servers often only work on Linux, Windows users should set them up in a virtual machine and forward the required TCP ports to the Windows host. For VBox users, port forwarding can be done like [this](http://stackoverflow.com/questions/9537751/virtualbox-port-forward-from-guest-to-host).
  
### BearSSL (Not for Win users)
* Clone the [BearSSL](https://github.com/nogoegst/bearssl.git) repo and follow the build instructions.
* Go to `bearssl/samples`, compile and run the demo server on port 10000 (for instance).
```
$ cd bearssl/samples
$ gcc server_basic.c -I../inc/ -L../build/ -lbearssl -o server_basic
$ ./server_basic 10000
```

### OpenSSL (Not for Win users)

* [Download](https://www.openssl.org/source/old/) the required OpenSSL version. 
* Untar, build and install the server (some troubleshooting [tips](http://stackoverflow.com/questions/16488629/undefined-references-when-building-openssl)):
```
$ tar -xf openssl-x.x.x.tar.gz
$ cd openssl-x.x.x && 
$ su -i 
$ make clean && ./config zlib 
$ make && make install
```
* Finally, setup and start the server on port 10000 (for instance).
```
$ openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
$ openssl s_server -key key.pem -cert cert.pem -accept 10000 -www
```
* You can test it by going to https://localhost:10000

### LibreTLS (Not for Win users)
TODO

### GnuTLS (Not for Win users)
TODO

### WolfSSL (Not for Win users)
MAYBE TODO

### BoringSSL (Not for Win users)
MAYBE TODO

## Results
After setting up everything, the State Learner will try to build a state machine of the choosen TLS server implementation. This results in a `.dot` file and `.pdf` showing the hypotheses and final state machines. 
The visualizations of these graphs are very large due to the large number of input and output types (the alphabet). Therefore, we merged equivalent state changes and labeled them 'Other' or 'All'.
We included a state machine graph visualization for every listed TLS server implementation. (TODO)

## Notes
* Windows 10 and Ubuntu 14.04 are used for this setup.
* TLS clients could also be tested using the StateLearner, but we will not go into that. 
* The installation and setup of most TLS server implementations, especially the relatively unknown ones, proved to be quite a hassle. Therefore, despite the fact that the steps shown above worked for us, it is quite likely that you need some additional/alternative steps to get it working properly.
* Installing different versions of an TLS implementation alongside is difficult to get working properly. That is why we removed one version before installing the other.

