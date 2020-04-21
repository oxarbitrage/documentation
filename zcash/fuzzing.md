# zcashd AFL

AFL(American Fuzzy Lop) is a security fuzzer tool. zcashd integrated AFL in a set of easy to use scripts to be run on different platforms and different versions of the software. 


### Build and run fuzzing

```
make clean # if you've previously build zcashd without AFL instrumentation
./zcutil/afl/afl-getbuildrun.sh DecodeHexTx
```

Alternatively you can:

```
./zcutil/afl/afl-get.sh /tmp/afl   # (or wherever you want to build AFL)
./zcutil/afl/afl-build.sh /tmp/afl DecodeHexTx -j$(nproc)
./zcutil/afl/afl-run.sh /tmp/afl DecodeHexTx
```

#### Common issues

You may have some problems at the running stage, specially if you are in a laptop. The help message will tell you what to do, here are a few common issues:

```
[-] PROGRAM ABORT : Pipe at the beginning of 'core_pattern'
```

This can be fixed with:

```
sudo sh -c "/bin/echo core > /proc/sys/kernel/core_pattern"
```

Another one:

```
[-] PROGRAM ABORT : Suboptimal CPU scaling governor

```

Can be fixed by editing files in /sys/devices/system/cpu or just by skipping the test as:

```
AFL_SKIP_CPUFREQ=1 ./zcutil/afl/afl-run.sh '/path/to/your/zcash/afl-temp' DecodeHexTx
```

### References

1. [The fuzzing project](https://fuzzing-project.org)
2. [AFL](https://lcamtuf.coredump.cx/afl/)
3. [Tutorial](https://fuzzing-project.org/tutorial3.html)
4. [Pull request](https://github.com/zcash/zcash/pull/4171)
