For large recons where manual website visit is not doable. The program would grab a list of valid urls and screenshot them using a headless browser.

> **Hold down!**
> 
> A browser must be installed prior using an screenshotter. Chrome or chromium is recommended.

## GoWitness

```
# https://github.com/sensepost/gowitness
gowitness file -f websites.txt -t <threads> 
```

## Other

HttpX can be used ot capture screenshots

```
echo example.com | httpx -ss
```