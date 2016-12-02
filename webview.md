## cookie 设置
```java

  public synchronized void fillCookie() {
        if (!SDKManager.getInstance().getHostInfoExtractor().isLogged()) {
            return;
        }
        CookieSyncManager.createInstance(SDKManager.getInstance().getApplicationContext());
        final HostInfoExtractor extractor = SDKManager.getInstance().getHostInfoExtractor();
        String[] hosts = HostWhiteListManager.get().getAll();
        for (String host : hosts) {
            setCookie(host, "wctk=" + extractor.getToken());
            setCookie(host, "access_token=" + extractor.getToken());
            setCookie(host, "X-Access-Token=" + extractor.getToken());
            setCookie(host, "X-ACCESS-TOKEN=" + extractor.getToken());
            setCookie(host, "_wac_cookie_flags=1");
        }
        flush();
    }

    public synchronized void clearCookie() {
        CookieSyncManager.createInstance(SDKManager.getInstance().getApplicationContext());
        String[] hosts = HostWhiteListManager.get().getAll();
        for (String host : hosts) {
            setCookie(host, "wctk=");
            setCookie(host, "access_token=");
            setCookie(host, "X-Access-Token=");
            setCookie(host, "X-ACCESS-TOKEN=");
            setCookie(host, "_wac_cookie_flags=0");
        }
        flush();
    }

    public void setCookie(String url, String cookieStr) {
        try {
            CookieManager cookieManager = CookieManager.getInstance();
            cookieManager.setAcceptCookie(true);
            cookieManager.setCookie(url, cookieStr + ";domain=." + url);
        } catch (Exception e) {
            Log.e("WacCookieManager", "setCookie failed", e);
        }
    }

    private void flush() {
        try {
            CookieSyncManager.getInstance().sync();
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                CookieManager.getInstance().flush();
            }
        } catch (Exception e) {
            Log.e("WacCookieManager", "flush failed", e);
        }
    }

```
