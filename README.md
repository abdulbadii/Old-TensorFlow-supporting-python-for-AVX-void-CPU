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
2. Prepare a location, write down or remember well its path, later this'd be the same path where the Python tar must be extracted to   
   Extract openssl tar in this path as a directory, make sure its owner is the user with 755 permission entirely   
3. So the path will have to gather 3 directories:   
   - `openssl-1.1.0l` source   
   - `openssl-1.1.0l.bin`    (a renamed/suffixed of previous one, automatically created as shown below)   
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

6. Enter into it   
   Edit file `Modules/mathmodule.c`, at the beginning of function `sinpi(double x)` at line 69 or so, insert this preprocessor code line and its closing one:   
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
7. Then try to build using a CL that will compile it against the newly created openSSL static library and install it in `/usr/bin`  
   It's set to skip some needless PGO-build modules tests   
   ```   
   ./configure --without-ensurepip --prefix=/usr CFLAGS='-march=native -O3' CPPFLAGS=-I../openssl-1.1.0l.bin/include LDFLAGS='-L../openssl-1.1.0l.bin/lib' LIBS='-lssl -lcrypto -ldl -lpthread -lz' && make PROFILE_TASK='-m test.regrtest --pgo -x test_asyncio -x test_buffer -x test_concurrent_futures -x test_compileall -x test_decimal -x test_io -x test_lib2to3 -x test_logging -x test_pickle -x test_readline -x test_signal -x test_socket -x test_weakref' profile-opt -j && strip python && sudo make install   
   ```

8. **Please note. IMPORTANT**:   
   Since `make` install location is set by `--prefix=/usr` just shown above, this'd put python binary in `/usr/bin`, simply replacing `python` link
   to point to this python 3.6.15 binary namedly python3.6   
   So since most system tools installations using main/system-wide, newest python, invoking it through this link, it must be linked back to it:   
   ```
   sudo ln -sf /usr/bin/{python3.14, python}`
   sudo ln -sf /usr/bin/python{3.14,}-config
   ```
   Correct it to your version accordingly if needed   
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
11. Enter it and Install `pip` of its version   
    ```
    . ~/TensorFlow1.5/activate`   
    curl -O https://bootstrap.pypa.io/pip/3.6/get-pip.py && python get-pip.py   
    pip --version
    ```
    For user *foo* it'd print:   
    `pip 21.3.1 from /home/foo/tensorFlow1.5/lib/python3.6/site-packages/pip (python 3.6)`   
    
13. Ensure TensorFlow dependencies availability   
    ```
    pip install wheel==0.3.0
    pip install absl-py==0.11.0   
    pip install protobuf==3.4.0   
    pip install six==1.10.0   
    pip install numpy==1.14.5   
    ```
14. Get the TensorFlow itself and then verify  
 ```
   pip install tensorflow==1.5`  
   pip show tensorflow`  
```
Satisfying every preceding requirement should succeed producing Python and TensorFlow that is runnning smoothly on any CPU incapable of AVX instruction   

For example, it will do   
https://github.com/abdulbadii/chessboard-reader-by-old-TensorFlow-and-Python   
