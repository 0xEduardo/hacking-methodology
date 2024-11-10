# Fuzzing

> **Rate Limited?**
> 
> Try to run techniques from [403 Bypass](https://caon.io/docs/exploitation/403bypass/)

> **Configure your options!** You can customize your ffuf with special information, here’s a sample:
> 
> [https://gist.github.com/felipecaon/d1e7c980d7bab1312ea81df1d0241f42](https://gist.github.com/felipecaon/d1e7c980d7bab1312ea81df1d0241f42)

`.ffufrc`
```
[general]
  colors = true

[http]
    headers = [
        "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:98.0) Gecko/20100101 Firefox/98.0"
    ]

[https]
    headers = [
        "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:98.0) Gecko/20100101 Firefox/98.0"
    ]
    
# run as ffuf -u https://example.com/FUZZ
[input]
    wordlists = [
        "/usr/share/seclists/Discovery/Web-Content/common.txt:FUZZ",
        "/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:PARAM"
    ]
```

```
# https://github.com/ffuf/ffuf
ffuf -w /path/to/wordlist -u https://target/FUZZ

# Multiple sources
ffuf -w http-methods:METHOD -w payloads:PAYLOAD -w headers:HEADER -u "https://example.com/PAYLOAD" -H "HEADER:127.0.0.1" -X "METHOD"

# Multiple URLs and mutiple files example
ffuf -u URL/FUZZ -w listofurls:URL -w listofpaths:FUZZ -maxtime 300 -t 500 -c -v

# Cool ffuf flags
-ac: Calibrate requests to unmatch false positives
-recursion: recursion
-se: Stop on erors
-sf: Stop on 95% 403 Forbidden, possible WAF ban
```

## Remove noise

Ffuf can generate a large output, even with `-ac` flag enabled. To filter dummy and extract only juicy information, it is possible to use `ffufPostprocessing`

```
# https://github.com/Damian89/ffufPostprocessing
ffufPostprocessing -result-file /tmp/ffuf/results.json -bodies-folder /tmp/ffuf/bodies/ -delete-bodies -overwrite-result-file
```

## Backup Files

Tempers file to find possible backup files based in file name

```
# https://github.com/mazen160/bfac
bfac --no-text --url http://example.com/test.php --level 2
```

## Recollapse

Generate a bunch of breaking-strings to test your target

- [https://github.com/0xacb/recollapse](https://github.com/0xacb/recollapse)

## Useful links

|URL|Description|
|---|---|
|[https://github.com/p0dalirius/webapp-wordlists](https://github.com/p0dalirius/webapp-wordlists)|This repo contains wordlists for a lot of webapps|
|[https://github.com/six2dez/OneListForAll](https://github.com/six2dez/OneListForAll)|Huge list of paths and files|
|[https://www.acceis.fr/ffuf-advanced-tricks/](https://www.acceis.fr/ffuf-advanced-tricks/)|ffuf tricks|
|[https://github.com/Escape-Technologies/graphql-wordlist/tree/main](https://github.com/Escape-Technologies/graphql-wordlist/tree/main)|Initially for graphql, works everywhere|