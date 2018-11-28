# Product:
"KNC is Kerberised NetCat. It works in basically the same way as either netcat or stunnel except that it is uses GSS-API to secure the communication. You can use it to construct client/server applications while keeping the Kerberos libraries out of your programs address space quickly and easily."

## Links
Official page:
http://oskt.secure-endpoints.com/knc.html
Source code repository:
https://github.com/elric1/knc/

## CVE-2017-9732 vulnerability:
knc (Kerberised NetCat) before 1.11-1 is vulnerable to denial of service (memory exhaustion) that can be exploited remotely without authentication, possibly affecting another services running on the targeted host.

The knc implementation uses a temporary buffer in read_packet() function that is not freed (memory leak). An unauthenticated attacker can abuse this by sending a blob of valid kerberos handshake structure but with unexpected type; instead of token type AP_REQ (0x0100) I sent 0x0000 at bytes 16 and 17 in my proof of concept. During the attack, gss_sec_accept_context returns G_CONTINUE_NEEDED and the memory is exhausted in the long run.

The attack might not even be logged, depending on how the Open Source Kerberos Tooling stack is configured.

## Exploit

```
./memory-exhaustion-poc.pl target-host target-port
```

Top output ~30 seconds later:

```
2 CPUs;  last pid: 20507;  load averages:  1.26,  0.84,  0.38                                                                                                   14:11:53

268 processes: 4 running, 264 sleeping

CPU states: 38.6% user,  0.0% nice, 27.1% system, 34.3% idle

     cpu 0: 18.7% user,  0.0% nice, 22.9% system, 58.4% idle

     cpu 1: 54.7% user,  0.0% nice, 30.7% system, 14.6% idle

Memory: 7857M real, 7734M used, 122M free  Swap: 4094M total, 0K used, 4094M free

 

  PID  PSID USERNAME   THR PRI NICE  SIZE   RES STATE   TIME     CPU COMMAND

20156 19806 username     1  20    0 2165M 2133M cpu1   19:41  46.19% knc
...
```



## Bugfix:
https://github.com/elric1/knc/commit/f237f3e09ecbaf59c897f5046538a7b1a3fa40c1
