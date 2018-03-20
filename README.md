# Bifrost 

| **`CPU build`** | **`Coverage`** | 
|----------------------|----------------------|
|[![Travis](https://travis-ci.org/ledatelescope/bifrost.svg?branch=master)](https://travis-ci.org/ledatelescope/bifrost) | [![Coverage Status](https://coveralls.io/repos/github/ledatelescope/bifrost/badge.svg)](https://coveralls.io/github/ledatelescope/bifrost) |

A stream processing framework for high-throughput applications.

### [Bifrost Documentation](http://ledatelescope.github.io/bifrost/)
### [Bifrost Roadmap](ROADMAP.md)

## Your first pipeline

Example pipelines can be found in the `testbench/` directory. For example, here's a snippet  
that reads data from a binary file, copies it to the GPU, runs an FFT, then writes the
output back to disk:

```python

# Get a list of binary data files
filenames   = glob.glob('testdata/*.bin')

# Setup pipeline
b_read      = BinaryFileReadBlock(filenames, window_len, 1, 'cf32', core=0)
b_copy      = CopyBlock(b_read, space='cuda', core=1, gpu=0)
b_fft       = FftBlock(b_copy, axes=1, core=2, gpu=0)
b_out       = CopyBlock(b_fft, space='system', core=3)
b_write     = BinaryFileWriteBlock(b_out, core=4)

# Run pipeline
pipeline = bfp.get_default_pipeline()
print pipeline.dot_graph()
pipeline.run()
```

And here's an example that reads a WAV audio file, generates a spectrum, and writes the output to a filterbank file:

```python
import bifrost as bf

data = bf.blocks.read_wav(['file1.wav', 'file2.wav'], gulp_nframe=4096)
data = bf.blocks.copy(data, space='cuda')
data = bf.views.split_axis(data, 'time', 256, label='fine_time')
data = bf.blocks.fft(data, axes='fine_time', axis_labels='freq')
data = bf.blocks.detect(data, mode='jones')
data = bf.blocks.accumulate(data, 2)
data = bf.blocks.transpose(data, ['time', 'pol', 'freq'])
data = bf.blocks.copy(data, space='cuda_host')
data = bf.blocks.quantize(data, 'i8')
bf.blocks.write_sigproc(data)

pipeline = bf.get_default_pipeline()
pipeline.shutdown_on_signals()
pipeline.run()
```

<!---
Should put an image of this pipeline here.
-->
## Feature overview

 * Designed for sustained high-throughput stream processing
 * Python and C++ APIs wrap fast C++/CUDA backend
 * Native support for both system (CPU) and CUDA (GPU) memory spaces and computation

 * Main modules
  - Ring buffer: Flexible and thread safe, supports CPU and GPU memory spaces
  - Transpose: Arbitrary transpose function for ND arrays

 * Experimental modules
  - UDP: Fast data capture with memory reordering and unpacking
  - Radio astronomy: High-performance signal processing operations

## Installation

### C library

Install dependencies:

    $ sudo apt-get install exuberant-ctags

### Python interface

Install dependencies:

 * numpy
 * matplotlib
 * contextlib2
 * pint
 * ctypesgen

```
$ sudo pip install numpy matplotlib contextlib2 pint git+https://github.com/davidjamesca/ctypesgen.git@3d2d9803339503d2988382aa861b47a6a4872c32
```

### Bifrost installation

Edit **user.mk** to suit your system, then run:

    $ make -j
    $ sudo make install

which will install the library and headers into /usr/local/lib and
/usr/local/include respectively.

You can call the following for a local Python installation:

    $ sudo make install PYINSTALLFLAGS="--prefix=$HOME/usr/local"

### Docker container

Install dependencies:

 * [Docker Engine](https://docs.docker.com/engine/installation/)
 * [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker)

Build Docker image:

    $ make docker

Launch container:

    $ nvidia-docker run --rm -it ledatelescope/bifrost

For CPU-only builds:

    $ make docker-cpu
    $ docker run --rm -it ledatelescope/bifrost

## Documentation

### [Online Bifrost Documentation](http://ledatelescope.github.io/bifrost/)

### Building the docs with Docker

To quickly build the docs using Docker, ensure that you have
built a Bifrost container as `ledatelescope/bifrost`.
Then, inside the `docs` folder, execute `./docker_build_docs.sh`,
which will create a container called `bifrost_docs`, then
run it, and have it complete the docs-building process for you,
outputting the entire html documentation inside `docs/html` on
your machine.

### Building the docs from scratch

Install sphinx and breathe using pip, and also install Doxygen.

Doxygen documentation can be generated by running:

    $ make doc

This documentation can then be used in a Sphinx build
by running

    $ make html

inside the /docs directory.

## Contributors

 * Ben Barsdell
 * Daniel Price
 * Miles Cranmer
 * Hugh Garsden
 * Jayce Dowell