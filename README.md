# Channel ID Private Key Protection

## About

There are two related patches in this project: The BoringSSL patch enables an
opaque key for Channel ID, and the Chromium patch enables an opaque key along
with providing an implementation of hardware-backed keys using the Intel SGX
SDK. Together, these patches demonstrate how Chromium could store its Channel ID
private keys in an Intel SGX enclave.

## Compile and run

Follow the following steps to compile and run:

1. Be on a machine with an SGX processor.
2. Enter the BIOS settings, and make sure SGX is set to `enabled`.
3. Install Linux.
4. Install the SGX Alpha SDK, following the included instructions.
5. Download the chromium repository following the instructions available. We
   will call its path `<chromium>`.
6. Checkout commit `f594a1085b49369c2479e5526dd0ef8b116e9af0` in the chromium
   repository.
7. `cd third_party/boringssl/src`, checkout commit
   `afe57cb14d36f70ad4a109fc5e7765d1adc67035`.
8. Chrome uses an included root to handle packages.
```
cd build/linux/debian_wheezy_<amd64/i386>-sysroot/
mkdir opt && cd opt && mkdir intel && cd intel
cp -r /opt/intel/sgxsdk .
```
9. Set up pkgconfig by creating the file `/usr/lib/pkgconfig/sgx.pc` with contents:

```
prefix=/opt/intel/sgxsdk
exec_prefix=${prefix}
includedir=${prefix}/include
libdir=${prefix}/lib64

Name: sgx
Description: Intel SGX SDK for Linux
Version: 1.0.0
Cflags: -I${includedir}
Libs: -L${libdir} -lsgx_urts -lsgx_uae_service -lsgx_tservice
```
10. add to `~/.bashrc`:
```
export LD_LIBRARY_PATH=/lib64:/usr/lib:/opt/intel/sgxsdk/lib64
export CHROME_DEVEL_SANDBOX="/usr/local/sbin/chrome-devel-sandbox"
```
(this second one should already be there if chrome compiles properly)
11. Apply `chromium/chromium.patch` to `<chromium>/src`.
12. Apply `boringssl/bssl.patch` to `third_party/boringssl/src`.
13. If you're using gn, from `<chromium>/src`:
```
gn args out/Default
add the line:
enable_nacl = false
```
14. Compile the enclave
```
cd net/ssl
make
cd ../../
```
15. Compile chrome
(first time: `ninja -C out/Default chrome chrome_sandbox`)
```
ninja -C out/Default chrome
```
16. Run chrome
```
out/Default/chrome
```
17. Load google.com

If you want to run tests:
```
ninja -C out/Default net_unittests
out/Default/net_unittests --gtest_filter=*ChannelID*
```

There are more tests within boringssl, with instructions in
`third_party/boringssl/src/BUILDING.md`
