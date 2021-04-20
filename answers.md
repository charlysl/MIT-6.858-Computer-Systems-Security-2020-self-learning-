# Answers to [Lab1: Buffer Overflows](https://css.csail.mit.edu/6.858/2020/labs/lab1.html)

## Part 1: Finding buffer overflows

### Exercise 1: description of a vulnerability

#### Buffer which may overflow

```
char value[512];
```

In function ```http_request_headers```, line 116 of ```http.c```.

This is a stack buffer that can be overflown inside said function at line 159:

```
url_decode(value, sp);
```

```sp``` is where the server will have parsed a request header's value, url encoded.

#### Structure of the input to the web server (HTTP request) to overflow the buffer and overwrite the return address

```
GET / HTTP/1
Pwned: <576 As>
```
The buffer's address is ```0x7fffffffda90```.
The return address is at ```0x7fffffffdcc8```.
```0x7fffffffdcc8 - 0x7fffffffda90 + 8 = 576```

#### Call stack that will trigger the buffer overflow

```
process_client
http_request_headers
```

## Part 3:
### Challenge

The attack builds on ```exploit-4.py```, and consists of creating the following structure in the stack:

```
| str     |
| &unlink |   
| &str    |
| &gadget | <- overwritten return address of the call to http_request_headers
```

So, by the time ```http_request_headers``` returns, the gadget will be executed, and will pop the string's address (which points to the path of the file that we want to unlink) into %rdi, and, when returning, it will execute unlink.

The **gadget** is ```pop %rdi; ret```, and has machine code ```5f c3```.

It can be found in the web server's binary in two different ways:

1) manually, by looking through the output of ```objdump -D zookd-nxstack | less```

2) automatically, by executing ```ROPgadget --binary zookd-nxstack | grep rdi | grep ret```

However, the gadget's address as obtained above is incorrect; this can be fixed by looking at the address of the function that contains it, ```__libc_csu_init```, when running in gdb, and adjust it accordingly.
