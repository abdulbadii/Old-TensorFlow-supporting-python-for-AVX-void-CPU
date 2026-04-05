# TensorFlow 1.5 supporting python for AVX-void CPU support
TensorFlow use on AVX instruction void CPU could still be as good as on otherwise   

As TensorFlow version 1.6 up to the latest now verily requires H/W of CPU capable of performing AVX instruction,   
TensorFlow version 1.5 stays working for any CPU incapable of AVX instruction   
Only the point of problem is now; it'd be suited to Python only version 3.6 or so, which is commonly difficult to build due to flaws in discrepancy between   
old and new S/W package leading to build inaccuracies or incorrectness, because it's been in EOL state (on its official page):   

Warning: Python 3.6.0 reached end-of-life on 2021-12-23. It is no longer supported and does not receive security updates. We recommend upgrading to the latest Python release.   

So here and now, this'd try to fix and revive the python 3.6 project and have the python correctly built up to be made use for old TensorFlow

1. Download its project source   
   https://www.python.org/ftp/python/3.6.15/Python-3.6.15.tar.xz   
2. extract it as a directory   
3. make sure the owner is the user and 755 permission, for entire it
4. fix, edit Modules/mathmodule.c
   
6. Do the CL
   ./configure --without-ensurepip --prefix=/usr CFLAGS='-march=native -O3' CPPFLAGS=-I/home/abdu/Downloads/openssl-1.1.0.bin/include LDFLAGS='-L/home/abdu/Downloads/openssl-1.1.0.bin/lib' LIBS='-lssl -lcrypto -ldl -lpthread -lz' && make PROFILE_TASK='-m test.regrtest --pgo -x test_asyncio -x test_buffer -x test_concurrent_futures -x test_lib2to3 -x test_logging -x -x test_pickle -x test_readline -x test_weakref' profile-opt -j3



pythonit iny in some 



curl https://bootstrap.pypa.io/pip/3.6/get-pip.py  | python
