# zcashd AFL

AFL(American Fuzzy Lop) is a security fuzzer tool. zcashd can be built for AFL wih the help of easy to use scripts. 

### Get and build AFL

```
./zcutil/afl/afl-get.sh -i /tmp/afl
```
Will get the last version of AFL from the internet and will build in the specified directory. 

### Build zcash for fuzzing

If you've previously build zcashd without AFL instrumentation

```
make clean
```

Next build for AFL:

```
./zcutil/afl/afl-build.sh -i /tmp/afl -f DecodeHexTx
```

Option `-f` can be one of the following: CheckBlock, DecodeHexTx, DeserializeAddrMan, DeserializeTx or ReadFeeEstimates

For additional options of the build please execute:

```
./zcutil/afl/afl-build.sh -h
```
### Start fuzzing

```
./zcutil/afl/afl-run.sh -i /tmp/afl -f DecodeHexTx
```

### All in one

There is a ascript that will combine the 3 steps above into 1:

```
./zcutil/afl/afl-run.sh -f DecodeHexTx
```

This will install AFL in `./afl-temp`


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
