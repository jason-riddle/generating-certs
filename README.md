#### Create the Root CA (1-generate-root-ca)

```bash
$ curl -O https://storage.googleapis.com/configs.kuar.io/ca-csr.json
```

```bash
$ cat ca-csr.json
{
  "CN": "Kubernetes CA",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
```

```bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca

$ ls -al
total 40
drwxr-xr-x  7 puu562  COF\Domain Users   238 Feb 10 12:02 .
drwxr-xr-x  5 puu562  COF\Domain Users   170 Feb 10 12:04 ..
-rw-r--r--  1 puu562  COF\Domain Users   214 Feb 10 12:01 ca-csr.json
-rw-------  1 puu562  COF\Domain Users  1679 Feb 10 12:02 ca-key.pem
-rw-r--r--  1 puu562  COF\Domain Users  1009 Feb 10 12:02 ca.csr
-rw-r--r--  1 puu562  COF\Domain Users  1359 Feb 10 12:02 ca.pem

$ openssl rsa -noout -text -in ca-key.pem
# Private-Key: (2048 bit)
# <Snip>

$ openssl req -noout -text -in ca.csr
# Certificate Request:
# <Snip>

$ openssl x509 -noout -text -in ca.pem
# Certificate:
# <Snip>

$ openssl x509 -noout -subject -issuer -in ca.pem
# subject= /C=US/O=Kubernetes/OU=CA/L=Portland/ST=Oregon/CN=Kubernetes CA
# issuer= /C=US/O=Kubernetes/OU=CA/L=Portland/ST=Oregon/CN=Kubernetes CA
```

```bash
$ curl -O https://storage.googleapis.com/configs.kuar.io/ca-config.json
```

```bash
$ cat ca-config.json
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "server": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ],
        "expiry": "8760h"
      },
      "client": {
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
```

#### Create the Server Certificate (2-generate-server-cert)

```bash
$ curl -O https://storage.googleapis.com/configs.kuar.io/apiserver-csr.json
```

```bash
$ cat apiserver-csr.json
{
  "CN": "*.c.PROJECT_ID.internal",
  "hosts": [
    "127.0.0.1",
    "EXTERNAL_IP",
    "*.c.PROJECT_ID.internal"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "API Server",
      "ST": "Oregon"
    }
  ]
}
```

```bash
$ cfssl gencert \
    -ca=../1-generate-ca/ca.pem \
    -ca-key=../1-generate-ca/ca-key.pem \
    -config=ca-config.json \
    -profile=server \
    apiserver-csr.json | cfssljson -bare apiserver
```

```bash
$ ls -al
total 40
drwxr-xr-x  7 puu562  COF\Domain Users   238 Feb 10 13:49 .
drwxr-xr-x  6 puu562  COF\Domain Users   204 Feb 10 13:42 ..
-rw-r--r--  1 puu562  COF\Domain Users   316 Feb 10 13:42 apiserver-csr.json
-rw-------  1 puu562  COF\Domain Users  1679 Feb 10 13:49 apiserver-key.pem
-rw-r--r--  1 puu562  COF\Domain Users  1131 Feb 10 13:49 apiserver.csr
-rw-r--r--  1 puu562  COF\Domain Users  1480 Feb 10 13:49 apiserver.pem
```

```bash
$ openssl rsa -noout -text -in apiserver-key.pem
# Private-Key: (2048 bit)
# <Snip>

$ openssl req -noout -text -in apiserver.csr
# Certificate Request:
# <Snip>

$ openssl x509 -noout -text -in apiserver.pem
# Certificate:
# <Snip>

$ openssl x509 -noout -subject -issuer -in apiserver.pem
# subject= /C=US/O=Kubernetes/OU=API Server/L=Portland/ST=Oregon/CN=*.c.PROJECT_ID.internal
# issuer= /C=US/O=Kubernetes/OU=CA/L=Portland/ST=Oregon/CN=Kubernetes CA
```

#### Generate the Client Certificate (3-generate-client-cert)

```bash
$ curl -O https://storage.googleapis.com/configs.kuar.io/admin-csr.json
```

```bash
$ cat admin-csr.json
{
  "CN": "admin",
  "hosts": [
    ""
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Cluster Admins",
      "ST": "Oregon"
    }
  ]
}
```

```bash
cfssl gencert \
    -ca=../1-generate-ca/ca.pem \
    -ca-key=../1-generate-ca/ca-key.pem  \
    -config=../1-generate-ca/ca-config.json \
    -profile=client \
    admin-csr.json | cfssljson -bare admin
```

```bash
$ ls -al
total 32
drwxr-xr-x  6 puu562  COF\Domain Users   204 Feb 10 14:00 .
drwxr-xr-x  7 puu562  COF\Domain Users   238 Feb 10 13:58 ..
-rw-r--r--  1 puu562  COF\Domain Users   235 Feb 10 13:58 admin-csr.json
-rw-------  1 puu562  COF\Domain Users  1679 Feb 10 14:00 admin-key.pem
-rw-r--r--  1 puu562  COF\Domain Users  1054 Feb 10 14:00 admin.csr
-rw-r--r--  1 puu562  COF\Domain Users  1403 Feb 10 14:00 admin.pem
```

```bash
$ openssl rsa -noout -text -in admin-key.pem
# Private-Key: (2048 bit)
# <Snip>

$ openssl req -noout -text -in admin.csr
# Certificate Request:
# <Snip>

$ openssl x509 -noout -text -in admin.pem
# Certificate:
# <Snip>

$ openssl x509 -noout -subject -issuer -in admin.pem
# subject= /C=US/O=Kubernetes/OU=Cluster Admins/L=Portland/ST=Oregon/CN=admin
# issuer= /C=US/O=Kubernetes/OU=CA/L=Portland/ST=Oregon/CN=Kubernetes CA
```
