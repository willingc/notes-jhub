## Proxies

### Forward

### Reverse

From issue 219:

> Older versions of Apache such as 2.2.x do not provide WebSocket support, therefore a newer version of Apache (i,e 2.4) included in the upcoming Debian Jessie release, supplies proxy_wstunnel for WebSocket applications. For Jupyterhub, for any URL spaces /(user/[^/]*)/(api/kernels/[^/]+/channels|terminals/websocket)/?, forward to ws(s)://servername:port_number, all other standard spaces, forward to http(s)://servername:port_number, that will do the trick!

```
ProxyPass / http://localhost:8000/
ProxyPassReverse / http://localhost:8000/

Header edit Origin {external address} localhost:8000
RequestHeader edit Origin {external address} localhost:8000

Header edit Referer {external address} localhost:8000
RequestHeader edit Referer {external address} localhost:8000
<Location ~ "/(user/[^/]*)/(api/kernels/[^/]+/channels|terminals/websocket)/?">
    ProxyPass ws://localhost:8000
    ProxyPassReverse ws://localhost:8000
</Location>
```

*@mbmilligan*
The **config for websocket proxying** quoted by @franga2000 didn't work for me either. What DID work was instead **being explicit about the URL matching**:

```
<LocationMatch "/mypath/(user/[^/]*)/(api/kernels/[^/]+/channels|terminals/websocket)(.*)">
    ProxyPassMatch ws://localhost:8999/mypath/$1/$2$3
    ProxyPassReverse ws://localhost:8999 # this may be superfluous
</LocationMatch>
```

For what it's worth, this is using Apache 2.2 on CentOS, with the wstunnel backport patches described here: https://gist.github.com/vitkin/6661683

*@ryanlovett*
Just adding another works for me example. Apache 2.4.7 in Ubuntu 14.04 suffers from a websocket bug that prevents notebook sessions from working properly in this context.

```
ProxyPreserveHost On
ProxyRequests off

ProxyPass /api/kernels/ ws://127.0.0.1:8888/api/kernels/
ProxyPassReverse /api/kernels/ https://127.0.0.1:8888/api/kernels/

ProxyPass / https://127.0.0.1:8888/
ProxyPassReverse / https://127.0.0.1:8888/

<Location ~ "/(user/[^/]*)/(api/kernels/[^/]+/channels|terminals/websocket)/?">
    ProxyPass ws://localhost:8888
    ProxyPassReverse ws://localhost:8888
</Location>
```

*@mbmilligan*
You definitely do need to proxy the main paths and the websockets separately. A couple more notes:

- you spell the url "/notebook" in one place and "/notebooks" in another, check for misspellings in your config;
- the Hub may need to be configured to listen to an external interface rather than localhost;
- recent versions will complain if started without SSL, so if you are terminating SSL at your proxy server (which is the common thing to do here) you may need to add the --no-ssl option



I had to enable proxy, proxy_http, headers and proxy_wstunnel.

*@lbiemans*
What works for me is:

```
    ProxyPass / http://localhost:8000/
    ProxyPassReverse / http://localhost:8000/

    Header edit Origin HOSTNAME.TLD localhost:8000
    RequestHeader edit Origin HOSTNAME.TLD localhost:8000

    Header edit Referer HOSTNAME.TLD localhost:8000
    RequestHeader edit Referer HOSTNAME.TLD localhost:8000
    <Location ~ "/(user/[^/]+)/(api/kernels/[^/]+/channels|terminals/websocket)/?">
            ProxyPass ws://localhost:8000
            ProxyPassReverse ws://localhost:8000
    </Location>
```

Don't forget to enable proxy, proxy_http, headers and proxy_wstunnel.


### Checklist

- [ ] Review working configurations in https://github.com/jupyterhub/jupyterhub/issues/219
- [ ] Review Apache version.
     * Apache 2.4 or higher
     * 2.2.x does not provide WebSocket support.
     * 2.2 on CentOS will work with wstunnel backport patches (See https://gist.github.com/vitkin/6661683).
- [ ] have you enabled `mod_proxy_wstunnel`?
- [ ] enable proxy, proxy_http, headers and proxy_wstunnel.
- [ ] Checked Apache logs?
- [ ] Check for mispellings in config (i.e. notebook vs notebooks). Be consistent.
- [ ] May need to be configure the Hub to listen to an external interface rather than localhost
- [ ] Note: recent versions of JupyterHub and single user notebook servers will complain if started without SSL, so if you are terminating SSL at your proxy server (which is the common thing to do here) you may need to add the --no-ssl option


### Resources

[Apache Forward and Reverse Proxies](http://httpd.apache.org/docs/2.0/mod/mod_proxy.html#forwardreverse)