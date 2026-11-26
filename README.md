# Docker VPS hosting: is 2 GB enough for your compose stack, which Sixtynet plan fits, and why some cheap plans block ports 80 and 443

A single small app plus its database fits in 2 GB of RAM. A busier stack with several services and a real database wants 2 to 4 vCPU and 4 to 8 GB. Those two numbers settle most of the plan choice before price even comes up.

There is a second filter that catches people shopping on price alone. Some inexpensive VPS plans sit behind IP policies that close ports 80 and 443, and Sixtynet sells three such lines. If you plan to serve a website or an API out of a container, that rules those plans out before you compare a single spec.

What follows is the sizing math, the full Sixtynet price list as it currently stands, which line actually works for a public Docker stack, and the two things worth confirming before you pay.

> **Quick pick.** One app plus a database, serving the public: Sixtynet **LAX-STARTER**, 2 vCPU / 2 GB RAM / 40 GB NVMe / 5 TB traffic, **$200 per year** on the annual term (listed at $240). Running five or six services with a database and a reverse proxy: **LAX-MICRO**, 4 vCPU / 4 GB / 80 GB NVMe / 10 TB, **$400 per year** (listed at $480).
> 👉 [See the current LAX plan specs and pricing at Sixtynet](https://sixtynet.net/aff.php?aff=39)

## Every Sixtynet plan on the current price list

Sixtynet quotes annual pricing with a discount already applied, and the page shows both the list figure and the discounted one. The annual saving works out to 13% to 17% depending on the plan. Quarterly terms are also discounted, though the page doesn't publish the exact rate.

| Plan | vCPU / RAM | Storage | Traffic | Annual price (USD) | Order |
| --- | --- | --- | --- | --- | --- |
| LAX-STARTER | 2 / 2 GB | 40 GB NVMe | 5 TB | $200 (list $240) | [Order LAX-STARTER](https://sixtynet.net/aff.php?aff=39) |
| LAX-MICRO | 4 / 4 GB | 80 GB NVMe | 10 TB | $400 (list $480) | [Order LAX-MICRO](https://sixtynet.net/aff.php?aff=39) |
| LAX-MEDIUM | 8 / 8 GB | 160 GB NVMe | 20 TB | $800 (list $960) | [Order LAX-MEDIUM](https://sixtynet.net/aff.php?aff=39) |
| LAX-LARGE | 8 / 16 GB | 320 GB NVMe | 40 TB | $1,600 (list $1,920) | [Order LAX-LARGE](https://sixtynet.net/aff.php?aff=39) |
| Residential-STARTER | 2 / 2 GB | 40 GB NVMe | 5 TB | $260 (list $300) | [Order Residential-STARTER](https://sixtynet.net/aff.php?aff=39) |
| Residential 4-core tier | 4 / 4 GB | 80 GB NVMe | 10 TB | Price on the pricing page | [Check the residential tiers](https://sixtynet.net/aff.php?aff=39) |
| Residential 8-core tier | 8 / 8 GB | 160 GB NVMe | 20 TB | Price on the pricing page | [Check the residential tiers](https://sixtynet.net/aff.php?aff=39) |
| Residential 16-core tier | 16 / 16 GB | 320 GB NVMe | 40 TB | Price on the pricing page | [Check the residential tiers](https://sixtynet.net/aff.php?aff=39) |
| IRVINE-PRO-STARTER | 2 / 2 GB | 40 GB NVMe | 5 TB | $350 (list $420) | [Order IRVINE-PRO-STARTER](https://sixtynet.net/aff.php?aff=39) |
| Irvine 4-core tier | 4 / 4 GB | 80 GB NVMe | 10 TB | Price on the pricing page | [Check the Irvine tiers](https://sixtynet.net/aff.php?aff=39) |
| Storage VPS | 4 / 4 GB | 300 GB + 200 GB bonus, RAID 10 | 10 TB | Price on the pricing page | [Check the Storage VPS](https://sixtynet.net/aff.php?aff=39) |

Two rows carry no figure because the price wasn't part of what I could confirm from the page, so check those directly rather than trusting a number from a review. Everything with a figure is quoted as an annual total, not a monthly rate.

Disk speed splits along the same lines as core count. The 2- and 4-core plans are listed at 12,500 IOPS, the 8- and 16-core plans at 25,000. That matters more for Docker than it sounds, and I'll come back to it.

## Which line you actually want for containers

The four product lines look similar in the table and behave very differently in practice.

| Line | IP and network | Public ports 80 / 443 | DDoS protection | Suits |
| --- | --- | --- | --- | --- |
| LAX | Datacenter IP, 2 Gbps to 10 Gbps premium network | No restriction listed | Yes | Web apps, APIs, dashboards, anything public |
| Residential | Residential IP, 2 Gbps to 10 Gbps premium network | Closed by policy | Not listed | Scraping, social accounts, work that needs a home-looking IP |
| Irvine | Residential IP, AS9929 China-optimized route, whitelist-only | Restricted, external probes rejected | Not listed | Private access over a China-friendly route |
| Storage | Datacenter, RAID 10 array, 10 Gbps network | No restriction listed | Yes | Backups, media libraries, data-heavy low-CPU jobs |

Sixtynet prints the warning on the residential product itself: because of IP risk control, ports 80 and 443 are not opened. The Irvine plans go a step further and reject all external probes and scans unless you whitelist or port-forward, which is fine for a private box and useless for a public site.

For Docker hosting specifically, that leaves the LAX line. It's the one sold with DDoS protection and without the port restriction, and it's the only line where `docker compose up` followed by a DNS record actually results in a reachable site.

The same price list mentions a 100M base network with 3x burst and 95th-percentile billing, plus free BGP session setup and limited BGP technical support. If your containers pull large images or push large uploads, ask about how the 95th-percentile measurement interacts with your traffic allowance before you commit to the smallest plan.

## What Docker itself eats, and what's left for your services

Docker Engine uses roughly 200 MB of RAM and very little CPU when it's idle. The real budget goes to the containers on top. A container shares the host kernel and adds single-digit megabytes of overhead, while a virtual machine needs 1 to 2 GB just to run its own operating system. That gap is why a 2 GB VPS is a working machine rather than a toy.

Take the 200 MB off a 2 GB plan and you have about 1.8 GB for workloads. A single small app and a database fit comfortably. A production stack with a reverse proxy, an app, Postgres, Redis and a background worker is where 2 GB starts swapping, and that's the point at which the spec sheet stops telling you anything useful and the OOM killer starts telling you everything.

Rough mapping against Sixtynet's tiers:

- **One app, one database, low traffic.** LAX-STARTER at 2 vCPU / 2 GB. Nextcloud or a small API with SQLite, or a single Node/Python service with Postgres.
- **Two to four services behind a proxy.** LAX-MICRO at 4 vCPU / 4 GB. Typical shape: app container, Postgres, Redis, Nginx Proxy Manager or Caddy, plus Portainer for a UI.
- **A media server that transcodes, or a busy multi-tenant stack.** LAX-MEDIUM at 8 vCPU / 8 GB. Jellyfin and Plex burn CPU during transcodes, and that's the workload where extra cores convert directly into fewer buffering complaints.
- **Several projects on one box, or a database with heavy writes.** LAX-LARGE at 8 vCPU / 16 GB, where the 320 GB and 25,000 IOPS give the volumes room to breathe.

That last point is worth spelling out. Slow storage bottlenecks Docker volumes before it bottlenecks the CPU. A database container on a disk that can't keep up with writes will show up as query latency long before the CPU graph moves, which is why the IOPS figure on the spec sheet is worth more attention than the core count for anything Postgres-shaped.

## Preinstalled Docker vs installing it yourself

Sixtynet's price list doesn't advertise a Docker image, so plan on installing Engine yourself over SSH. On a Linux VPS that's a few commands from Docker's own repository, and it has an advantage: you get the exact Engine and Compose versions your project expects instead of whatever the template shipped with.

That last part matters if you pin versions in CI, need Compose v2 specifically, or intend to run k3s on the same box. Ripping out a preinstalled engine to install a specific one is more work than a clean install, so a plain OS image is often the faster path for anyone who already knows their way around a shell.

👉 [Start from a clean OS image on a Sixtynet LAX plan](https://sixtynet.net/aff.php?aff=39)

## Annual pricing, and what hourly billing costs elsewhere

Sixtynet's public pricing is built around annual and quarterly terms, with the annual discount already baked into the figure you see. That's a good deal if you're staying put and a bad deal if you're testing an idea for three weeks.

For a sense of what the alternative structure costs, look at how hourly billing is priced elsewhere in this market. LightNode lists $7.71/month and $0.012/hour for the same entry tier. At 730 hours a month, the hourly rate comes to roughly $8.76, about 12% more than the monthly price. Hourly billing earns its premium when you need a box for four days. Over twelve months it's a surcharge for flexibility you're not using.

If your project has a known lifespan measured in weeks, price the hourly option before committing to an annual term. If it's a service you intend to keep running, the annual discount on a LAX plan is the cheaper route, and the arithmetic isn't close.

## Containers are disposable. Your volumes are not.

Anything written inside a container is gone the moment that container is replaced or upgraded. Point a database at the container's filesystem and the first image update takes the data with it. Named volumes store the data on the host outside the container, so a Postgres directory or an uploads folder survives restarts and rebuilds.

The backup consequence is easy to miss: you back up the volume, not the container. A snapshot of a running container is worth roughly nothing if the actual data lives in a volume that wasn't included.

Before you order anywhere, ask whether panel-level backups or snapshots cover volumes or just the OS image. Docker's own documentation treats volume snapshots as the correct unit for this reason, and providers vary on what their snapshot tooling actually captures. If the answer is OS-only, budget for a `pg_dump` to object storage or a restic job against the volume mount, and put it in the compose file rather than in your memory.

👉 [Check what the Sixtynet plans include before you order](https://sixtynet.net/aff.php?aff=39)

## Who should not buy a Docker VPS at all

If you're hosting a static site, a brochure page or a single WordPress install, a shared or managed plan is cheaper and someone else handles updates. Docker solves a problem you don't have.

Shared hosting also cannot run Docker, which is worth knowing before you try. The Docker daemon needs root or sudo to manage containers, networks and kernel features like namespaces and cgroups, and shared accounts don't grant that by design. A VPS is the standard home for containers because it's the smallest thing that gives you root.

If you need a server for two days to test a migration, hourly billing on a provider that offers it beats signing an annual term anywhere.

## FAQ

**Is 2 GB of RAM enough for a Docker VPS?**
For one small app and a database, yes. Engine takes about 200 MB, leaving roughly 1.8 GB, which covers a lightweight app plus SQLite or a modest Postgres. Once you add a reverse proxy, Redis, a worker and a second app, move to 4 GB.

**Can I run several websites on one VPS?**
Yes. Each site gets its own container and its own compose service, and a reverse proxy routes by domain. Nginx Proxy Manager, Caddy and Traefik all handle routing and certificate issuance. The constraint is RAM, not architecture: each additional stack with a database adds a few hundred megabytes.

**Do I need a control panel?**
No. SSH and a compose file cover everything. A panel helps when you're running several projects and want a visual overview, and Portainer in a container does that job without touching the host.

**How do SSL certificates work with containers?**
A reverse proxy requests and renews them automatically from Let's Encrypt. The simplest validation method needs port 80 reachable from the internet, which is another reason a plan with 80 and 443 closed won't work for public web hosting. DNS-based validation is the workaround when ports are restricted.

**Does the Docker Desktop licence apply to a VPS?**
No. Docker Engine, the CLI and the container format are open source and free. Only Docker Desktop, the Mac and Windows GUI, carries a paid commercial licence, and that licence is free for personal use, education, non-commercial open source, and businesses under 250 employees and $10 million in annual revenue. On a Linux VPS you install Engine from the command line, so Desktop licensing rarely enters the picture. It only matters if your team develops on Macs and you're counting seats.

**Can I run Kubernetes or k3s on one of these?**
A 4-core plan with 4 GB is a realistic single-node starting point for k3s with a couple of light workloads. Two cores gets tight once the control plane takes its share, so treat LAX-STARTER as a Docker host and not a cluster.

## The decision, in one pass

One app plus a database, public-facing: **LAX-STARTER** at $200 a year. Several services plus a database: **LAX-MICRO** at $400 a year. Media transcoding or a write-heavy database: step up to **LAX-MEDIUM** or **LAX-LARGE** for the cores and the 25,000 IOPS. Skip the Residential and Irvine lines entirely unless your work needs a home-looking IP and no public ports, and skip the whole category if a shared plan would do.

Two things to confirm at checkout. First, whether backups or snapshots cover your volumes, because that determines how much of your weekend the disaster recovery plan consumes. Second, the renewal terms on your chosen billing cycle, since the annual figure you see is discounted and the page also sells quarterly terms with their own rate. On the Irvine line specifically, refunds are issued as account balance rather than money back, which is stated on the product page.

👉 [Compare the Sixtynet Docker-ready plans and pick your term](https://sixtynet.net/aff.php?aff=39)
