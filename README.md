# Python supporting TensorFlow 1.5 to get AVX instruction-void CPU into work
TensorFlow uses on AVX instruction-void CPU may be as good as that on more modern CPU   

While a TensorFlow version newer than 1.5 requires very pricey CPU capable of performing AVX instruction, the TensorFlow version 1.5 can still work on a CPU incapable of performing AVX instruction   

Now the point is; this TensorFlow can only be utilized by using Python version 3.6 or older only, which is now difficult to build due to incompatibilility or interfacing incorrectness between it and current OS mainstream packages, leading to build failures.   
This because of its due EOL (end-of-life) as stated on its official page:   
##### Warning: Python 3.6.0 reached end-of-life on 2021-12-23. It is no longer supported and does not receive security updates. We recommend upgrading to the latest Python release   

So here and now, this is to fix and revive the Python 3.6 project to have this Python correctly built and be made use for old TensorFlow version 1.5   
It requires, as a dependency, certain openSSL version   

1. Get openSSl 1.1.0   
   https://github.com/openssl/openssl/releases/download/OpenSSL_1_1_0l/openssl-1.1.0l.tar.gz  
2. Prepare a location, write down or remember well its path, later this'd be the same path where the Python tar must be extracted to   
   Extract openssl tar in this path as a directory, make sure its owner is the user with 755 permission entirely   
3. So the path would gather directory:   
   - `openssl-1.1.0l` source   
   - `openssl-1.1.0l.bin`    (a renamed/suffixed of previous one automatically created, shown below)   
   - `Python-3.6.15` source   
4. Enter into openssl-1.1.0l source directory and build   
   ```
   cd openssl-1.1.0l && ./config -march=native --prefix=$PWD.bin && make -j && make install   
   ```
   Notice `$PWD.bin`. This suffixes it as `openssl-1.1.0l.bin` which is then automatically created during `make install` execution   

5. Now download Python 3.6.15 project source from its download page   
   https://www.python.org/ftp/python/3.6.15   
   Find and click the download link, or click:   
   https://www.python.org/ftp/python/3.6.15/Python-3.6.15.tar.xz   
   Extract it as directory in the path just explained above, ensure the owner is the user with 755 permission entirely   

6. Enter it and edit file `Modules/mathmodule.c`
   Go down to the beginning of function `sinpi(double x)` at line 69 or so, and insert this preprocessor code line and its closing one:   
   ```
   #if !defined(__GLIBC__) || __GLIBC__ < 2 || (__GLIBC__ == 2 && __GLIBC_MINOR__ < 38)
   
   #endif
   ```   
   So this:
   ```
   69	static double   
   70	sinpi( double x)\
   {   
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
   103 }   
   104 #endif   
   ```
8. Build it using CL that would compile it against the newly created openSSL static library and has option to skip useless PGO-build module testsinstall
   it will install it in `/usr/bin`   
   ```   
   ./configure --without-ensurepip --prefix=/usr CFLAGS='-march=native -O3' CPPFLAGS=-I../openssl-1.1.0l.bin/include LDFLAGS='-L../openssl-1.1.0l.bin/lib' LIBS='-lssl -lcrypto -ldl -lpthread -lz' && make PROFILE_TASK='-m test.regrtest --pgo -x test_asyncio test_buffer test_concurrent_futures test_compileall test_decimal test_httplib test_io test_lib2to3 test_pickle test_readline test_signal test_socket test_tarfile test_trace test_venv test_weakref' profile-opt -j && strip python && sudo make install   
   ```   
9. Verify the one there  
   ```
   ./python --version`  && ./python -c 'import ssl; print(ssl.OPENSSL_VERSION)'
   ```
   `Python 3.6.15`   
   `OpenSSL 1.1.0l  10 Sep 2019`   

10. **Please note. IMPORTANT!**   
    Simply inspect the python related files   
    ```
    ls -l /usr/bin/pyt*
    ```
    `lrwxrwxrwx 1 root root 19 Apr  5 09:33 /usr/bin/python -> /usr/bin/python3.14`   
    `lrwxrwxrwx 1 root root 9 Apr  5 09:30 /usr/bin/python3 -> python3.6`   
    `-rwxr-xr-x 1 root root 14384 Feb 15 07:49 /usr/bin/python3.14`   
    `-rwxr-xr-x 1 root root 3385 Feb 15 07:49 /usr/bin/python3.14-config`   
    `-rwxr-xr-x 2 root root 3090792 Apr  5 12:23 /usr/bin/python3.6`   
    `lrwxrwxrwx 1 root root 17 Apr  5 09:30 /usr/bin/python3.6-config -> python3.6m-config`   

    Since `make` install location set by `--prefix=/usr` shown in CL above will put python3.6.15 binary in `/usr/bin`, the installation may possibly replace system-wide `python` link with another pointing to it. Then, as most system tools using this main/system-wide python, invoking it will mistakenly run one that just installed. On that case, must link it back   
    ```
    sudo ln -sf /usr/bin/{python3.14, python}`
    sudo ln -sf /usr/bin/python{3.14,}-config
    ```
    Correct the actual version number accordingly   

11. Next is to have a Python 3.6 environment, say, it's `TensorFlow1.5` in home directory:   
    ```
    python3.6 -m venv ~/TensorFlow1.5
    ```   
12. Enter it and Install `pip` of its version and verify   
    ```
    . ~/TensorFlow1.5/activate`   
    curl -O https://bootstrap.pypa.io/pip/3.6/get-pip.py && python get-pip.py   
    pip --version && rm get-pip.py
    ```
    Upon success if user is *foo* it'll print:   
    `pip 21.3.1 from /home/foo/TensorFlow1.5/lib/python3.6/site-packages/pip (python 3.6)`   
    
13. Ensure TensorFlow dependencies availability   
    ```
    pip install wheel==0.3.0
    pip install absl-py==0.11.0   
    pip install six==1.10.0   
    pip install numpy==1.14.5   
    pip install protobuf==3.4.0   
    pip install keras==2.1.5
    ```
14. Get the TensorFlow itself and then verify  
 ```
   pip install tensorflow==1.5`  
   pip show tensorflow`  
```
Satisfying every preceding requirement should succeed producing Python and TensorFlow running smoothly on a CPU void of AVX instruction   

For example it will do   
https://github.com/abdulbadii/chessboard-reader-by-old-TensorFlow-and-Python   
