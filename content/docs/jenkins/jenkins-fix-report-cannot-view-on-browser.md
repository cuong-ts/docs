# Jenkins fix report cannot view on browser

```groovy
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "style-src 'self' 'unsafe-inline';")
```



