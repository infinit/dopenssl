# dOpenSSL

![C/C++ CI](https://github.com/bernardoaraujor/dopenssl/workflows/C/C++%20CI/badge.svg)

- [Intro](#intro)
- [Dependencies](#dependencies)
- [Clone, Build, Install](#clone--build--install)
- [Example](#example)
- [Maintainers](#maintainers)

## Intro

**Disclaimer:** This is the continuation of the [original implementation](https://github.com/infinit/dopenssl) by [Infinit](https://infinit.sh/). [Drake](https://github.com/infinit/drake) has been replaced by [CMake](https://cmake.org/) as default build system.

The dOpenSSL library extends the OpenSSL Project cryptographic library so as to provide deterministic random generation functionalities. Basically, dOpenSSL guarantees that should a big number or a cryptographic key be generated, given a PRNG's state, the result would be always the same.

The OpenSSL random generator introduces entropy in many places, making it unusable in a deterministic way. Thus, some functions have been cloned and adapted in order to guarantee determinism.

This product includes software developed by the OpenSSL Project for use in the OpenSSL Toolkit.

## Dependencies
The library relies upon the following libraries:

 * [**OpenSSL**](http://www.openssl.org) which provides the fundamental cryptographic operations. Install v1.0.2 into Ubuntu 18.04 with:

```
$ sudo apt-get install libssl1.0-dev -y
```

## Clone, Build, Install
Assuming you're on Ubuntu 18.04:

```
$ git clone http://github.com/bernardoaraujor/dopenssl-ng
$ cd dopenssl-ng
$ mkdir build; cd build
$ cmake ..; make
$ sudo make install
```

## Example
The following source code can be found in `src/sample.c`.

It shows how dOpenSSL can be used to generate cryptographic keys (in this case 2048-bit RSA keys) in a deterministic way i.e based on a given passphrase (seed).

The program therefore assures that, given the same passphrase (seed), the exact same RSA key will be generated.

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <dopenssl/rsa.h>
#include <dopenssl/rand.h>

int main(int argc, char **argv)
{
  if (argc != 2)
  {
    printf("usage: %s [passphrase]\n", argv[0]);
    return(1);
  }

  unsigned int const bits = 2048;

  char const* passphrase = argv[1];
  unsigned int const length = strlen(passphrase);
  unsigned int const size = bits / 8;
  unsigned int const occurences = size / length + 1;

  unsigned char* seed = malloc(occurences * length + 1);
  memset(seed, 0x0, occurences * length + 1);

  /* Concatenate the passphrase so as to create a 256-byte buffer */
  unsigned int i;
  for (i = 0; i < occurences; i++)
    strncpy(seed + (i * length), passphrase, length);

  /* Initialize the deterministic random generator */
  if (dRAND_init() != 1)
  {
    printf("[error] unable to initialize the random generator\n");
    return (1);
  }

  /* Generate the RSA key */
  RSA* rsa = NULL;

  if ((rsa = dRSA_deduce_privatekey(bits, seed, size)) == NULL)
  {
    printf("[error] unable to generate the RSA key pair\n");
    return (1);
  }

  /* Display the RSA components */
  RSA_print_fp(stdout, rsa, 0);

  /* Clean the resources */
  if (dRAND_clean() != 1)
  {
    printf("[error] unable to clean the random generator\n");
    return (1);
  }

  return (0);
}
```

After installing `libdopenssl` on your system, do the command below to build this example:
```
$ gcc src/sample.c -o sample -ldopenssl -lcrypto
```

Finally, just launch the sample program by providing a passphrase:

```Shell
$ ./sample "chiche donne nous tout"
Private-Key: (2048 bit)
modulus:
    00:c2:06:28:58:c4:ee:e8:d0:65:e5:2f:fd:d4:1c:
    86:73:fb:77:ab:cc:ee:50:3e:20:b1:d2:7e:3c:bf:
    20:35:2f:b7:1e:ab:4a:03:d0:9f:c7:8f:46:44:8f:
    03:a4:3c:da:e2:19:f0:fe:41:e2:7b:41:75:76:3a:
    b0:d0:de:eb:26:dd:4c:c7:50:d5:33:61:bf:71:8a:
    6e:b2:bb:bf:1b:2e:14:6b:a3:1c:c7:0d:5a:31:67:
    2c:9f:e9:17:56:31:8c:9f:86:93:f7:b3:2e:2c:c0:
    59:24:43:5c:ed:bd:0b:bc:4d:65:33:c7:89:4e:62:
    f7:9f:cd:da:5c:f9:e4:c9:b2:9c:43:6f:78:06:32:
    e6:f5:07:1e:c0:fa:1e:6d:4b:c8:68:a8:93:a7:53:
    e7:11:92:e0:e0:43:6d:11:36:a6:37:18:9b:33:c3:
    8a:75:cf:3a:10:18:67:3c:13:07:6c:a6:6b:f2:6b:
    36:aa:fd:9c:29:5a:38:95:0d:e0:a2:81:07:41:9a:
    17:62:a4:9e:fc:a7:32:1a:3d:79:83:75:fa:73:e8:
    47:e2:ac:08:2e:65:cc:12:a1:ac:59:b1:e8:e1:4d:
    6c:0e:8e:01:02:53:f1:52:04:a3:3f:03:c4:02:0a:
    9f:35:3f:a2:b9:4d:66:31:63:5a:77:3d:28:ab:bb:
    24:6f
publicExponent:
    01:b4:65:67:bf:94:56:af:f4:12:61:6b:73:7e:e8:
    89:5e:95:39:44:ac:7c:85:5c:fe:65:4f:23:1e:b8:
    96:50:3f:bc:27:89:ae:8a:84:4a:d9:07:8a:67:5f:
    20:7f:85:90:45:32:db:0f:28:0b:00:2c:3f:16:d1:
    4c:69:82:df:d5:05:35:8c:ed:2e:69:b9:cc:c2:69:
    50:5b:30:96:b9:51:b5:8d:4e:25:9a:ef:d7:fa:ae:
    65:f2:3b:41:32:19:e3:04:81:1d:a5:ec:d8:d1:04:
    ad:e5:af:42:4a:1d:91:88:9c:6e:09:32:43:3a:e9:
    d3:83:60:64:b2:54:4c:98:1b
[...]
```

Note that you should get exactly the same output since dOpenSSL guarantees the result is deterministic.

## References
- [*The Mathematics of the RSA Public-Key Cryptosystem*](http://www.mathaware.org/mam/06/Kaliski.pdf). Burt Kaliski. **RSA Laboratories**.
- [*How does RSA work?*](https://hackernoon.com/how-does-rsa-work-f44918df914b). Jeronimo (Jerry) Garcia. **Hackernoon**.

## Maintainers
 * Website: https://about.me/bernardo_ar
 * Email: bernardoaraujor@gmail.com
