Creating high-quality implementations for benchmarks is the most time-consuming part of benchmarking and arguably the most important since it the foundation in the evaluation of many other tasks, such as creating new optimizations in compilers or virtual machines, source-code analyzes, comparing various implementations of a programming language, etc. There are many factors that need to be accounted for to make the resulting implementation useful, correct, convenient, and reusable by others. The payoff for a community is the same as for having high-quality publications or libraries readily available: we can stand on other people's shoulders.

In this guide, we introduce the important points for writing high-quality implementations from 3 starting points, from simplest to the most complicated:

1. An existing Wu-Wei implementation in the same language
2. An existing Wu-Wei implementation in a different language (part of an existing benchmark)
3. No existing Wu-Wei implementation

The various considerations are progressively introduced since the later tasks subsume the previous ones by requiring additional considerations.

# 1. Create a new implementation from an existing Wu-Wei implementation in the same language

This is the simplest case. It may be used to try different optimizations techniques by hand to investigate their performance tradeoffs before the creation of a fully automatic transformation in a compiler or execution environment (ex: loop unrolling strategies, vectorization strategies, optimization techniques from optimization guides for various architectures, different object/matrix representations in memory, etc.). It may also be used to try different existing libraries that implement some part of the core computation of a benchmark (ex: various JavaScript libraries from npm that perform numerical computations). You can see an example [here](https://github.com/Sable/lcpc16-analysis/tree/master/benchmarks/backprop/implementations), in which multiple versions of an implementation were compared for the performance evaluation in a publication about automatic vectorization techniques for MATLAB.

There are two manual steps to do, (1) copy and rename the existing implementation directory and (2) change the copy's short-name in the implementation.json file to the same value as the copy directory name. For example, suppose you have an existing c implementation for the template benchmark. Starting from the repository root you would do:

    cd benchmarks/template/implementations
    cp -r c c-modified
    cd c-modified
    open implementation.json # Modify the 'short-name' property to 'c-modified'
    rm -rf .git # (optional) to avoid pushing changes in the wrong repository later...

The files that perform the core computation (the files under the implementation.json 'core-source-files' by convention) can then be modified to obtain a different variation. A new copy of the implementation may be created for each additional interesting variation that is tried.

The original implementation may need to be modified also. In order to keep the different versions consistent with less effort, the copies of the non-core files can be replaced with either hard- or soft-links to the files of the original implementation.

The main advantage of this approach is that all the rest of the Wu-Wei infrastructure and benchmark implementation need not be modified to compare the performance metrics of the variation(s) against the base version. The presence of a new (valid) implementation directory is sufficient for the tools to use it automatically for all the other phases of the benchmark cycle.
    
# 2. Create a new implementation from an existing Wu-Wei implementation (and benchmark) in a different language

Creating a new implementation in a different language for an existing benchmark requires thinking about a few issues in order to make the comparison between the different implementations correct and the new implementation convenient to reuse. To make the task easier, we suggest using existing templates that correctly address those issues as starting points and replace their code with custom code written from scratch or ported from existing benchmark suites.

Every implementation should be self-contained and be stored under the 'benchmarks/*benchmark-name*/implementations/*implementation-name*' directory. The *benchmark-name* and *implementation-name* should correspond to the short-names of each artifact and are also respectively properties of the 'benchmark.json' and 'implementation.json' description files. A new implementation can be bootstrapped using an existing template with the following command from the repository root:

    wu install *template-source* --destination benchmarks/*benchmark-name*/implementations/*implementation-name*

The install tool will automatically update the implementation's 'implementation.json' description file's short-name to *implementation-name* and save the template files under the 'benchmarks/*benchmark-name*/implementations/*implementation-name*' directory to ensure consistency between short-name and the directory. 

Here is a list of currently available templates for various language implementations:

| Language              | Description                               | Source                            |
| :-------------------- | :---------------------------------------- | :-------------------------------- |
| MATLAB                | Relies on the Ostrich Mersenne-Twister random number generator ('ostrich_rand()') available globally. This template is compatible with the [MATLAB](https://github.com/Sable/ostrich-matlab-environment) and [Octave](https://github.com/Sable/ostrich-octave-environment) environments, and the [MATLAB concatenate compiler](https://github.com/Sable/ostrich-matlab-concatenate-compiler).  | https://github.com/Sable/matlab-implementation-template.git |
| JavaScript            | Uses the [precompiled Ostrich Mersenne-Twister random number generator for JavaScript](https://github.com/Sable/ostrich-twister-prng.js/). This template is compatible with the [Node](https://github.com/Sable/ostrich-node-environment), [Chrome](https://github.com/Sable/ostrich-chrome-environment), [Firefox](https://github.com/Sable/ostrich-firefox-environment), [Safari](https://github.com/Sable/ostrich-safari-environment) environments, and the [Browserify compiler](https://github.com/Sable/ostrich-browserify-compiler).  | https://github.com/Sable/js-implementation-template.git |
| C                      | Uses the Ostrich Mersenne-Twister random number generator from the [C implementation common files](https://github.com/Sable/ostrich-c-implementation-common). This template is compatible with the [Native](https://github.com/Sable/ostrich-native-environment) and the [GCC compiler](https://github.com/Sable/ostrich-gcc-compiler).  | https://github.com/Sable/c-implementation-template.git |

If you create a new template for another language, please do a pull request on this handbook with the source (url) of your template to add it to this list. We recommend using a git repository for traceability of the changes but any url may work.

The templates are implemented to show how to make the implementation both correct and convenient to reuse by addressing the following issues. We will use the matlab template example for illustration. You may follow the examples by installing the template in a new repository by doing:

    mkdir implementation-tutorial && cd implementation-tutorial
    wu init
    wu install https://github.com/Sable/benchmark-template.git
    mkdir -p benchmarks/template/implementations # Ensure there is an implementations folder present
    wu install https://github.com/Sable/matlab-implementation-template.git --destination benchmarks/template/implementations/matlab
    
You may open the 'benchmarks/template/implementations/matlab/implementation.json' file in order to follow along. When you do, in the first part of the file you will find the following properties:

        {
            "type": "implementation",
            "short-name":"matlab",
            "description":"Template for matlab implementations",
            "language":"matlab"
            ...
        }

The 'type' property specificies that the rest of the description is for an implementation, the 'short-name' is a unique name used with the various tools to refer to this specific implementation, the 'description' is to provide general explanations about the source of the benchmark what is interesting about this specific implementation. The 'language' property is a short canonical name that specifies the programming language in which the implementation was written. The other properties in the file are discussed as we cover the various topics.

## Highlighting the interesting part of the implementation sources

All benchmarks need to perform setup and teardown steps around the interesting part of the computation in order to produce a repeatable correct result and some metric measurements that will be communicated to the rest of the benchmarking infrastructure. It is convenient in some studies (code instrumentation, run-time profiling, etc.) to differenciate between the interesting and non-interesting part of the computation. 

An implementation can highlight the interesting parts of the computation by listing the interesting files under the 'core-source-files' property of the 'implementation.json' description file. In the matlab template, the 'implementation.json' file list the following files:

        {
            "type": "implementation",
            "short-name":"matlab",
            ...
            "core-source-files": [
               { "file": "./kernel.m"}
            ],
            "runner-source-file": { "file": "./runner.m" },
            "libraries": [
                { "file": "./verify.m" }
            ]
            ...
        }

The uninteresting but necessary code for interfacing the interesting part of the computation with the rest of the infrastructure is in the 'runner-source-file'. Some part of the runner file may be put in separate file(s) under the 'libraries' property for clarity or because it is reused from files common to multiple benchmarks (more on that later when we address the dependencies). 

Note that all file paths are wrapped in a JSON object with a 'file' property. This is a convention used to disambiguate the usage of literal strings for different purposes in the description files. Moreover, the file paths are relative to the parent directory of the description file, the 'implementation.json' parent directory ('benchmarks/template/implementations/matlab/') in this example.

## Passing parameters to an implementation

An implementation needs to specify whether and how it uses the parameters that may vary between different experiments but are common to all implementations of the same benchmark. The first and most common case is the input size used, which by convention may be one of 'small', 'medium, 'large' values for which all benchmarks should provide concrete values.

For any benchmark, the 'small' value means the implementation should execute quickly but we do not really bother about the time spent in the computation. This is used to quickly test the correctness of the implementation (after various automatic transformations during the build phase). The 'medium' value means the implementation should execute long enough for performance measurements to stabilize but still quickly enough to deliver timely results, typically 1-10 seconds. The 'large' value means the input should be scaled up to see how the performance varies with a bigger load.

Since the concrete values are common between different implementations, they are specificed in the 'benchmark.json' description file. In our example, ('benchmarks/template/benchmark.json') you will find the following values:

        {
            "type": "benchmark",
            "short-name":"template",
            ...
            "input-size": {
                "small": 10,
                "medium": 3000,
                "large": 7000
            }
        }

### Using an expand macro for the input-size parameter

When creating a new implementation, to make it consistent with the other existing implementations of the same benchmark, you will simply refer to the same input size value with an 'expand' macro. The expand macro will eventually resolve to the benchmark concrete input value(s) in the implementation's configuration. The path of the expand macro refers to an element of the [configuration](https://github.com/Sable/wu-wei-handbook/blob/master/README.md#configuration-elements) that is used in the run phase. In this example, the '/experiment/input-size' is used as an argument to the implementation's runner and may take the 'small, 'medium', or 'large' values: 

        {
            "type": "implementation",
            "short-name":"matlab",
            ...
            "runner-source-file": { "file": "./runner.m" },
            "runner-arguments": [
                { "expand": "/experiment/input-size" }
            ],
            ...
        }

The Wu-Wei tools will resolve one of the 'small', 'medium', or 'large values to its concrete value that comes from the 'benchmark.json' description file in the configuration of the current phase of the benchmarking cycle.

### Parameterizing an implementation

An experiment may modify the behaviour of an implementation through build-time parameters, which change the operations performed by a compiler during the build phase to produce a runner from the implementation source code, and/or through run-time parameters, which modify the behaviour of the runner that is executed during the run phase.

For most dynamic languages, the implementation parameters are passed at run time to the runner file. For static languages, the implementation of some benchmarks may require parameter(s) at build time. We cover both separately.

#### Run-time parameters

To support run-time parameters, a single-step is required. The 'implementation.json' description file simply needs to define the experiment parameters that should be passed as arguments to the runner with the 'runner-arguments' property. This is the case for the experiment size in the matlab template:

        {
            "type": "implementation",
            "short-name":"matlab",
            "description":"Template for matlab implementations",
            ...
            "runner-source-file": { "file": "./runner.m" },
            "runner-arguments": [
                { "expand": "/experiment/input-size" }
            ],
            ...
        }

The exact mechanism by which the runner arguments are converted to concrete values and passed to an implementation are specific to the execution environment that is used. In most cases, if the implementation language provides facilities for obtaining commandline arguments, the runner will receive the arguments in the same order as defined in the runner-arguments. Otherwise, in most other cases the arguments will be passed as positional arguments to a 'run' or 'runner' function. In both cases, JSON numbers are usually converted to ints/floats and JSON string as strings. For the complete details you should refer to the document of the environment your are using.


#### Build-time parameters


To support build-time parameters two steps are required:

1. The compiler needs to expect a parameter from the implementation (which may be optional)
2. The implementation needs to define the value(s) of that parameter from an experiment's property

This is the case in the [PolyBench/C correlation benchmark's c implementation](https://github.com/Sable/polybench-correlation-benchmark) when used with the [ostrich-gcc-compiler](https://github.com/Sable/ostrich-gcc-compiler).

In this example, the compiler expects an optional 'compilation-flags' parameter from the implementation, which may contain multiple flags. These flags will be passed as arguments to the 'gcc' executable:

        {
            "type": "compiler",
            "short-name":"gcc",
            ...
            "commands": [
                {   "executable-name": "gcc",
                    "options":[
        	            ...
                        { "config": "/implementation/compilation-flags", "optional": true },
                        ...
                    ]
                }
            ]
        }


The PolyBench/C correlation implementation defines a 'compilation-flags' property that prepends the '-D' prefix to the value of the 'input-size' parameter of the experiment that itself resolve to a concrete value defined in the 'benchmark.json' description file.

        {
            "type": "implementation",
            "short-name":"c",
            "description":"Reference C implementation ported from PolyBench/C",
            "language":"c",
            ...
            "compilation-flags": [
               { 
                 "prefix": "-D",
                 "value": { "expand": "/experiment/input-size" }
               },
               ...
            ],
            ...
        }


### Multiple arguments

Sometimes, more than one parameter influence the behaviour of an implementation. In this case, a combination of these parameters needs to be defined for each of the benchmark 'small', 'medium', and 'large' input sizes. Each individual parameter can be defined as a property on an object:

    {
        "type": "benchmark",
        ...
        "input-size": {
            "small": {
                "parameter1": 1,
                "parameter2": 2,
            },
            "medium": {
                "parameter1": 5,
                "parameter2": 10
            },
            "large": {
                "parameter1": 100,
                "parameter2": 500
            }
        }
    }

An implementation can then refers to these parameters with an additional 'path' property with the 'expand' macro. The 'path' property uses the [json-pointer format](TODO-PUT-REFERENCE):

    {
        "type": "implementation",
        ...
        "runner-arguments": [
            { 
                "expand": "/experiment/input-size",
                "path": "/parameter1"
            },
            {             
                "expand": "/experiment/input-size",
                "path": "/parameter2" 
            } 
        ]
        ...
    }

The same technique can be used for build-time parameters.

## Consistent Pseudo-random number generation between languages

For replicability of experiments, the execution of implementations should be deterministic, identical between runs, and consistent between different implementations of the same benchmarks. In order to do so we need:
- Consistent inputs
- Consistent deterministic execution

which should lead to replicable and consistent outputs between runs and implementations.

By checking the output of a run against the expected output, we can detect errors that could be introduced by a compiler during the build phase, errors that are introduced when creating a variation of an implementation by hand, or execution errors that could be introduced by an execution environment. Moreover we can easily detect inconsistencies between implementations in different languages.

In order to achieve those properties we need an implementation pseudo-random number generator for each implementation language that is:

1. Fast to execute in all languages for quick input generation;
2. Consistent (it should produce the same sequence of numbers in all languages);

In addition, it should be easy to maintain and distribute on as many platforms as possible.

We support two pseudo-random number generators (prng), one based on the V8/Octane Benchmarking Suite pseudo-random number generator and the other based on the Mersenne-Twister algorithm. Both support C, JavaScript, and MATLAB. The former is supported for historical reasons but we encourage newer benchmarks to use the latter.

### V8/Octane benchmarking suite algorithm

This algorithm is itself based on the Robert Jenkins' 32-bit integer hash function. The original version we used is available in the [Octane suite's source code](https://github.com/chromium/octane/blob/master/base.js). It replaces the Math.random standard library function of JavaScript with a deterministic alternative.

As part of the work on the [Ostrich benchmarking suite](https://github.com/Sable/Ostrich), we ported back the algorithm to C and ensured the results were consistent between C and JavaScript. We later ported the C version to MATLAB using the [MEX format](http://people.sc.fsu.edu/~jburkardt/m_src/mex/mex.html) for C code, which can be compiled with the MEX compiler proper to the Mathworks' [MATLAB](http://www.mathworks.com/help/matlab/ref/mex.html) and [Octave](https://www.gnu.org/software/octave/doc/interpreter/Getting-Started-with-Mex_002dFiles.html) dialects of the MATLAB language.

Since the Mathworks' MEX compiler introduces many heavy dependencies on different platforms that may require multi-gigabytes downloads in some cases and for which the installation is hard to automate, the usage of this algorithm is deprecated for newer benchmarks in favor of the Mersenne-Twister algorithm.

For reference here are example implementations that use this algorithm:

| Language | Description                                                  | Source(s)                                 |
| :------- | :----------------------------------------------------------- | :---------------------------------------- |
| C        | The [backprop benchmark](https://github.com/Sable/backprop-benchmark) C implementation uses the [Ostric C-implementation common files](https://github.com/Sable/ostrich-c-implementation-common) which includes the [common_rand.c file](https://github.com/Sable/ostrich-c-implementation-common/blob/master/common_rand.c) with the algorithm. | https://github.com/Sable/backprop-benchmark https://github.com/Sable/ostrich-c-implementation-common |
| JS       | The [backprop benchmark](https://github.com/Sable/backprop-benchmark) JavaScript implementation uses the [Ostric JS-implementation common files](https://github.com/Sable/ostrich-js-implementation-common) which includes the [common_rand.js file](https://github.com/Sable/ostrich-js-implementation-common/blob/master/common_rand.js) with the algorithm. | https://github.com/Sable/backprop-benchmark https://github.com/Sable/ostrich-js-implementation-common |
| MATLAB   | This [MATLAB implementation template](https://github.com/Sable/matlab-implementation-template) relies on an execution environment providing a 'createMatrixRandJS' function that calls the mex-compiled C version of the algorithm. The environment can be either of our wrappers for the [MathWorks Matlab virtual machine](https://github.com/Sable/ostrich-matlab-environment) or for the [Octave virtual machine](https://github.com/Sable/ostrich-octave-environment). | https://github.com/Sable/matlab-implementation-template https://github.com/Sable/ostrich-matlab-environment https://github.com/Sable/ostrich-octave-environment|


### Mersenne-Twister algorithm

[This algorithm](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/emt.html) is natively implemented in [multiple languages](https://en.wikipedia.org/wiki/Mersenne_Twister). We verified that it provides consistent results for [MATLAB and Python/Numpy without any external dependency and easy fast support for C and JavaScript](https://github.com/Sable/ostrich-twister-prng). Its compilation relies only on standard widely-available tools for which the installation may be automated.


| Language     | Description                                                  | Source              |
| :-------     | :----------------------------------------------------------- | :------------------------------------ |
| C            | This version is integrated in the common files for c implementations. It is used by the [C implementation template](https://github.com/Sable/ostrich-c-implementation-common) as an automatic dependency, you can look at it for example usage. | https://github.com/Sable/ostrich-c-implementation-common |
| JS           | This version is available as a standalone npm package. It is used by the [JS implementation template](https://github.com/Sable/js-implementation-template). | https://github.com/Sable/ostrich-twister-prng.js/ |
| MATLAB       | This version is part of the execution environments for [MATLAB](https://github.com/Sable/ostrich-matlab-environment) and [Octave](https://github.com/Sable/ostrich-octave-environment) and is callable with the 'ostrich_rand()' function, which follows the same calling conventions as the builtin 'rand()' function. See the [MATLAB template](https://github.com/Sable/matlab-implementation-template) for example usage. | None required for the benchmark implementation |


## Automatic verification of the output's correctness

The automatic verification of output can help ensure that the different variations of implementation all behave consistently except for their performance characteristics. It also allows the automatic validation of compiler transformations. 

The tools can verify the output consistency between different implementation executions automatically. To ensure the mechanism is active, check that the 'benchmark.json' description file defines the expected outputs with the 'expected-output' property:

    {
        "type": "benchmark",
        ...
        "input-size": {
            "small": ...,
            ...
        },
        "expected-output": {
            "small": expected_value
        }
    }

After an execution, the verification will be made against the 'output' property of the JSON output of the implementation runner:

    {
        ...
        "output": *value*
        ...
    }

We suggest computing a 'checksum' value on the output of the kernel and using that checksum value as an output for the benchmark. For integer-based computations, the checksum may be the md5sum (or another checksum function) of all the values of the (multi-dimensional) output. For floating-point operations, the checksum needs to tolerate some rounding differences because the order of operations between different implementations or executions may be different and lead to different outputs. Refer to the templates above for an example that computes a checksum, scales the output, and remove the least-significant part of the result to tolerate errors.     
    

An implementation runner may also test that an execution output corresponds to its expected value with a mechanism of its choice. If the output is invalid, the runner can either return a non-zero error code or throw an exception. The details depend on the execution environment implementation. As this requires more work in every implementations of the benchmark, we favor the automatic approach using the 'expected-output' property. We still support this approach on older benchmarks for backward-compatibility.

## Time measurement on the core computation

The implementation runner is also responsible for providing the time taken for the core computation to execute using the 'time' property on the output object:

    {
        ...
        "time": *time-in-seconds*
        ...
    }

## Factorization of reusable code with dependencies

Multiple implementations in the same language, possibly in different benchmarks, may need to perform similar operations. The operations can then be factored out into reusable libraries. Those libraries can be installed either using a number of options:

1. Supported language-specific package manager (ex: npm for Node.js/JS);
2. Arbitrary list of commands in an install script;
3. Wu-Wei artifact dependency.

We cover each in turn.

### Supported language-specific package manager

At the moment, only the [npm package format for JavaScript](https://docs.npmjs.com/files/package.json) is supported. During installation, if a 'package.json' file is present in the root folder of an artifact, the 'npm install' command will be run from that folder to automatically install all dependencies.

Other package managers may be supported in the future.

### Custom install script

An artifact may perform a custom sequence of steps during installation. It may be used to generate input data, obtain dependencies, etc. To do so, an executable script can be put in the root directory of the artifact named 'install'. The script will be called every time the installation step is invoked from the commandline with 'wu install', it is therefore the responsibility of the script to make sure the same things are not reinstalled if they are already present.

### Wu-Wei dependency

For convenience, when a dependency is either another Wu-Wei artifact or a directory of files, the description files for artifacts may also list their dependencies under a 'dependencies' properties. The 'wu install' tool will retrieve the dependency files from their source, figure out where and how to install from its description file (if present), install transitive dependencies, and move the artifact in its final destination in the repository.

An implementation may use that dependency mechanism to retrieve code that is common to multiple benchmark implementations. It is especially useful for languages which do not provide standard package managers, such as C. The PolyBench/C correlation benchmark shows how the 'utilities' files from the suite can be reused in multiple benchmarks.

First, the code should be factorized in a separate directory. We suggest using a git repository for version control and traceability of changes. We did it by putting it in the [polybench-c-utilities git repository](https://github.com/Sable/polybench-c-utilities).

Second, an implementation may then use it in its dependencies by specifying the destination directory, as in the [polybench-correlation-benchmark](https://github.com/Sable/polybench-correlation-benchmark). The files from the dependency can then be used in the implementation 'libraries' and 'include-directories' for the correct compilation for the benchmark:

    {
        "type": "implementation",
        "short-name":"c",
        ...
        "libraries": [
           { "file": "./utilities/polybench.c" } 
        ],
        "include-directories":[
            { "file": "./utilities/" },
            { "file": "./" }
        ],
        "dependencies": [
            {
                "source": "https://github.com/Sable/polybench-c-utilities.git",
                "destination": "./utilities"
            }
        ],
        ...
    }

## Attribution and Licensing

Creating a high-quality benchmark implementation requires a significant amount of work and the use of the resulting work may be governed by a license. In order to recognize the effort that was contributed by various authors and honor the current licensing, the implementation should have a few additional files that otherwise are not strictly required for performing experiments:

1. The origin of the benchmark should be mentioned in the README file for the implementation
2. The different authors of the implementation should also be mentioned in the AUTHORS file, including the original authors of the implementation if obtained from an existing source
3. If the implementation has been imported from an existing source, the original license of the implementation should be included. Otherwise, a new license that explains how the implementation can be used, modified, and distributed should be included. In both cases, the license text should be in the LICENSE file.


# 3. Create a new Wu-Wei implementation (and benchmark) from scratch

Some additional considerations need to be taken into account if a new benchmark is created. These are covered here.

### Choosing the input-size values

Ideally, the input size chosen should ensure that the fatest implementation available now or in the future runs for long enough for the performance measurements to be meaningful. However, that may conflict with the need for obtaining results on currently available implementations, which may be significantly slower than the fatest possible, in a reasonable time. The benchmark default input sizes may therefore change over time to take into account the evolution of the programming language represented, the speed of their implementation, and the evolution in the speed of hardware.

As a first approximation, we suggest that the input sizes are chosen according to the fastest implementation available at the time the benchmark is created. Here are some guidelines according to the size of the input:

- *small*: The execution time does not matter since the size is used to test for correctness. You should pick the smallest size that still exercises the core computation of an implementation fully;
- *medium*: The fastest implementation should run for 1-10 seconds;
- *large*: The fastest implementation should run for >10 seconds.


#### File-input

There is no convention at the moment for how to use input data coming from a file. We will update this document once we have found simple conventions that work well in many cases.


# Distribution of the new implementation

Once a new implementation has been thoroughly tested, it may be distributed in a number of ways depending on how it is likely to be reused and extended in the future.

## Publication-specific repository for reproducibility of the experiments

In this case, the new implementation might be quite specific to a given research question and tied to particular study in a given (eventual) publication. In this case, the overhead of creating and maintaining a git repository specifically for that implementation is really unjustified. We suggest to create a single paper-specific git repository with all the artifacts used, including the different variations of the benchmark implementations used to derive claims and produce figures. We have done so for an LCPC publication on automatic vectorization of Matlab code and we published all the artifacts used and variations of implementations in this [lcpc2016-analysis repository](https://github.com/Sable/lcpc16-analysis). Some of these implementations were written by hand to investigate the effectiveness of a techniques and others were produced automatically from a tool. The second option may be useful before a tool is fully is integrated as a compiler artifact to be used during the build phase.

## Integration with the original benchmark to be used as a canonical implementation for the language represented

A new implementation of a benchmark in a previously unsupported language may be a good candidate to become a canonical implementation for a benchmark in that language. A canonical implementation is useful to compare different languages and serves as a good starting point for performance optimizations that are specific to the language in other studies. 

To distribute a new implementation for a benchmark as a canonical implementation, ask the maintainer of the benchmark to add it by doing a pull request on the benchmark git repository. The modification in the pull request may either directly add the implementation and its source code into the benchmark repository or add a dependency in the 'benchmark.json' description file that refers to a implementation-specific git repository.

## Benchmark-independent distribution for archiving

Finally, an implementation may be distributed in its own repository independently for archiving or future reference. At least, the benchmark it implements should be listed in its README file and the instructions for installing it alongside an existing benchmark implementations should be provided. The 'install' tool may be used for that purpose with a command such as:

    wu install *implementation-source* --destination benchmarks/*benchmark-name*/implementations/*implementation-name*
    

# Implementation checklist

1. List the core computation (kernel) files in the implementation description ’core-source-files’ property
2. Set the implementation parameters (‘input-size’) either at build-time or run-time from the ‘experiment’ properties
3. Use a pseudo-random number generator that is consistent with the other implementations, if you have a choice preferably use the Mersenne-Twister algorithm
4. Setup automatic verification of the output for some know parameters
5. Output a checksum of the output as an output JSON ‘output’ property
6. Output the time measured as an output JSON ‘time’ property
7. Setup dependencies
8. Add a README listing the source, a LICENSE, and list AUTHORS
9. Choose input-size parameters for a new benchmark that follow the guidelines
10. Distribute your implementation according to the likelihood and context of reuse (as part of a publication-specific experiment, a pull request on an existing benchmark, an independent git repository)
