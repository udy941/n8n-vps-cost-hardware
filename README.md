# n8n hosting: real hardware requirements, the actual cost math, and how to pick a VPS that survives production webhooks

Self-hosting n8n is the cheapest way to run serious automation — the Community Edition is free, there are no per-execution fees, and you control your own data. The catch is that "n8n hosting" means different things depending on where you put it: your laptop, a PaaS template, or a proper VPS. This article walks through the hardware n8n actually needs, what each hosting route costs in practice, and why a DDoS-protected VPS like Sharktech's Smart VPS is a sensible home for a production instance — including the exact configuration numbers, billing discounts, and the setup mistakes that cost people their credentials.

## Quick pick

If you just want the answer: Sharktech's Smart VPS starts at **$7.95/month** for a 2 vCPU / 4 GB resource pool — enough for 20–50 active workflows — and the annual term applies a 50% discount, which works out to roughly **$3.98/month** (about $47.70 for the year). Every VM includes 40 GiB NVMe storage, 4TB of monthly transfer, one IPv4 address, and DDoS protection by default. 👉 [Check Smart VPS current pricing and deploy](https://bit.ly/SharKtech)

## Three ways to host n8n, and what they actually cost you

The hosting decision comes down to three routes, and they differ more in hidden costs than in sticker prices.

**Managed n8n Cloud.** Zero setup, but you pay per workflow execution volume and you don't touch the underlying machine. Fine for teams that never want to see a terminal.

**PaaS deploy templates.** Railway and Northflank both maintain n8n templates. Railway's queue-mode template deploys six services at once: Caddy for routing, the n8n editor, a dedicated webhook processor, workers, plus Redis and Postgres. It's genuinely production-shaped. The trade-off is architectural: Northflank's own guide points out that a default container uses ephemeral disk, so an SQLite-based n8n loses all data on restart unless you attach Postgres or a volume. You're renting convenience, and the metered compute adds up once workflows run around the clock.

**A VPS you control.** One machine, Docker, your database, your backups. Google Cloud Run users learned the hard way why this matters: in the official Cloud Run "easy mode" deployment, data lives in memory, and any scale-to-zero event or redeploy wipes it. On a VPS, the disk is yours and it doesn't disappear.

The VPS route is what this article focuses on, because it's the one where your choices actually change the outcome.

## How much hardware n8n really needs

Less than most VPS sellers would like you to believe — until the day it needs everything at once.

The commonly cited floor is **1 vCPU, 2 GB RAM, and 20 GB of SSD storage** to install and run n8n at all. Community forum users run personal automation and light AI tasks on old laptops with Docker capped at 4 GB of RAM. For standard business automation — 20 to 50 active workflows, concurrent webhook triggers from a CRM, basic database operations — the working recommendation is **2 vCPU, 4 GB RAM, and 50 GB of NVMe storage**.

Here's the part most hosting guides skip: n8n's workload is spiky by nature. It idles almost completely for most of the day, then a batch job or a burst of webhooks wants the entire machine simultaneously. That's why sizing for the average load underbuys, and why the pragmatic production recommendation for a single instance running alongside Postgres is **4 vCPU / 8 GB RAM / 80 GB storage**. You're buying headroom for the 3 a.m. cron run, not for the Tuesday afternoon average.

One more spec that matters more than RAM: storage type. n8n's execution data and the Postgres database behind it hit the disk constantly, so NVMe beats SATA SSD in a way you'll actually notice when workflows queue up.

## Sharktech plans for n8n hosting, side by side

Sharktech sells a few product lines; two of them fit n8n well: **Smart VPS** (a Proxmox-based resource pool where you carve up CPU, RAM, and storage across one or many VMs) and **Public Cloud** (an OpenStack-based cloud with fixed monthly tiers). The company also sells Dedicated Cloud — the same OpenStack platform with a fixed, pre-paid resource allocation instead of metered overage — and bare-metal servers, but those suit steady enterprise workloads rather than a single automation instance, so this comparison sticks to the two lines most n8n users will actually choose between. All prices below are USD, from Sharktech's own product pages and cost calculator.

