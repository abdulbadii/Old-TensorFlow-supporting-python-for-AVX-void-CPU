# Python supporting TensorFlow 1.5 to get AVX instruction-void CPU into work
TensorFlow uses on AVX instruction-void CPU may be as good as that on more modern CPU   

As all TensorFlow version newer than 1.5 require very pricey CPU capable of performing AVX instruction, TensorFlow version 1.5 can still work on any CPU incapable of AVX instruction   

Now the point of problem is; this TensorFlow can only be utilized by using Python version 3.6 or older only, which is now difficult to build due to incompatibilility or incorrectness interfacing between it and current OS mainstream packages, leading to build failures.   
This because of its due EOL (end-of-life) as stated on its official page:   
##### Warning: Python 3.6.0 reached end-of-life on 2021-12-23. It is no longer supported and does not receive security updates. We recommend upgrading to the latest Python release   

So here and now, this is to fix and revive the Python 3.6 project to have this Python correctly built and be made use for old TensorFlow version 1.5   
It requires, as a dependency, certain openSSL version   

1. Get openSSl 1.1.0   
   https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_0l/openssl-1.1.0l.tar.gz  
   Extract it as a directory and make sure the owner is the user with 755 permission, entirely   

2. To prepare the build result, create another directory at the same place of the source extracted (being sibling each other), almost with the same name only slighty renamed, e.g. suffixed with few letters   
   `...........1.1.0l`   
   become   
   `...........1.1.0.bin`  
   Within it prepare these 2 directories:   
   ```
   mkdir -p openssl-1.1.0.bin/{include,lib}
   ```
4. Enter to the source directory and build   
   ```
   cd openssl-1.1.0l && ./config no-shared -march=native && make -j
   ```
6. Copy the process result to the prepared directory
   ```
   cp -r include/openssl ../openssl-1.1.0.bin/include   
   cp libcrypto.a libssl.a ../openssl-1.1.0.bin/lib
   ```
5. Now Python 3.6.15; download its project source, go to the python download page   
   https://www.python.org/ftp/python/3.6.15   
   Find and click the download link, or click:   
   https://www.python.org/ftp/python/3.6.15/Python-3.6.15.tar.xz   
   Then extract it as a directory. Make sure the owner is the user with 755 permission, entirely   
6. Edit `Modules/mathmodule.c` at the beginning of function `sinpi(double x)` at line 69 or so, insert it with preprocessor code line and its closing one:   
   ```
   #if !defined(__GLIBC__) || __GLIBC__ < 2 || (__GLIBC__ == 2 && __GLIBC_MINOR__ < 38)
   
   #endif
   ```   
   So this:
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
   103 }   
   104 #endif   
   ```
7. Enter to the Python 3.6.15 source directory to try building by a CL that compiles against the newly made openSSL static library and install it in `/usr/bin`  
    It's set to skip some needless PGO build tests   
   ```    
    ./configure --without-ensurepip --prefix=/usr CFLAGS='-march=native -O3' CPPFLAGS=-I/home/abdu/Downloads/openssl-1.1.0.bin/include LDFLAGS='-L/home/abdu/Downloads/openssl-1.1.0.bin/lib' LIBS='-lssl -lcrypto -ldl -lpthread -lz' && make PROFILE_TASK='-m test.regrtest --pgo -x test_asyncio -x test_buffer -x test_concurrent_futures -x test_lib2to3 -x test_logging -x test_pickle -x test_readline -x test_weakref' profile-opt -j && sudo make install
   ```

8. **Please note**:   
   As `make` install location is set by `--prefix=/usr` that points actually to `/usr/bin`, this wil replace main/system-wide `python` link
   to link to this python 3.6.15
   So as most installations are linking to the newer one, it must be linked back to it:   
   ```
   sudo ln -sf /usr/bin/{python3.14, python}`
   sudo ln -sf /usr/bin/python{3.14,}-config
   ```   
   We can simply inspect python 3.6.15 related files   
   ```
   ls -l /usr/bin/pyt*
   ```
   ```
   lrwxrwxrwx 1 root root 19 Apr  5 09:33 /usr/bin/python -> /usr/bin/python3.14
   lrwxrwxrwx 1 root root 9 Apr  5 09:30 /usr/bin/python3 -> python3.6
   -rwxr-xr-x 1 root root 14384 Feb 15 07:49 /usr/bin/python3.14
   -rwxr-xr-x 1 root root 3385 Feb 15 07:49 /usr/bin/python3.14-config
   -rwxr-xr-x 2 root root 3090792 Apr  5 12:23 /usr/bin/python3.6
   lrwxrwxrwx 1 root root 17 Apr  5 09:30 /usr/bin/python3.6-config -> python3.6m-config
   ```
9. Verify   
   ```
   python3.6 --version`    
   python3.6 -c 'import ssl; print(ssl.OPENSSL_VERSION)'
   ```   

10. Next is to have a Python 3.6 environment, say, it's `TensorFlow1.5` in home directory:   
    ```
    python3.6 -m venv ~/TensorFlow1.5
    ```   
11. Enter it and Install `pip`   
    ```
    . ~/TensorFlow1.5/activate`   
    curl https://bootstrap.pypa.io/pip/3.6/get-pip.py  | python
    ```
12. Ensure TensorFlow dependencies availability   
    ```
    pip install wheel==0.3.0
    pip install absl-py==0.11.0   
    pip install protobuf==3.4.0   
    pip install six==1.10.0   
    pip install numpy==1.14.5   
    ```
13. Get the TensorFlow itself and then verify  
 ```
   pip install tensorflow==1.5`  
   pip show tensorflow`  
```
Satisfying every preceding requirement should succeed producing Python and TensorFlow that is runnning smoothly on any CPU incapable of AVX instruction   

For example, it will do   
https://github.com/abdulbadii/chessboard-reader-by-old-TensorFlow-and-Python   
