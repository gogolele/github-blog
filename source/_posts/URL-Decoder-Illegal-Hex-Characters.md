title: "URL Decoder Illegal Hex Characters"
date: 2015-03-27 11:28:19
categories: dev
tags: [java,URLDecoder]
---

When we do URL decoding for URL query string, or cookie / session strings, we may encountered the illegal hex problem. Exceptions like this
```
IllegalArgumentException: java.lang.IllegalArgumentException: URLDecoder: Illegal hex characters in escape (%) pattern - For input string: ".P"
at java.net.URLDecoder.decode(Unknown Source)
IllegalArgumentException: java.lang.IllegalArgumentException: URLDecoder: Illegal hex characters in escape (%) pattern - For input string: "zn"
at java.net.URLDecoder.decode(Unknown Source)
```

According to RFC 3986 percent-encoded character has following format: % + hex. So if you want to properly escape url that has unescaped % chars without breaking the whole url before actually decoding it, you need to replace only those % signs that are not followed by hex.

Reference, reserved characters is ok, but unresevered characters are not ok. should replace % --> %25.
http://en.wikipedia.org/wiki/Percent-encoding

The solutions by regex is easy by this code.

```java
class Main
{
  public static void main (String[] args) throws java.lang.Exception
  {
    String url = "http://example.com/test?q=%.P%20some%20other%20Text";
    url = url.replaceAll("%(?![0-9a-fA-F]{2})", "%25");
    System.out.println(url);
  }
}
```

-EOF-
