# Fuzzing of python's interpreter - Cpython 3.10.13 with AFL++
Source code here: https://github.com/python/cpython

Corups data for fuzzing test will be are simple python scripts. For example, one is:
```
# Initialize the list
weekdays = ["Sunday", "Monday", "Tuesday",
            "Wednesday", "Thursday", "Friday", "Saturday"]
print("Seven Weekdays are:\n")
# Iterate the list using for loop
for day in range(len(weekdays)):
    print(weekdays[day])
```
In the directory of compiled cpython, use this command, to execute any python script:
```
./python <filename.py>
```

For fuzzing we need to build cpython twice. 
The first build will be for fuzzing (address and undefined sanitizers + afl-clang-lto compiler).
The second build is just for coverage report.

Prepeare before builds:
```
git clone https://github.com/python/cpython --branch v3.10.13 cpython3
cd cpython3 
mkdir build_afl && mkdir build_coverage
```
Copy all corpus data in build_afl directiry:
```
cp -r corpus cpython3/build_afl
```

I will be use a ready AFL++ docker-container. Mount cpython3 directory in docker with -v option:
```
docker run --rm -ti -v /path/to/cpython3:/cpython3 aflplusplus/aflplusplus
```
Build cpython3 with afl-clang-lto compler, set ax_cv_c_float_words_bigendian=no, ASAN_OPTIONS="detect_leaks=0", AFL_USE_ASAN=1, AFL_USE_UBSAN=1
```
cd /cpython3/build_afl
../configure ax_cv_c_float_words_bigendian=no CC="afl-clang-lto"
AFL_USE_ASAN=1 AFL_USE_UBSAN=1 ASAN_OPTIONS="detect_leaks=0" make -j`nproc`
```
After build cpython3, preload all python libs and run fuzzing:
```
AFL_PRELOAD=\
../build/lib.linux-x86_64-3.10/_asyncio.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_bisect.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_blake2.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_cn.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_hk.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_iso2022.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_jp.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_kr.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_codecs_tw.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_contextvars.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_crypt.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_csv.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_ctypes.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_ctypes_test.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_curses.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_curses_panel.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_datetime.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_decimal.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_elementtree.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_heapq.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_json.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_lsprof.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_md5.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_multibytecodec.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_multiprocessing.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_opcode.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_pickle.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_posixshmem.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_posixsubprocess.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_queue.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_random.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_sha1.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_sha256.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_sha3.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_sha512.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_socket.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_statistics.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_struct.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testbuffer.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testcapi.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testclinic.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testimportmultiple.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testinternalcapi.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_testmultiphase.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_uuid.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_xxsubinterpreters.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_xxtestfuzz.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/_zoneinfo.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/array.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/audioop.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/binascii.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/cmath.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/fcntl.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/grp.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/math.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/mmap.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/nis.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/ossaudiodev.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/pyexpat.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/resource.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/select.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/spwd.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/syslog.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/termios.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/unicodedata.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/xxlimited.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/xxlimited_35.cpython-310-x86_64-linux-gnu.so:\
../build/lib.linux-x86_64-3.10/zlib.cpython-310-x86_64-linux-gnu.so \
afl-fuzz -i in -o out -- ./python @@
```
In another window, let's run collect coverage in live mode.
Firstly, build cpython with coverage flags:
```
cd /cpython3/build_coverage
../configure LDFLAGS="-ftest-coverage -fprofile-arcs" CFLAGS="-ftest-coverage -fprofile-arcs"
make -j`nproc`
```
And run afl-cov to collect coverage:
```
afl-cov --live -d ../build_afl/out -e "./python AFL_FILE" -c .
```
