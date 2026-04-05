# Python supporting TensorFlow 1.5 to put AVX instruction-void CPU into work
TensorFlow use on AVX instruction-void CPU could still be as good use as on more modern ones   

As all TensorFlow version 1.6 up to the latest now require very costly H/W, CPU capable of performing AVX instruction,   
TensorFlow version 1.5 stays working for any CPU incapable of AVX instruction   
Only the point of problem is it can only be utilized by use of Python version 3.6 or so only, which is now rather difficult to build due to compatiblility flaws or version discrepancies between old and new S/W package leading to build inaccuracies/incorrectness, and due to it's been in EOL (end-of-life) state (on its official page):   
##### Warning: Python 3.6.0 reached end-of-life on 2021-12-23. It is no longer supported and does not receive security updates. We recommend upgrading to the latest Python release   

So here and now, this page would try to fix and revive the python 3.6 project and have the python correctly built and will easily be made use for old TensorFlow   
It needs as a dependency, certain openSSL version:   

1. Download its project source, go to the python download page   
   `https://www.python.org/ftp/python/3.6.15`   
   browse, find and click the download link or click:   
   `https://www.python.org/ftp/python/3.6.15/Python-3.6.15.tar.xz`   
   Then extract it as a directory   
3. Make sure the owner is the user with 755 permission, entirely   
4. Fix, edit `Modules/mathmodule.c` at the beginning of function `sinpi(double x)` at line 69 or so, insert it with preprocessor code line and its closing one:   

   **#if !defined(__GLIBC__) || __GLIBC__ < 2 || (__GLIBC__ == 2 && __GLIBC_MINOR__ < 38)**
   
   **#endif**   
   So from:
```
    69	static double   
    70	sinpi( double x) {   
    ...
    ...
    103 }   
```
   
become:   

```
    69	#if !defined(__GLIBC__) || __GLIBC__ < 2 || (__GLIBC__ == 2 && __GLIBC_MINOR__ < 38)   
    70	static double   
    71	sinpi( double x)   
    72	{   
    ...
    ...   
    103	}   
    104	#endif   
```   
6. Get openSSl 1.1.0   
   https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_0l/openssl-1.1.0l.tar.gz  
   Extract it as a directory and make sure the owner is the user and has 755 permission, entirely   
7. Create another directory of the same name but slighty renamed and/or suffixed with few letters at the same place of the source extracted,  
   which is meant for its build result, so   
   `...........1.1.0l`   
   become   
   `...........1.1.0.bin`   
   `$ mkdir -p openssl-1.1.0.bin/{include,lib}`   

9. Enter to the source directory and build:   
```
   $ cd openssl-1.1.0l   
   $ ./config no-shared -march=native && make -j
```
11. Copy the process result to the prepared directory
```
    $ cp -r include/openssl ../openssl-1.1.0.bin/include   
    $ cp libcrypto.a libssl.a ../openssl-1.1.0.bin/lib
```
13. Enter to the Python 3.6.15 source directory to try building by a CL which will compile it against the newly made openSSL static library and install it in `/usr/bin`
    It's set to skip some needless PGO build tests   
```    
    ./configure --without-ensurepip --prefix=/usr CFLAGS='-march=native -O3' CPPFLAGS=-I/home/abdu/Downloads/openssl-1.1.0.bin/include LDFLAGS='-L/home/abdu/Downloads/openssl-1.1.0.bin/lib' LIBS='-lssl -lcrypto -ldl -lpthread -lz' && make PROFILE_TASK='-m test.regrtest --pgo -x test_asyncio -x test_buffer -x test_concurrent_futures -x test_lib2to3 -x test_logging -x test_pickle -x test_readline -x test_weakref' profile-opt -j && sudo make install
```

13. **Please note**: As `make` install location is set by `--prefix=/usr` that points actually to `/usr/bin`, this wil replace main/system-wide `python` link
    to link to this python 3.6.15
    So as most installations are linking to the newer one, it must be linked back to it:
```
    $ sudo ln -sf /usr/bin/{python3.14, python}
    $ sudo ln -sf /usr/bin/python{3.14,}-config
```
    and python 3.6.15 related file should be
    $ ls -l /usr/bin/pyt*
 ```
    lrwxrwxrwx 1 root root 9 Apr  5 09:30 /usr/bin/python3 -> python3.6
    -rwxr-xr-x 1 root root 3385 Feb 15 07:49 /usr/bin/python3.14-config
    -rwxr-xr-x 2 root root 3090792 Apr  5 12:23 /usr/bin/python3.6
    lrwxrwxrwx 1 root root 17 Apr  5 09:30 /usr/bin/python3.6-config -> python3.6m-config
    -rwxr-xr-x 2 root root 3090792 Apr  5 12:23 /usr/bin/python3.6m
    -rwxr-xr-x 1 root root 3130 Apr  5 09:30 /usr/bin/python3.6m-config
```
15. Verify it   
    `$ python3.6 --version`    
    `$ python3.6 -c 'import ssl; print(ssl.OPENSSL_VERSION)'`   

16. Have a python 3.6 environment, eg. namedly `TensorFlow1.5` in, say, home directory   
    `$ python3.6 -m venv ~/TensorFlow1.5`   

17. Enter it and Install `pip`   
    `. ~/TensorFlow1.5/activate`   
    `curl https://bootstrap.pypa.io/pip/3.6/get-pip.py  | python`   

18. Ensure TensorFlow dependencies
    ```
    $ pip install absl-py==0.11.0   
    $ pip install protobuf==3.4.0   
    $ pip install wheel==0.3.0   
    $ pip install six==1.10.0   
    $ pip install numpy==1.14.5   
```
19. Obtain TensorFlow 1.5   
    `pip install tensorflow==1.5`   

20. Upon success satisfying every preceding requirement, now one of the most popular AI tool, the Python and TensorFlow tandem may be afforded      

For example, it will do:   
https://github.com/abdulbadii/chessboard-reader-by-old-TensorFlow-and-Python




