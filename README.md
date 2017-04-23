# Testing TLS protocol implementations using State Machine Learning (SML) 

This repository contains a setup for learning (Mealy) State Machine models of different TLS server implementations by using the L* algorithm.
It makes use of [LearnLib](http://learnlib.de/), a library for active model learning. On top of that, we use the [StateLearner](https://github.com/jderuiter/statelearner) software made by Joeri de Ruiter. 
The work is an extention of his [research](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-de-ruiter.pdf) on this topic. More specifically, we will examine the state machines of current TLS implementations compared to their older versions which were tested in the research paper. 
Next to that, some additional implementations will be tested and compared. Every time we mention 'the research' in this manual, we refer to that paper.
After the installation and setup, we will show how to run different TLS servers locally and how to test these implementations. Then, we will show and and discuss the resuling state machines. Finally, we will conclude.

## Installation

Download and install the following before starting:
* Java SDK 1.8
  * For Linux users, install using apt-get or another package manager:
```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```
  * For Windows users, download it from the Oracle [website](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

* Graphviz, which contains the `dot` utility for visualizing the learned models, you need the command line tools.
  * For Linux users, install using apt-get or another package manager:
```
$ sudo apt-get install graphviz
```
  * For Windows users, download it from the [website](http://graphviz.org/download_windows.php). After installation, make sure to add a PATH environment variable pointing to the install location: `C:/Program Files (x86)/Graphviz2.38/bin`.

* Clone the [StateLearner](https://github.com/jderuiter/statelearner) repo and either follow the build instructions of that repo, or use an IDE like Eclipse:
  * `Import > Existing Maven Projects > [location of the StateLearner] > Finish`
  * `Run As > Maven Build`
  * `Run As > Maven Install` (requires the use of a __JDK__ instead of a __JRE__)

## Setup
You will first setup the state learner and then you can choose (one of) the different TLS implementation servers to use the learner on.

### Setup and run the State Learner
* Locate the example properties file in `StateLearner/examples/configuration` and overwrite it with the properties file of this repo. You probably want to change a number of lines:
  * keystore_filename (line 5): The path to the example keystore
  * output_dir (line 10): The output folder name
  * alphabet (line 60): All message types the State Learner is allowed to send to the server. On line 36, you can see all supported messages. The larger the used alphabeth, the longer the learning phase will take (about exponentially).
  * port (line 64): The port the current TLS implementation-under-test listens on
  
* Run the State learner using the command line, or via an IDE like Eclipse:
  * `Run As > Run Configurations > Arguments (tab) > Program arguments (field) = examples/configuration/[config name].properties`
  * `Run As > Run Configurations > Main class (field) = nl.cypherpunk.statelearner.Learner`
  * `Run As > Java Application`
  
It will not work yet, because there is no TLS server to connect to. You can setup one or more TLS servers using the following steps. **As these servers often only work on Linux**, Windows users should set them up in a virtual machine and forward the required TCP ports to the Windows host. For VBox users, port forwarding can be done like [this](http://stackoverflow.com/questions/9537751/virtualbox-port-forward-from-guest-to-host).
* You can test the TLS servers by going to https://localhost:10000. After ignoring the self-signed-certificate warnings, you should see a test page in most cases.

### OpenSSL
* [Download](https://www.openssl.org/source/old/) the required OpenSSL version. 
* Untar, build and install the server (some troubleshooting [tips](http://stackoverflow.com/questions/16488629/undefined-references-when-building-openssl)):
```
$ tar -xf openssl-x.x.x.tar.gz
$ cd openssl-x.x.x
$ su -i 
$ make clean && ./config zlib 
$ make && make install
```
* Finally, setup and start the server on port 10000 (for instance):
```
$ openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
$ openssl s_server -key key.pem -cert cert.pem -accept 10000 -www
```

### BearSSL
* Clone the [BearSSL](https://github.com/nogoegst/bearssl.git) repo and follow the build instructions. At the end, the file `/build/libbearssl.so` should have been generated.
* Compile, export the library and run the demo server on port 10000 (for instance):
```
$ cd bearssl/samples
$ gcc server_basic.c -I../inc/ -L../build/ -lbearssl -o server_basic
& sudo cp ../build/libbearssl.so /usr/lib
& ldconfig
$ ./server_basic 10000
```

### GnuTLS
* Install the required libraries:
  * [libgmp](https://gmplib.org/)
  * [libnettle](http://www.lysator.liu.se/~nisse/nettle/)
```
$ sudo apt-get instal -y nettle-dev nettle-bin
```
* Either install the optional libraries, or exclude them during the installation of GnuTLS (using the argument --without-[library name]):
  * [libtasn1](https://www.gnu.org/software/libtasn1/)
  * [p11-kit](https://p11-glue.freedesktop.org/p11-kit.html)
* [Download](http://www.gnutls.org/download.html) the required GnuTLS version.
* Untar, build and install the server:
```
$ tar -xf gnutls-x.x.x.tar.gz
$ cd gnutls-x.x.x
$ su -i
$ ./configure && make && make install
```
* Create server certificates and start the server on port 10000 (for instance) (some troubleshooting [tips](https://www.gnutls.org/manual/html_node/gnutls_002dserv-Invocation.html)):
```
$ certtool --generate-privkey > x509-ca-key.pem
$ echo 'cn = GnuTLS test CA' > ca.tmpl
$ echo 'ca' >> ca.tmpl
$ echo 'cert_signing_key' >> ca.tmpl
$ certtool --generate-self-signed --load-privkey x509-ca-key.pem --template ca.tmpl --outfile x509-ca.pem
$ gnutls-serv -p 10000 --http --x509cafile x509-ca.pem --x509keyfile x509-server-key.pem --x509certfile x509-server.pem
```

### LibreTLS / WolfSSL / BoringSSL
MAYBE TODO

## Results
After setting up everything, the State Learner will try to build a state machine of the choosen TLS server implementation. This results in a `.dot` file and `.pdf` showing the hypotheses and final state machines. 
The visualizations of these graphs are very large due to the large number of input and output types (the alphabet). For that reason, and because the mentioned research uses the same edge-reduction method, we merged equivalent state changes and labeled them 'Other' or 'All'.
We included a state machine graph visualization for a number of specific TLS server implementations and will compare these to the research paper as well as to each other.
The main authentication path will be colored green and any seemingly strange or redundant states or edges are colored orange.

### OpenSSL 1.0.2
As a first experiment, we tried to reproduce one of the results of the aformentioned research. 
This research paper did not include the state machine of this TLS implementation, thus we can only compare aspects like the number of states and learning time with the research. Below, the generated state machine is shown:
![OpenSSL 1.0.2 state machine diagram](/graphs/OpenSSL_1.0.2.png?raw=true "OpenSSL 1.0.2 state machine diagram")
As you can see, the state machine includes 7 states, just like in the research. State 0 to 4 are TLS handshake states, state 5 is the authenticated state and state 2 is the connection closed state. 
Learning the state machine took about the same time as the research: 6 minutes. Some interesting findings include:
* In state 0, when a ChangeCipherSpec, ApplicationData(Empty) or HeartbeatMessage(Empty) is sent, the connection will always be closed without responding with the appropriate alert 'Unexpected Message'.
* In state 5, when a ChangeCipherSpec is sent, the server goes the seemingly redundant state 6 after which every message yields the alert 'Bad Record MAC' and server will go to state 2. This could indicate that in state 6, the server does not use the same MAC key/scheme as the client.  
* In state 5, when a ClientHello(DHE,RSA) is sent, the server returns the 'Handshake Failure' alert which would indicate that the sender was unable to negotiate an acceptable set of security parameters given the options available, but this is not the case: it is an alert 'Unexpected Message'.
* In state 1 to 5, the server happily accepts empty data packets, but except for when in the authenticated state 5, this should lead to state 2.
* The heartbeat requests are ignored in every state. This might be the solution of the software to the Heartbleed vulnerability.

### OpenSSL 1.1.0e
This is the newest stable OpenSSL version which we will compare with the older version above. Below, the generated state machine is shown:
![OpenSSL 1.1.0e state machine diagram](/graphs/OpenSSL_1.1.0e.png?raw=true "OpenSSL 1.1.0e state machine diagram")
As you can see, the state machine now includes 6 states instead of 7. State 0 to 4 are TLS handshake states, state 5 is the authenticated state and state 2 is the connection closed state. 
Learning took about 8 minutes. Some interesting findings compared to OpenSSL 1.0.2 are:
* In state 0, every unexpected message will now result in the appropriate alert 'Unexpected Message'.
* There is no longer a state 6. This seems to prove that this state was redundant in the older OpenSSL implementation.
* In state 5, when a ClientHello(DHE,RSA) is sent, the server still responds with an alert 'Handshake Failure' instead of an alert 'Unexpected Message'.
* Now, the server only accepts empty ApplicationData in the authenticated state. This shows that the acceptance of this messages in the older OpenSSL implementation was strange behavior.

### BearSSL 0.4
This implementation is relatively new and unknown as it is only 'Alpha' state software. That is why it is not tested in the research and proved to be quite difficult to get accurate results from which we will discuss later. 
It is interesting, because on its website it promises to be a very simple and secure implementation of TLS. Below, the generated state machine is shown:
![BearSSL 0.4 state machine diagram](/graphs/BearSSL_0.4.png?raw=true "BearSSL 0.4 state machine diagram")
As you can see, the graph contains 6 states. The shown test took about 18 minutes, but often it was still running after 5 hours, we will explain later later why this was the case. Some interesting findings are:
TODO

### GnuTLS 3.5.9
This is the newest stable version of GnuTLS. The research also looked into GnuTLS, but these were much older versions (3.3.8 and 3.3.12). We will compare the results to these older versions. Below, the generated state machine is shown:
![GnuTLS 3.5.9 state machine diagram](/graphs/GnuTLS_3.5.9.png?raw=true "GnuTLS 3.5.9 state machine diagram")
As you can see, the graph contains 6 states. The shown test took about 7 minutes, which was much shorter than that of versions 3.3.8 (45 min) and about the same as version 3.3.12 (9 minutes). Some interesting findings are:
TODO

### LibreTLS / WolfSSL / BoringSSL
MAYBE TODO

### Conclusions
TODO

## Notes
* Windows 10 and Ubuntu 14.04 are used for this setup.
* TLS clients could also be tested using the StateLearner, but we will not go into that. 
* The installation and setup of most TLS server implementations, especially the relatively unknown ones, proved to be quite a hassle. Therefore, despite the fact that the steps shown above worked for us, it is quite likely that you need some additional/alternative steps to get it working properly.
* Installing different versions of an TLS implementation alongside is difficult to get working properly. That is why we removed one version before installing the other.