| Plan | What you get | Base price | Billing terms | Order |
| --- | --- | --- | --- | --- |
| Smart VPS | Resource pool: 2 vCPU / 4 GB to start, scalable to 128 vCPU / 256 GB; 40 GiB NVMe + 4TB transfer + 1 IPv4 included per VM | $7.95/mo | Monthly, or −25% quarterly / −35% semi-annual / −50% annual | [ Deploy Smart VPS](https://bit.ly/SharKtech) |
| Public Cloud Small | Fixed base resource allocation + hourly metering above it | $39.00/mo | Monthly | [ Configure Small](https://bit.ly/SharKtech) |
| Public Cloud Medium | Larger fixed allocation, resource ceiling applies | $79.00/mo | Monthly | [ Configure Medium](https://bit.ly/SharKtech) |
| Public Cloud Large | Larger fixed allocation, resource ceiling applies | $249.00/mo | Monthly | [ Configure Large](https://bit.ly/SharKtech) |
| Public Cloud Enterprise | No preset resource ceiling | $499.00/mo | Monthly | [ Configure Enterprise](https://bit.ly/SharKtech) |

A note on the resource pool concept, because it's unusual: you're buying capacity, not a single machine. A 2 vCPU / 4 GB pool can run one n8n box, or split into a small production instance plus a throwaway test instance. Each VM picks its own location at deploy time — Los Angeles, Denver, Chicago, Las Vegas, or Amsterdam — so you can put the n8n instance near whichever APIs it hammers the most.

The Public Cloud tiers work differently, and the fine print matters. Small through Large carry resource ceilings — Sharktech explicitly notes these cap how much infrastructure you can consume, not how much money you can be billed — while Enterprise has no preset ceiling. Usage above each tier's included allocation is metered hourly across the whole cloud pool; the listed rates are $0.0025 per CPU core-hour and $0.0035 per GB of RAM per hour, with storage billed separately by type (NVMe, SSD, or HDD). For a single n8n instance that runs below its allocation, you'd simply pay the base fee.

One honest limitation: the exact included allocation per Public Cloud tier isn't published on the calculator page itself, so if you're choosing between tiers, use Sharktech's cloud cost calculator or check the plan page before ordering — don't guess from the base price alone.

## Why annual billing changes the math

The Smart VPS discount structure is where the real cost difference lives, and it's automatic — no coupon needed.

At $7.95/month, twelve months of monthly billing costs $95.40. Switch to annual and the 50% discount drops that to about **$47.70 for the year**, an effective $3.98/month. That's roughly $47 saved for exactly the same hardware. For a machine whose entire job is running an n8n instance 24/7, there's no scenario where monthly billing wins unless you genuinely expect to tear it down within a quarter.

Put that in context with what else is on the market: a comparable 2 vCPU / 4 GB / 64 GB managed VPS offer runs $9.58/month elsewhere, and that's a single fixed VM. The Smart VPS annual price gets you the same RAM class plus a resizable pool and built-in DDoS protection for less than half the monthly rate. 👉 [See the annual Smart VPS rate on Sharktech](https://bit.ly/SharKtech)

The counterweight, stated plainly: longer terms mean prepayment, and Sharktech's discounts come with a no-refund policy. If you're not sure you'll still be running n8n in eight months, start monthly and switch to annual once the instance has earned its keep.

## Why DDoS protection matters specifically for n8n

An n8n instance lives and dies by its webhook endpoints. Every trigger — payment notifications, form submissions, chatbot events — arrives as an inbound HTTP request to your server. If your host treats DDoS mitigation as a paid add-on, the machine hosting your business logic becomes the single most exposed box in your stack.

Sharktech built its network around DDoS mitigation from the start rather than bolting it on afterward; the protection is part of the default configuration on every plan, alongside 10–100G network connectivity. For comparison, AWS Shield Advanced starts at $3,000/month on its own. You don't need that scale, but the asymmetry is worth knowing: on Sharktech, mitigation is simply included, and the public calculator shows no surprise line items for it.

If your automations stay internal — cron jobs, database syncs, no public webhooks — this matters less, and you can weight price and location more heavily.

## Setting up n8n on a Sharktech VPS

Deployment takes minutes; the two configuration details below are what actually protect your data. The official supported path is Docker, via the one-line installer (`curl -fsSL https://get.n8n.io | sh`, Linux with Docker installed) or Docker Compose if you want Postgres alongside. n8n serves on port 5678, and you'll want it publicly reachable since that's how webhooks find you. Note that the npm install method is deprecated as of n8n 3.0 — Docker is the current recommendation.

1. Deploy a VM from your Smart VPS pool, picking the location nearest your webhook sources. Linux is the straightforward choice; Windows is supported if a specific workflow needs it.
2. Run the one-line installer or bring up Docker Compose. Give the container access to your storage — a data-heavy instance wants that NVMe.
3. Configure Postgres instead of defaulting to SQLite. SQLite on local disk works, but Postgres is what makes backups and future scaling (separate workers, queue mode) possible later.
4. Open port 5678 or put a reverse proxy in front, then claim the owner account immediately. The first visitor to an unclaimed n8n instance becomes its owner — you don't want that to be a stranger.

Two things people lose data over, both worth fixing on day one:

- **Back up your `N8N_ENCRYPTION_KEY`.** n8n encrypts stored credentials with it. Lose the key, and every credential in the database becomes permanently undecryptable — no recovery path exists.
- **Schedule nightly backups.** The official recommendation is a second container that copies all data nightly; the RAM cost is negligible and it won't affect your workflow capacity.

Sharktech's firewall rules and private networking are configurable from the management interface, so you can keep Postgres off the public internet entirely and expose only n8n itself.

## Who this setup is wrong for

Honesty section. Skip Sharktech if any of these describe you:

- **You want managed.** No one updates your Docker images or watches your Postgres for you. If "SSH into a server" sounds like a chore, a managed n8n platform or a template host fits better — you'll pay more per month for someone else to carry the pager.
- **You're testing n8n for a weekend.** Start with `npx n8n` on your own machine, or a monthly Smart VPS you can abandon. Don't prepay a year for an experiment.
- **You need absolute-minimum spend.** The $7.95 entry point is real, but bare-metal-cheap shared VPS offers exist lower. You're trading the DDoS protection, NVMe, and pool flexibility to save a couple of dollars.

The no-refund policy deserves one more line: it applies regardless of reason, so treat the first month at monthly billing as your evaluation period before committing to the annual term.

## FAQ

**Can I run n8n on a 2 GB RAM VPS?** It installs and runs, but 4 GB is the practical floor for anything with real workflows, and 8 GB if Postgres shares the machine. RAM is the first resource n8n exhausts during parallel executions.

**Do I have to use Docker?** For self-hosting, yes in practice — it's the officially recommended method, and the npm alternative is deprecated from n8n 3.0 onward.

**Will my webhooks work from behind a home connection?** Only through a tunnel, which requires Docker and adds a dependency you don't control. A VPS with a public IP removes that entire layer of fragility, which is the main reason n8n hosting on a VPS outgrows a local setup.

**Smart VPS or Public Cloud for n8n?** Smart VPS for a single n8n instance — cheaper, simpler, and the pool covers headroom for spikes. Public Cloud makes sense once you're running multiple VMs (n8n editor, workers, separate Postgres) and want one managed pool with hourly metering, or expect unpredictable bursts that would force a VPS upgrade.

**What happens when I outgrow the pool?** You upgrade the Smart VPS resource pool through the billing platform and reallocate the added capacity to whichever VM needs it — no migration, no new IP, no rebuilding credentials.

## Before you click order

Match the plan to the workload and the decision makes itself: personal automations and a handful of workflows → Smart VPS at monthly billing, $7.95. Production instance with Postgres, webhook volume, and AI agent nodes → the same Smart VPS pool sized to 4 vCPU / 8 GB, on annual billing for the 50% discount. Multi-VM setups with database and production separated → start with Public Cloud Small at $39 and add VMs as needed, checking the tier's included allocation on the plan page first.

One last check before paying: confirm the billing cycle dropdown reflects the term you actually want, because the 25/35/50% discounts apply at prepayment and the no-refund policy means there's no undo. If the numbers above look right for your workload, 👉 [deploy your n8n VPS on Sharktech](https://bit.ly/SharKtech) and have the Docker installer running before the minute's out.
