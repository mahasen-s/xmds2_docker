# XMDS2 Docker
If you can install docker, you can run `xmds2`

## Installation
### Linux/MacOS
You should have Docker (https://docs.docker.com/install/) and Git (https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed

1. Clone this repository using `git clone https://github.com/mahasen-s/xmds2_docker.git`
2. Choose one of the following BLAS distributions:

    a. GSL: Use the GNU Scientific Library (GSL). Try this option first
    b. MKL: Use the Intel Math Kernel Library (MKL)
    c. ATLAS: Use the Automatically Tuned Linear Algebra System (ATLAS). You need to make sure that OS-level throttling of the CPU is disabled. Read the Dockerfile for more information. In general, do not use this

3. Build the image of your selected BLAS distro using the Dockerfile in the corresponding folder, e.g. 
```
cd ./xmds2_docker/GSL
docker build -t xmds .
```
4. Add a shortcut to your `/.bashrc` and reload environment 
```
cd ..
cat script>>~/.bashrc
source ~/.bashrc
```
5. Test the installation. You may need to supply the full `.xmds` filepath, e.g.
```
xmds2 /home/mahasen/xmds2_docker/test/lorenz.xmds
```
