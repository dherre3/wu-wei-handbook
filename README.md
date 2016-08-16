# Wu-Wei Handbook

The [Wu-Wei benchmarking toolkit](https://github.com/Sable/wu-wei-benchmarking-toolkit) is (1) a set of conventions for organizing artifacts related to benchmarking on the file system and describing them using JSON files and (2) a set of commandline utilities for performing common benchmarking tasks with artifacts. The endgoal is to make benchmarking results drastically easier to obtain, replicate, compare, analyze, publish, etc. and to eventually obtain a benchmarking commons similar to other package repositories (ex: Linux distributions or the npm ecosystem).

# Installing the Wu-Wei tools

    git clone https://github.com/Sable/wu-wei-benchmarking-toolkit.git
    cd wu-wei-benchmarking-toolkit
    npm install
    npm link


# Replicate a benchmarking experiment

## Creating a new benchmarking repository

    mkdir my-suite
    cd my-suite
    wu init
    
## Install artifacts

### Installing an existing benchmark

    wu install https://github.com/Sable/polybench-correlation-benchmark.git
    
### Installing an existing compiler
    
    wu install https://github.com/Sable/ostrich-gcc-compiler.git
    
### Installing an existing execution environment

    wu install https://github.com/Sable/ostrich-native-environment.git
    
or 

### Install an existing experiment which defines the previous artifacts as dependencies

    TODO
    
## Verify the installation and setup the current platform

    wu list
    
## Executing all valid combinations of artifacts on the current platform

    wu run
    
## Report execution results (runs)

    wu report
    
## Clear existing builds and runs
    
    wu build --clear
    wu run --clear
    


    
