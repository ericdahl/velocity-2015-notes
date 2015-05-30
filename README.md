# Putting web performance best practices together to create the perfect single page application

Chris Love, Love 2 Dev

[slides](http://www.slideshare.net/docluv/putting-performance-best-practices-together-for-a-spa) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41656)

- mobile browsers
  - aggressively purge cache
    - iOS 3 Safari had a 26KB limit on cache items
  - less memory
  - weaker processosr
- cellular connections
  - devs should be testing on connections like this rather than local
  - unreliable, losing connections
  - more expensive (see http://whatdoesmysitecost.com/)
- the web is obese (see http://httparchive.org/)
- "fast food frameworks"
  - angular, ember, jquery, react, etc
  - most aren't modular and have memory leaks
  - synatic sugar means browsers don't know browser APIs
  - see http://microjs.com/ for alternatives
    - list of really small libraries that are < 5kb gzipped
- SPA goal
  - enough information to render page in the first 14KB (ed: for first round trip ?  outdated based on old icwnd?)
- cutting the mustard
  - feature detection
    - if the client doesn't have some key browser features, then route them to legacy website
  - don't use polyfills


# Building an effective observability stack 

Laine Campbell, Pythian
