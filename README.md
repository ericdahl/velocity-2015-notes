# Putting web performance best practices together to create the perfect single page application

Chris Love, Love 2 Dev

[slides](http://www.slideshare.net/docluv/putting-performance-best-practices-together-for-a-spa) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41656)

- mobile browsers
  - aggressively purge cache
    - iOS 3 Safari had a 26KB limit on cache items
  - less memory
  - weaker processsor
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
  - business availability: pre-emptively identify issues before they become problems
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

# The principles of microservices

Sam Newman (ThoughtWorks) 

[slides](http://www.slideshare.net/spnewman/principles-of-microservices-velocity) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41106)

- wrote book "Building Microservices"
- microservices
  - modeled after business domain rather than tech boundaries
  - small autonomous services that work together
  - "small enough and no smaller"
- success stories: Gilt, REA, Netflix
- microservices give more choices
  - DBs, deployment platform, etc
  - but this means that it can take time tofigure out what path to take (exhausting)
  - organizations are not used to being able to make choices
    - i.e., the monolithic app usually dictates the framework, language, etc
  - should have some framework to help with this
    - 12 factor app
      - goal: if you follow these, your software will work well on heroku
- 8 Principles of Microservices
  - 1. Model after the business domain
    - before, everyone had the same 3 tier architecture (presentation, business logic, data access)
      - issues:
        - simple change often slices between all of the layers
        - each layer is often managed by different people
    - when boundaries are around a business domain
      - a change to what the business does is more likely to sit inside of a service boundary
      - when teams own services, they can become experts in that domain
      - more obvious and clear boundaries
        - less breaking API changes
      - how to find these boundaries?
        - ask "what are the capabilities that the system offers"?
          - e.g., registering a new customer, etc
          - collect similar capabilities and organize them into services
        - see "Implementing Domain-Driven Design" by Vaughn Vernon
  - 2. Culture of Automation
    - story:
      - at REA, they had a monolithic system which they had been investing in automating to cloud
      - wanted to move to smaller units
      - goal: microservices by 3 months
      - after 12 months, they finally had 10
      - but after 18, they had 60
        - once they had the infrastructure/automation in place, growth exploded
  - 3. Hide Implementation Details
    - had thing to get right
    - allow services to remain independent so they can change without impacting each other
    - common 'service-to-service' integration pattern
      - two services read/write to the same DB schema
      - this makes it very difficult to make changes; everything is visible
        - so, hide your databases, exposing them only via APIs
    - serializing complex nested domain objects often exposes more info than ideal
    - could create client libraries to call service
      - this often starts to take on implementation logic
      - "be careful of client libraries"
      - beware of DRY for this
        - DRY is okay for within a single process
        - but enforcing DRY across multiple processes/organizations by creating libraries can introduce coupling
  - 4. Decentralize all the things
    - embrace self-service
      - do you have to go to another team to launch new VMs?
    - but should have some shared governance
      - e.g., to decide what DBs to use
    - internal open source
      - use pull request to request changes for another team
        - requires high level of organizational maturity
    - dumb pipes, smart endpoints
      - middleware should not have behavior/logic; should be dumb
      - logic should be in the fringes (services)
    - how to organize business workflow?
      - orchestration
        - service which is the brains of the operation, calling other services
        - not ideal
          - logic is gradually drained from the edge services into this orchestration service
        - choreograph based approach
          - each service knows what its role is in the ssytem
            - e.g., submits events to some topic where other services may be listening
            - can be more resiliant
            - but,
              - if a step fails, it's harder to debug
              - process is defined implicitly rather than explicitly (as in orchestration)
                - so you need something to monitor for system invariants
  - 5. Deploy Independently (most important of the 8)
    - should be able to make a change to a service and deploy it to prod without having to deploy anything else
    - if you have two services that you have to deploy together, fix it
    - one service per host (or container or VM)
      - when you have multiple services in one host/container/VM, hard to reduce side effects
        - e.g., if a service uses all the CPU, others impacted
    - customer driven contracts
      - end-to-end-testing
        - start up all of the services and then test them together
        - doesn't scale well
        - see http://googletesting.blogspot.com/2015/04/just-say-no-to-more-end-to-end-tests.html
        - see pact library for this: https://github.com/realestate-com-au/pact
      - e.g., shipping -> inventory service dependency
        - shipping service has a set of expectations of inventory service
        - inventory service should have set of tests based on each of its clients expectations (e.g., shipping)
      - how to implement a breaking change?
        - don't try to do a lock step release
          - consumers need to be in charge of deciding when they release
        - co-exist endpoints
          - create a v2 endpoint alongside v1
          - give clients time to migrate to v2 from v1
  - 6. Consumer First
    - need to think about consumer's needs before tech details
    - documentation
      - make it easy for people to consume service
      - swagger is good for this
    - API Gateways
      - skeptical of generic ones
      - api keys are useful in determining who is using the service
      - (Sam is looking into 3scale which looks nice)
    - Service Discovery
      - using tools like zookeeper, etcd, consul
      - allows machines to register themselves for others to find them
      - (Sam is particularly impressed with Consul)
  - 7. Isolate Failure
    - if a service breaks and your entire system breaks, you don't have a SOA but instead a monolith chopped up
    - avoid having a "distributed single point of failure"
      - story: classified ads website for a print company which turned itself into an online company
        - ad-hoc IT company
        - used Strangler App Pattern
          - intercept calls to old system and potentially redirect to the new system
          - was working well but had an issue with connections to downstream services
          - downstream service was failing in worst possible way, by being really slow
          - fixed in 3 ways
            - fixed timeout values
            - added one thread pool per downstream service
            - added circuit breakers
              - protect you if something breaks
              - after a certain number of timeouts/errors, mark the node as failed and fail-fast
              - useful for unplanned outage but also planned outage
                - flip circuit breaker, deploy new service, flip it back on
              - for java, use hystrix
    - if you go into microservices blindly, you'll be badly bitten
  - 8. Highly Observable
    - each independent service should expose its health on some status page
      - put business metrics here too
    - aggregation
      - treat nodes as stateless and move info to a central place
      - logs: use ELK stack for on-site
      - stats
        - on-prem: graphite/statsd
        - off-site: new-relic
      - Correlation IDs
        - how to track down the context in which something happened?
        - originating request gets some ID assigned to it
        - ID is passed through the system
        - even better? using parent and child IDs so you can generate call graphs
        - important to put this in earlier rather than later; may be hard to add
      - semantic monitoring
        - ensure that the system is working rather than focusing on low-level checks
        - e.g., could run end-to-end test cases in production for some things
        - if one service is at 100% is that a problem? not necessarily if many nodes
      
# Putting the network to work

Manish Vachharajani, F5 Networks 

[velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/42768)

- avoid render blocking by js, even if script is at the end of the body
- many pages still don't enable transport compression (http gzip)
  - though for some cases, this can add some latency to compress/decompress (ed: what cases? already compressed data?)
- css @imports
  - clean code but causes sequential loading
- limited to 6 connections for the same domain
- semantic compression: minification or jpg
  - dynamically size images for the device
- NY to London RTT is 28ms
  - 84 ms to establish TCP connection
  - 224 ms to establish https connection
  - google recommends aiming for 200-300ms for page render
- TLS optimizations
  - first, use https://www.ssllabs.com/
  - session re-use
    - session tickets
      - rather than session cache (where often the caches aren't shared among https terminator servers)
      - eliminates a round trip
    - SSL false start
      - guessing what cipher the server will want to use and pretransmit http request along with message
      - eliminates a round trip
      - browsers use heuristics to determine this
    - OCSP stapling
      - client has to validate that the server cert is legit
        - could download the CRL to determine this
          - this is a new connection/DNS lookup and overhead
          - instead, use OCSP so the server sends along cert verification to client
- (SPONSOR PART)
  - smart proxies (like BigIP): alternative to doing it yourself
    - can inline js/css and images
    - can optimize images
    - can do URL rewriting to help with browser caching

# Failure is an option

Ian Malpass, Etsy 

[slides](https://speakerdeck.com/indec/failure-is-an-option) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/4168)

- 3 truths
  - 1. you will create bugs
  - 2. you will build the wrong thing (understanding of requirements is not perfect)
  - 3. you will not foresee th unexpected
- failure is inevitable
  - expensive failure is not
  - traditional response: erect barriers to prevent people from doing harm
  - if we spend time and effort hiring good people, need to turst them to do the right thing
- deploys
  - at etsy, can deploy master at any time with minimal ceremony
  - uses https://github.com/etsy/deployinator
  - should be fast and easy
    - this means that deploys can happen often
    - this means that changes are smaller (and less risky)
    - code reviews
      - smaller changes are easier to review
      - large diffs: nothing is found by reviewers because it is overwhelming (missed bugs)
- manual testing
  - very little of this is done
  - they have minimal QE resources, usually focusing on areas that are tough to test
  - QA is a job function of all engineers
  - QA should be a partner rather than a barrier at end of development
- security
  - like QA, collaborate rather than act as gatekeepers
  - partner with them to get feedback early
- automated testing
  - more tests that are complex means that it takes longer to run
    - means that deploys take longer (e.g., 1h to deploy)
    - they found that a small minority of the tests take the majority of the time
      - deleted many fo these because they were often obscure with minimal benefit
      - no tolerance for flaky tests (tests which pass/fail at random)
        - refactor or delete
- try command
  - etsy internal tool
  - run tests on full blown staging environment prior to pushing
- graphs
  - aggregate metrics to combine multiple
    - e.g., their "screwed users" metric
  - graphs have dotted lines to indicate when deploys happen
  - engineers should be able to make their own dashboards
- dark code
  - code that's not executed
  - this allows for shipping code incrementally but not necessarily releasing it
  - feature flags
    - to turn feature on/off
    - individual engineers can turn a flag on
    - can enable some percentage of traffic to use new feature
    - increment percentage as desired
      - if there's some bug, just drop it to zero
    - this is the closest thing to a feature launch at etsy
    - sort of like A/B testing
  - prototype
    - allow users to opt-in to new features
- we still fail
  - culture of blamelessness
    - this is core to how we learn from failure
    - have postmortems after something goes wrong
      - share results with everyone
- celebrate ailure
  - 3 armed sweater awad
    - awarded to the individual or team who breaks site trying to do something spectacular
    - learn from the mistake

# Crafting performance alerting tools

Allison McKnight, Etsy

[slides](https://speakerdeck.com/aemcknig/crafting-performance-alerting-tools) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/42804) | 

(no notes; standing room only)

# osquery: Approaching security the hacker way

Mike Arpaia, Facebook

[slides](https://speakerdeck.com/marpaia/osquery-approaching-security-the-hacker-way) | [velocity page](velocityconf.com/devops-web-performance-2015/public/schedule/detail/41762)

- other talks go into more depth on osquery itself
- secrecy has stifled innovation
  - not much software out there to help with modern attacks
  - we implement the same fixes, poorly
 - Attacker Math 101
  - see https://www.trailofbits.com/resources/attacker_math_101_slides.pdf
  - economics behind how attackers think
  - attack graphs to determine what attackers will focus on
- osquery
  - performant host instruementation tool with sql interface
    - issue sql query and get back live information at runtime
    - tables for
      - running processes
      - loaded kernel modeules
      - route tables
      - installed software
      - active network connections
      - etc
    - osqueryi
      - interactive mode
      - a sql console
    - never shells out (e.g., by using ```ps```)
    - osqueryd
      - daemon for host monitoring
      - can periodically execute queries to find state changes
      - logs results for aggregation and analytics
  - has file integrity checks
    - e.g., to know if a file /bin/** has changed
  - design decisions
    - expose capabilities safely
      - tables allow us to give away answers without giving away the questions (for security issues)
      - capabilities can be configured rather than developed
        - good for security people who may not be comfortable developing new plugins
      - MIDAS
        - predecessor to osquery
        - drawbacks
          - required writing code in python to create a new check
          - internal facebook and external code diverged
            - osquery is developed 100% opensource on github

# Enabling microservices at Orbitz

Steve Hoffman and Rick Fast, Orbitz

[slides](http://cdn.oreillystatic.com/en/assets/1/event/122/Enabling%20microservices%20at%20Orbitz%20Presentation.zip) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/40700)

- 2010: only releasing twice a year
  - now it's about 1-4 days per release
- experiment: decompose a "monolithic service" into 40+ subservices
  - using spring-boot app in docker container
  - uses consul which runs on every container
  - network: VIP -> multiple haproxy instances -> spring-boot 
- before continuous delivery
  - build -> test -> discard build
  - instead, deploy the successful build
- used yeoman to help generate java microservices
  - builds up a chassis with libraries to set up logstash connection etc
  - generated code is pushed to a new repo (in atlassian stash)
  - once code hits master its not going to stop until its pushed to production
    - uses pull requests
    - once reviewers click OK, merge to master
- pipeline
  - build/unit-test/publish
    - pull latest version from master (with gradle)
    - updates version number to a RC version
    - creates a docker container which has the spring-boot jar along with consul/logstash forwarders
    - smoke/acceptance tests use embedded DBs in spring-boot framework
  - deploy
    - using ansible
    - deploys docker image to some host group (e.g., dev)
    - spring-boot
      - uses custom health checks at /health
        - used with consul to ensure things are running ok
    - once new version is up and /health is OK, gracefully stop previous version
      - keep doing this until all deployments are done
      - took 30 minutes to serially update every node
      - now uses marathon with mesos
- spring-boot configuration
  - components configure themselves based on what environment they are in
    - e.g., using graphite vs aws cloudwatch
    - swagger
    - hystrix
    - retrofit + consul
- summary
  - use localhost helpers with docker deployments
  - people don't scale, use automation
  - deliniate configuration concerns
    - if it's known at compile time -> build into docker image
    - known at boot time -> build into ansible playbook/launcher
    - if it changes anytime -> externalize with consul or etcd or zookeeper
    
# Burnout in tech

John Allspaw (Etsy), Christina Maslach (UC Berkeley), Amanda Folson (PagerDuty), Katherine Daniels (Etsy), Laura Bell (SafeStack Limited), Vijay Gill (Salesforce.com) 

[velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/43153)

- burnout has 3 components
  - 1. exhaustion
    - primarily due to work overload
    - results in problems related to personal health
    - possibly results in sacraficing personal relationships
  - 2. cynicism (main component of burnout)
    - very negative view of the job; "take this job and shove it"
    - lost motivation/passion
    - does bare minimum effort
  - 3. professional inefficacy
    - this is seen later in burnout process
    - negative feelings turn in on oneself
    - precursor to mental health issues (e.g., depression)
- burnout as a stress phenomenon
  - chronic form of stress; not a result of a specific incident
  - "erosion of my soul"
  - others may notice it in you prior to you noticing
- outcomes of burnout
  - poor quality of work
  - low morale
  - absenteeism
  - turnover
  - health problems
  - depression
  - family problems
- six strategic areas
  - if there's a mis-match between a person and the job
  - workload
    - what most people think of as a burnout cause
    - demands are too high; resources too low
    - one of the best predictors of the exhaustion component of burnout
    - most people think this is the main cause but often it isn't
  - control
    - how much say/discretion/choice you have about how you do your job
    - do you have a choice or are you micromanaged? 
  - reward
    - principle of recognition and positive feedback of doing something well
    - if you do something well, do people notice?
      - often a pay increase/bonus isn't as valuable as simple recognition
  - community
    - relationships with coworkers, boss, vendors, etc
    - can you trust each other?
    - when this goes bad, the workplace becomes toxic
    - signs: bullying, gossiping, politics
  - fairness
    - if people feel like they aren't being treated fairly, cynicism and burnout result
    - people must think they are being treated fairly (if they are but they don't think they are, it doesn't count)
  - values
    - what you're doing shouldn't be in conflict with what you value
    - leads to "erosion of soul"
- standard approach for burnout is to ask "what's wrong with this person?" or "how to fix this person?"
  - usually it's more widespread than one person
  - need to think about how to improve work environment
- what to do about burnout
  - preventing it is a better strategy than waiting to treat it
  - building engagement is the best approach
  - organizational intervention often more productive than individual intervention
- book references
  - The Truth About Burnout
    - recommended particularly
  - Banishing Burnout
- question panel
  - Vijay: high competency gap leads to issues
  - hard to know how to fix burnout because not many people evaluate what they try

# Mobile image processing

Tim Kadlec, Akamai

[slides](https://speakerdeck.com/tkadlec/mobile-image-processing-at-velocity-sc-2015) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41625)

- 2012: 10% of every photo ever taken up to that point was taken in that year
  - 1839: modern photography was invented
- 1.8 billion photos shared each day
- sites are now around 2MB with 63% of the website being images
- decoding jpg images
  - browser has to go through ~4 steps to decode
  - chrome shows image decode time in dev tools
  - ie/edge does as well; easy to use
  - firefox/safari don't expose
  - experiment
    - website with about 10 images, where some pages have them appropriately sized and some are full size
    - on desktop: 
      - appropriately sized: 8ms to decode
      - 400px (for 200px size): 26ms
      - 6x200pz: 266ms (+3000% increase)
    - results on nexus 4
      - appropriately sized: 30ms to decode
      - 400px: 100ms
      - 6x size: 1500ms (5000% increase)
    - rule number 1: resize images for the device
      - applies to sprites too, which may be larger than necessary
      - simply reducing size by 50px is minor but saves a lot fo bytes
      - for larger sizes, have more breakpoints
  - 4:2:0 subsampling has large memory gains
  
# Teaching GitHub for Poets

[velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/41347)

- everyone in Kickstarter can commit code
  - GitHub for Poets is a class used to teach non-techies how to commit code to make fixes, usually for small things like typos and documentation
  - uses pull requests
    - someone watches and approves these within an hour or so (which then are deployed to production continuously)
- why do github for poets?
  - its a lightweight process
  - before:
    - notice a typo
    - send report to manager
    - manager tells other manager at weekly meeting
    - typo bug is put into some list
    - ticket is filed
  - cultural values
    - transparency and inclusivity
    - pull request process builds consensus
    - version control is communication
    - 
# Yesterday's perf best-practices are today's HTTP/2 anti-patterns

[slides](http://bit.ly/http2-opt) | [velocity page](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/42385)

- half of browsers already support http2
- chrome will deprecate SPDY (and NPN) in early 2016
- HTTP2 intro
  - premise: optimizing for low latency delivery
  - we are latency bound
    - increasing bandwidth has diminishing returns
  - uses one tcp connection
    - multiplexed and prioritized streams
    - binary framing layer
    - header compression
      - http 1.1: headers were uncompressed (which can be significant with cookies)
  - HPACK header compression
    - optimizes sent headers with huffman encoding and references to previously sent headers
  - for more info, see High Performance Browser Networking
- on http1, 74% of active connections carry just a single transaction (Patrick McManus, Mozilla)
- domain sharding
  - generally bad for http2
  - this is the single biggest problem to fix when switching to http2
- http2 can coalese connections on your behalf if
  - tls cert is valid for both hosts
  - hosts resolve to the same IP
- concatenation/minifaction/spriting
  - http1.1: careful use is good
    - improved compression
    - but some files can't be streamed (css)
    - single byte update causes a full fetch
      - inefficient, particularly if concat js is mostly unchanging libraries
  - http2: avoid it
- inlining (embedding js/css/images in html)
  - resources can't be cached independently
  - breaks multiplexing and prioritization in http2
  - http1.1: use carefully
  - http2: replace with server push
    - jetty has smart push
      - uses referer header to build a map to guess what resources will be requested
      - automatic
- tl;dr
  - remove:
    - domain sharding
    - concatenation
    - inlining
  - pick your http2 server carefully
  
- question: current state of http2 in servers?
  - http2 working group has a link
  - nginx will support by end of the year, but nothing is out yet
  - h2o
  - nghttp
  - nodejs module
  - apache has a module you can build in
  - apache traffic server "best version implemented as of today"
