# https

Downloads by URL for [voc](https://github.com/vishaps/voc): `http://` with
[http](https://github.com/norayr/http), `https://` with TLS 1.3 written in Oberon
([tls](https://github.com/norayr/tls), no C library for TLS). The same code is used by polpo.

```
make            # fetches and builds strutils, base64, Internet, http and tls, then https
make tests      # local TLS test servers (python3 with cryptography): good, refused, cut, 1 MB
make tests NET=net   # also https://example.com/ and http://example.com/
build/fetch https://example.com/ page.html
```

```oberon
IMPORT https, http;
VAR h: http.Client;
...
IF https.Get("https://example.com/", h) & h.rspnComplete THEN
  (* h.rspnFirstLine^, h.rspnBody, h.rspnContentLength *)
  IF https.Save(h, "page.html") THEN ... END
END
```

The server certificate chain is checked up to a root of the CA bundle (`SSL_CERT_FILE`, else
`/etc/ssl/certs/ca-certificates.crt`), with the host name, CertificateVerify and Finished; the
connection is refused otherwise. A body is complete when it has its Content-Length or its last
chunk, and a TLS connection that ended without close_notify is not complete.
