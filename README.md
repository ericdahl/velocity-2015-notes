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

[slides](http://cdn.oreillystatic.com/en/assets/1/event/122/Building%20an%20effective%20observability%20stack%20%20Presentation.pdf) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41148)
- objectives of observability
  - business velocity
  - business availability: preemptively identify issues before they become problems
  - business efficiency: cost, resource utilization
  - business scalability: identifying constraints and bottlenecks
- principles of observability
  - store business and operations together (don't silo tech / business data)
  - store at low resolution for core KPIs
  - support self-service visualization
    - people should be able to create their own dashboards
  - keep your architecture simple and scalable
  - democratize data for all
    - everyone should be able to access data
  - collect and store once
- outcomes of observability
  - high trust and transparency
  - continuous deployment
  - engineering velocity
  - happy staff
  - cost efficiency
- traditional monitoring
  - has many datastores/collectors
  - problems:
    - too many dashboards (too many places to go)
    - recording data in multiple places
    - resolution too high (e.g., recording every minute rather than second)
    - hard to automate
    - logs not centralized
  - better monitoring
    - use sensu rather than nagios for variable time polling
    - 1 second resolution time
    - supports ephemeral services
  - use Apdex for performance tracking
  - measure as much as possible; alert on as little as possible
  - automate remediation if possible
    - spin up new instances to meet demand
    - kill/restart unresponsive instances
  - PVOS (Pythian Operation Visibility Stack)
    - resides in AWS via cloudformation and opsworks
    - only externally accessible services are rabbitmq and dashboards
    - why?
      - everybody needs monitoring
      - everybody builds it slightly differently
      - but they all have broad similarities
      - github has 160+ "elk stack" repos, much of which is broken
    - design
      - sensu agents push to rabbitmq 
      - event handlers in sensu review then forward to graphite
    - visualization
      - logs/events: kibana
      - telemetry: grafana
      - incidents/alerts: uchiwa
    - misc
      - scaling graphite is a pain because you have to specify what type of node each is and set up hashing algorithm
