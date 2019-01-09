# build maskrcnn-benchmark for win10+vs2017



- ## Prepared environment

- First you need to install pytorch>=1.0 and Microsoft Visual Studio 2017. You can install pytorch by pip or others. For the version of VS, I only test 2017.

- ## Change the CUDA files

- Build maskrcnn-benchmark on windows will rise errors: 

- ```
  /maskrcnn-benchmark/maskrcnn_benchmark/csrc/cuda/ROIAlign_cuda.cu(275):  error: no instance of function template "THCCeilDiv" matches the  argument list argument  types are: (long long, long)
  ```

- Reference form https://github.com/facebookresearch/maskrcnn-benchmark/pull/271, we should change two files below:

- ROIAlign_cuda.cu， ROIPool_cuda.cu 

- ```
  /*  added function */
  int  ceil_div(int a, int b){ 
  	return  (a + b - 1) / b; 
  }
  ...
  
  /* replace the lines with THCCeilDiv, there are 2 palces in each file */
  dim3  grid(std::min(ceil_div((int)output_size, 512), 4096));
  // dim3  grid(std::min(THCCeilDiv(output_size, 512L), 4096L));
  ```

- ## Change setup.py

- Reference form https://github.com/facebookresearch/maskrcnn-benchmark/issues/254#issuecomment-449562202

- ```
  # add 'extra_compile_args' and 'extra_link_args' in get_extensions()
  extra_compile_args  = {"cxx": ['/MD']}
  extra_link_args  = ['/NODEFAULTLIB:LIBCMT.LIB']
  
  # change the ext_modules in get_extensions()
  ext_modules = [
  	extension(
          "maskrcnn_benchmark._C",
          sources,
          include_dirs=include_dirs,
          define_macros=define_macros,
          extra_compile_args=extra_compile_args,
          extra_link_args = extra_link_args,
  		)
  ]
  ```

- ##  Setting up the build environment

- Reference from https://github.com/pytorch/pytorch, Install PyTorch on windows.

- ```
  set  "VS150COMNTOOLS=D:\Program Files (x86)\Microsoft Visual  Studio\2017\Community\VC\Auxiliary\Build"
  set  CMAKE_GENERATOR=Visual Studio 15 2017 Win64
  set  DISTUTILS_USE_SDK=1
  REM  The following two lines are needed for Python 2.7, but the support for it is  very experimental.
  set  MSSdk=1
  set  FORCE_PY27_BUILD=1
  REM  As for CUDA 8, VS2015 Update 3 is also required to build PyTorch. Use the  following line.
  set  "CUDAHOSTCXX=%VS140COMNTOOLS%\..\..\VC\bin\amd64\cl.exe"
  
  call  "%VS150COMNTOOLS%\vcvarsall.bat" x64 -vcvars_ver=14.11
  ```

- ## Build Maskrcnn-benchmark

- ```
  cd maskrcnn-benchmark
  python setup.py install
  # or you can use 
  pip install -e .
  ```