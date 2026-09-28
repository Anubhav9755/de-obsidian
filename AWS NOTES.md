### Lesson 1: What Is Cloud Computing (and why does it exist)

#### The analogy first (unconventional bit)

Before electricity grids existed, if a factory wanted power, it had to **build its own power plant**. Buy the generator, hire someone to maintain it, size it for your _peak_ need (even though you'd use that peak maybe 2 days a year), and eat the cost every single day whether you used it or not.

Then the electric grid showed up. Now the factory just **plugs in and pays for what it uses**. No generator to own. No maintenance staff. If it needs more power next month, it just uses more — the grid scales invisibly behind the wall socket.

Cloud computing did to _computers_ what the grid did to _electricity_.

#### Definition
Cloud computing is the on-demand delivery of computing resources (servers, storage, databases, networking) over the internet, with pay-as-you-go pricing — instead of you owning and running the physical hardware yourself.

The four properties that actually matter (memorize these as a set, not as isolated words):

- **On-demand** — you get resources in minutes, not the weeks it takes to order and rack a physical server
- **Elastic** — capacity grows and shrinks automatically with real demand
- **Pay-as-you-go** — you're billed for usage (like an electricity meter), not a fixed upfront purchase
- **Someone else manages the physical layer** — AWS worries about the data center, the cooling, the hardware failures; you don't

#### The industry problem this solves

Picture **Hotstar (or any streaming platform) during an India–Pakistan cricket match**. Traffic might be 10x normal for 3 hours, then crash back to baseline.

In the old world (own your data center): you'd have to buy enough servers to survive that 3-hour spike — and those servers sit **90% idle for the other 362 days of the year**, quietly costing money and depreciating.

In the cloud world: you spin up extra capacity for those 3 hours, and scale it back down right after. You pay for the spike, not for owning spike-capacity year-round.

This single idea — **CapEx → OpEx** (Capital Expenditure → Operating Expenditure) — is why cloud computing exists commercially, not just technically. You stop buying hardware (a capital expense, like buying a building) and start renting exactly what you use (an operating expense, like a electricity bill). This is the single most-tested one-liner across every cloud fundamentals interview, so lock it in.

#### Interview questions (be able to answer out loud, not just recognize)

1. _"Explain cloud computing to someone with no technical background."_ → Use the electricity grid analogy. Interviewers specifically test whether you can de-jargon a concept — this is a communication test disguised as a technical one.

2. _"What's the actual difference between cloud computing and just renting a server from a hosting company?"_ → The key word is **elastic**. A rented server from a traditional host is usually a fixed size for a fixed monthly fee — you can't have it 10x itself for 3 hours and shrink back automatically. Cloud can.

3. _"Why would a company choose cloud over building its own data center, cost aside?"_ → Speed (minutes vs months to provision), and offloading operational burden (AWS handles hardware failure, cooling, physical security) so the company's engineers focus on the product, not the electricity bill.

#### Scenario question

Q. "A college friend is building a startup app. They don't know if they'll get 10 users or 10,000 users in month one. Their uncle offers to buy them a physical server for their startup. What do you tell them, and why?"

Ans. You don't know if you'll get 10 users or 10,000. If you buy hardware sized for 10, it'll crash the day you go viral. If you buy hardware sized for 10,000 'just in case,' you're paying for capacity you might never use, and if the startup fails in month two, that server is a sunk cost sitting in a room somewhere. On the cloud, you launch on the smallest instance, and if you _do_ blow up, you scale up in minutes — and if you don't, you've paid almost nothing. The uncle's generosity is actually the riskier bet here, not the cheaper one.

---
### Cloud Deployment Models — Public, Private, Hybrid

This is about **who owns and who shares the infrastructure you're running on**:

- **Public cloud** — infrastructure owned by a third party (AWS, Azure, GCP) and shared across many customers over the internet. You rent a slice of it.
- **Private cloud** — infrastructure dedicated to a single organization, either on their own premises or hosted exclusively for them. Nobody else's workloads run on that hardware.
- **Hybrid cloud** — a deliberate mix: some workloads stay private, some run public, and they're connected to work together.

#### The industry problem this solves

Think about a **bank**. Banks have two very different kinds of workload:

1. Core customer transaction data, subject to strict regulation, audit trails, and sometimes a legal requirement that it never leaves a specific data center → this often stays **private** (or in a "sovereign" region)
2. Their public-facing mobile app backend, marketing website, fraud-detection ML models, load-testing infrastructure → these benefit hugely from public cloud's elasticity and speed, and there's no regulatory reason to keep them private

So the bank runs **hybrid**: private cloud (or on-prem) for the regulated core, public cloud (AWS) for everything that benefits from elasticity, connected via a secure link between the two. This isn't a compromise — it's usually the _correct_ architecture, not a stepping stone toward "someday we'll go fully public."

This is also why AWS sells **AWS Outposts** (AWS hardware, physically installed in _your_ data center, but managed through the same AWS console) — it exists specifically to serve hybrid customers who need AWS's tooling but data that never leaves their building.

#### Interview questions

1. _"Why wouldn't a company just go 100% public cloud if it's cheaper and faster?"_ → Regulatory/compliance requirements (data residency laws, finance/healthcare regulation), existing sunk investment in on-prem hardware they haven't depreciated yet, or ultra-low-latency needs that colocated private hardware handles better than a shared public network.

2. _"What's the difference between private cloud and just 'on-premises servers'?"_ → On-prem is literally in your building with traditional management. Private _cloud_ still gives you cloud-like self-service and elasticity (virtualization, an internal portal to spin up VMs) — it's "cloud-style operations," just not multi-tenant or off-premises. This distinction trips people up — private cloud isn't just "old-school servers," it borrows cloud's operating model.

3. _"Where does hybrid cloud usually go wrong in practice?"_ → The network link between the private and public side becomes the bottleneck or single point of failure, and teams underestimate the complexity of keeping data consistent and secure across both environments.

#### Scenario question

Que. "A hospital chain wants to use AWS for their patient appointment-scheduling app (public-facing, needs to scale during flu season) but has a legal requirement that patient medical records never leave their own data center. Design the deployment model and explain your reasoning."

Ans. **Deployment model: Hybrid.**
Reasoning, step by step:

1. **Split the workload by what it actually is, not by convenience.** The appointment-scheduling app is public-facing and needs elasticity (flu season spikes, people booking from phones at all hours) — that's a textbook case _for_ public cloud: unpredictable demand, no regulatory constraint on where the scheduling logic itself runs.

2. **Isolate the constraint.** The hard requirement is specifically "patient medical records never leave our data center" — that's a data-residency constraint, not an app-architecture constraint. It doesn't say the _scheduling app_ can't be public; it says the _medical records_ can't leave. Those are two different things, and conflating them is the mistake people make — they either force everything private (killing the elasticity benefit) or put everything public (violating the legal requirement).

3. **Architect around the boundary.** Run the scheduling app itself on AWS (EC2/Lambda behind a load balancer, auto-scaling for flu season). Keep the actual patient medical records database on-prem/private, inside the hospital's own data center. Connect the two with a secure private link (AWS Direct Connect or a VPN) so the public app can query patient data _without that data ever being stored or persisted on AWS_ — request comes in, hits the on-prem database, response goes back out, nothing sensitive lands on AWS disks.

4. **Why not just keep everything private to be safe?** Because then you lose the entire reason you wanted AWS — the app can't auto-scale for flu season without manual capacity planning again, which is exactly the problem cloud computing exists to solve (Lesson 1).

5. **Why not just encrypt everything and put it all public?** Because the requirement as stated isn't "encrypt it," it's "never leaves the data center" — a legal/compliance line, not a technical risk to mitigate with encryption. You don't get to reinterpret a stated compliance requirement to fit a cleaner architecture; you architect around the requirement as given.

---

### Cloud Service Models — IaaS vs PaaS vs SaaS

Think about **getting pizza for dinner**, at three levels of "how much do I do myself"- 
**Make it from scratch**, **Buy a pizza kit** — dough and sauce pre-made, you just assemble and bake it, **Order delivery** — pizza just shows up. You do nothing but eat.

#### Definition

- **IaaS (Infrastructure as a Service)** — AWS gives you the raw building blocks (a virtual machine, storage, network) and **you** install the OS updates, the runtime, the app, everything above the hardware layer. _Example: EC2._

- **PaaS (Platform as a Service)** — AWS manages the OS, the runtime, the patching — you just hand over your application code and it runs. _Example: Elastic Beanstalk, AWS Lambda (for compute specifically)._

- **SaaS (Software as a Service)** — the entire application is already built and managed by someone else; you just use it through a browser or API. _Example: Gmail, or from AWS's own portfolio, Amazon Chime._

The one-line test interviewers actually use: **"How much of the stack do you manage vs how much does the provider manage?"** Draw the stack (hardware → OS → runtime → app → data) and ask where the line falls.

|Layer|IaaS (EC2)|PaaS (Elastic Beanstalk)|SaaS (Gmail)|
|---|---|---|---|
|Data & app|You|You|Provider|
|Runtime & middleware|You|Provider|Provider|
|OS|You|Provider|Provider|
|Virtualization/hardware|Provider|Provider|Provider|
#### The industry problem this solves

A **small startup engineering team** doesn't want to spend engineer-hours patching operating systems and configuring web servers — that's not what makes their product special. So they deploy on **Elastic Beanstalk (PaaS)**: they push code, AWS handles provisioning the EC2 instances underneath, load balancing, OS patches, scaling. The team's time goes entirely into product code, not infrastructure babysitting.

Meanwhile, a company doing something highly custom — say, a specific GPU driver configuration for an ML training pipeline — needs **EC2 (IaaS)**, because PaaS's convenience comes at the cost of control, and they need the control.

#### Code / hands-on

**IaaS way (EC2) — you do the setup:**

```bash
# You SSH into a raw EC2 instance and do everything yourself
sudo yum update -y                     # YOU patch the OS
sudo yum install -y python3            # YOU install the runtime
pip3 install flask                     # YOU install dependencies
python3 app.py                         # YOU run and manage the process
```

**PaaS way (Elastic Beanstalk) — you just hand over code:**

```bash
eb init -p python-3.9 my-flask-app     # tell AWS "this is a Python app"
eb create my-flask-env                 # AWS provisions EC2, load balancer, OS, everything
eb deploy                              # you just push your code from here on
```

Same end result (a running Flask app), but notice: in the IaaS version, _you_ typed every infrastructure command. In the PaaS version, AWS did all of that behind `eb create` — you only ever touch your own app code again. That gap is the entire IaaS/PaaS distinction, made visible in two terminal sessions.

#### Interview questions

1. _"Classify these AWS services: EC2, Lambda, RDS, S3."_ → EC2 = IaaS (you manage the OS). Lambda = PaaS-ish, often called "serverless" as its own category, but functionally you only hand over code, AWS handles everything else. RDS = PaaS for databases specifically (AWS patches and manages the DB engine, you just use it). S3 — trickier: often classified as its own thing ("storage as a service") but if forced into the three-tier model, it's closest to SaaS-like from a management perspective (you never touch an OS or server at all). Be ready to justify your reasoning, not just recite the label.

2. _"Why would a company deliberately choose IaaS when PaaS is less work?"_ → Control. Custom OS-level configuration, licensing requirements that need a specific OS version, or compliance rules that require the company itself (not AWS) to manage patching for audit reasons.

3. _"Is Lambda IaaS, PaaS, or something else?"_ → Most correctly it's "serverless" / FaaS (Function as a Service) — a step beyond PaaS where you don't even manage a running server process, just the function code itself, and it's billed per invocation, not per running instance.

#### Scenario question

_"A team of 3 engineers is building an MVP in 6 weeks and none of them have deep Linux/sysadmin experience. They need a web app running fast. IaaS, PaaS, or SaaS building block — and why?"_

It's not "PaaS is easier" in the abstract, it's specifically that a 3-person team with no sysadmin depth would burn their scarce 6 weeks patching OSes instead of shipping product. That's the justification an interviewer wants, not just the label.

---
#### The Shared Responsibility Model

**The Shared Responsibility Model** splits security duties into two halves:

- **AWS is responsible for "security OF the cloud"** — the physical data centers, the hardware, the network infrastructure, the hypervisor, and (for managed services) the underlying software. This is fixed and non-negotiable — you can't opt into managing AWS's data center security even if you wanted to.

- **You are responsible for "security IN the cloud"** — your data, your IAM permissions, your OS patching (on IaaS), your firewall/security group configuration, your encryption choices, who has access to what.

**Where the line falls shifts depending on the service type** - 

- **EC2 (IaaS)** — AWS secures the physical hardware and hypervisor; **you** patch the guest OS, configure the firewall (security groups), manage your app's own security
- **RDS (PaaS-ish, managed DB)** — AWS patches the database engine and underlying OS **for you**; you're still responsible for who has network/IAM access to it, and your own data
- **S3** — AWS handles everything about the storage infrastructure; you are 100% responsible for whether your bucket is public or private, because that's a setting _you_ control

#### The industry problem this solves

This isn't abstract — it's the difference between "AWS's fault" and "your fault" in real breaches. **The overwhelming majority of AWS-related data breaches you read about in the news (misconfigured S3 buckets left public, leaked API keys, over-permissioned IAM roles) are not AWS failures — they're customer-side failures**, because the customer owned "security IN the cloud" and got it wrong. AWS's data centers being physically breached is vanishingly rare; a developer accidentally making an S3 bucket public, or committing an AWS access key to a public GitHub repo, happens constantly.

#### Code / hands-on

Here's the model made concrete — this is a security group (a firewall rule you're responsible for, sitting on top of hardware AWS already secured):

```bash
# You are responsible for this — AWS didn't set this for you
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0          # <-- THIS is the dangerous line
```

That `0.0.0.0/0` on port 22 (SSH) means "anyone on the entire internet can attempt to SSH into this server." AWS will not stop you from doing this — it's not AWS's job to. It gave you a secure, isolated instance; whether you then open the front door to the whole internet is entirely on your side of the line. 

The fix — restricting the CIDR to your own IP — is a two-second edit, but it's a security decision only _you_ can make.

```bash
# Fixed version — only your own IP can SSH in
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.25/32
```

#### Interview Question 

- _"If a company's S3 bucket is left public and customer data leaks, whose fault is it — AWS's or the company's?"_ → The company's. AWS provided a private-by-default, secure storage service; the company (or a developer) actively changed a setting to make it public. 

- _"Does the responsibility split change between EC2 and RDS? How?"_ → Yes — on EC2 you patch the OS yourself; on RDS, AWS patches the OS and DB engine for you. Moving up the IaaS→PaaS spectrum  shifts the responsibility line further onto AWS's side.

---
#### IAM Fundamentals — Users, Groups, Roles, and Policies

#### Definition

- **IAM User** — a persistent identity (person or application) with long-term credentials (password, access keys)
- **IAM Group** — a collection of users that share the same permissions; you attach policies to the group, not to each user individually
- **IAM Role** — an identity **without** long-term credentials that anything (a person, an AWS service, an application) can **assume temporarily** to get a set of permissions for a limited time
- **IAM Policy** — a JSON document that defines _what actions are allowed or denied on what resources_

#### The industry problem this solves

Say your EC2 instance needs to read files from an S3 bucket. The naive (and dangerous) approach: create an IAM User, generate an access key + secret key, and hardcode them into your app's config file on the EC2 instance. Now if that instance is ever compromised, or if that code is accidentally pushed to a public GitHub repo (this happens constantly — it's the single most common real-world AWS security incident), the attacker has **permanent, standing credentials** to your AWS account.

The correct approach: create an **IAM Role** with exactly the S3-read permission needed, and **attach that role to the EC2 instance itself**. AWS automatically rotates temporary credentials to the instance in the background — no access keys exist anywhere in your code, ever. If the instance is compromised, the attacker gets credentials that expire in hours, not a permanent key that works forever until someone notices and revokes it.

**"never hardcode credentials — use IAM roles."**

##### Code / hands-on - An IAM Policy is just JSON.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-data-pipeline-bucket",
        "arn:aws:s3:::my-data-pipeline-bucket/*"
      ]
    }
  ]
}
```

Reading in plain English: _"Allow the action of reading objects and listing contents, but only on this one specific bucket (and everything inside it) — nothing else."_ Notice what's **not** here: no `s3:DeleteObject`, no `s3:PutObject`, no permission on any other bucket. This is the **principle of least privilege** — grant exactly what's needed, nothing more — made literal in JSON. Every AWS security question, at some level, is really asking "did you apply least privilege here."

#### Interview Question 

_"Can you attach a policy directly to a user instead of a group?"_ → Yes, technically, but it doesn't scale — best practice is always group-level policies (or roles), because managing permissions per-individual-user becomes unmanageable as a team grows.

#### Scenario question

_"You're building a Lambda function (as part of a data pipeline) that needs to read from an S3 bucket and write results into a DynamoDB table. Walk through exactly how you'd set up its permissions — what IAM construct do you use, and what would the policy look like conceptually?"_

Ans- Lambda functions get an **execution role**, and that role's policy would grant `s3:GetObject` on the source bucket and `dynamodb:PutItem` on the target table.

---

#### EC2 — Elastic Compute Cloud

#### Definition

**EC2 (Elastic Compute Cloud)** gives you resizable virtual servers ("instances") in the cloud. You pick:

- **AMI (Amazon Machine Image)** — the "OS + pre-installed software" template your instance boots from.
- **Instance type** — the hardware profile: how much CPU, RAM, network performance (e.g. `t3.micro` = tiny/cheap/burstable, `c5.xlarge` = compute-heavy, `r5.large` = memory-heavy)
- **Security Group** — the instance-level firewall (you configured this back in Lesson 4)
- **Key pair** — the SSH credential used to log into the instance

Instance type naming isn't random — it's actually a small readable code:  
`t3.micro` → **t** (family: burstable general purpose) **3** (generation) **.micro** (size)  
`c5.xlarge` → **c** (family: compute-optimized) **5** (generation) **.xlarge** (size)

Once you can read `m5.2xlarge` and know "general purpose, 5th gen, extra-large" without looking it up, you've internalized this — that's the actual skill, not memorizing every instance type.

### EC2 Pricing Models

Think of it like booking a hotel room, four different ways:

| Model                                  | Analogy                                                                          | How it works                                                                                                                                                       | Best for                                                                                              |
| -------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| **On-Demand**                          | Walk-in hotel rate                                                               | Pay per second, no commitment, most expensive per-hour                                                                                                             | Unpredictable, short-term, or spiky workloads                                                         |
| **Reserved Instances / Savings Plans** | Booking a year-long lease                                                        | Commit to 1 or 3 years upfront (or partial upfront), get up to ~40-70% discount vs On-Demand                                                                       | Steady-state, always-on workloads you know you'll run long-term (e.g. a production database server)   |
| **Spot Instances**                     | Bidding on unsold hotel rooms the hotel would rather sell cheap than leave empty | You bid on AWS's _spare_, unused capacity at up to 90% discount — but AWS can reclaim it with ~2 minutes' notice if someone else needs that capacity at full price | Flexible, interruption-tolerant, stateless work — batch jobs, big data processing, CI/CD test runners |
| **Dedicated Hosts**                    | Renting the entire building, not just a room                                     | A physical server dedicated entirely to you (no other AWS customer's instances share that hardware)                                                                | Strict compliance/licensing requirements (e.g. software licensed "per physical core")                 |

The interview-critical intuition: **AWS is selling you a discount in exchange for you taking on risk or commitment.** Reserved = you take on a _time commitment_ risk (locked in for 1-3 years) in exchange for a discount. Spot = you take on an _interruption_ risk (can be kicked off anytime) in exchange for a much steeper discount. On-Demand = zero risk, zero discount.

#### The industry problem this solves

Before EC2, a startup wanting a server had to either buy hardware (weeks of lead time, big upfront cost) or sign a fixed hosting contract (pay for a fixed-size box whether you use 10% or 100% of it). EC2 solves **both** problems: you get a running server in under 2 minutes (you saw this in Lesson 1), and you can **resize it** — literally change the instance type — as your needs change, or run 100 of them for an hour during a traffic spike and terminate them right after, paying only for that hour.

This is also _why_ EC2 is the backbone service most other AWS services are secretly built on top of — Elastic Beanstalk (Lesson 3) is EC2 with automation wrapped around it; even some managed services run on EC2 fleets behind the scenes that AWS manages for you.

#### Code / hands-on

Launching an EC2 instance via CLI

```
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \     # which AMI (Amazon Linux 2023, in this case)
  --instance-type t3.micro \             # which hardware size (free-tier eligible)
  --key-name my-ssh-key \                # which key pair to allow SSH login
  --security-group-ids sg-0123456789abcdef0 \  # which firewall rules apply
  --subnet-id subnet-0abc123def456789 \  # which network it lives in
  --count 1
```

Then checking it's actually running:

```bash
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"
```

And — the part people forget until their bill teaches them — **terminating it when done**:

```bash
aws ec2 terminate-instances --instance-ids i-0abcd1234efgh5678
```

That last command matters more than it looks. EC2 bills by the second while an instance is _running_ — a forgotten `t3.micro` left running for a month is trivial money, but a forgotten `p3.2xlarge` (GPU instance) left running is a real, painful bill. This is the #1 accidentally rack up surprise AWS charges — leaving something running after the exercise, not the exercise itself.

#### Interview questions

1. _"What determines the cost of an EC2 instance?"_ → Instance type (CPU/RAM/network profile), how long it runs (billed per second), the region, and whether you're using On-Demand, Reserved, or Spot pricing.

2. _"What's the difference between stopping and terminating an instance?"_ → Stopping keeps the instance (and its attached EBS storage) intact, just powered off — you can start it again later, same data. Terminating destroys the instance permanently (and by default its root volume too, unless configured otherwise).

3. _"How would you reduce EC2 costs for a predictable, always-on workload vs a flexible batch job?"_ → Reserved/Savings Plans for predictable steady-state workloads (commit upfront, get a discount); Spot Instances for flexible, interruption-tolerant batch jobs (bid on spare AWS capacity at a steep discount, but AWS can reclaim it with short notice).

#### Scenario question

_"You're running a data processing batch job that takes 2 hours, runs once a night, and can tolerate being interrupted and restarted if needed. What EC2 pricing model would you use, and why not just use a normal On-Demand instance running 24/7?"_

The 2-hour nightly batch job → **Spot Instances**. It's flexible (runs whenever), interruption-tolerant (if it dies mid-run, you just restart it — no user is sitting there waiting on it live), and predictable in duration, so the steep Spot discount is nearly free money here. Running it On-Demand 24/7 would mean paying for 22 idle hours a day for a job that only needs 2 — pure waste, and it doesn't even need the "always-on" guarantee that would justify Reserved pricing either, since it only ever runs for a 2-hour window.

---

#### S3 — Simple Storage Service

S3 is a **massive, infinitely expandable filing cabinet in the cloud** — except instead of folders and files in the traditional sense, it's **buckets** (the cabinet) and **objects** (the files inside), and you never have to tell it "buy more cabinet space" — it just expands as you add files, with no capacity planning at all.

#### Definition

**S3 (Simple Storage Service)** is object storage — you store and retrieve files ("objects") inside containers ("buckets"), accessed over HTTP(S), with virtually unlimited capacity and **99.999999999% durability** (11 nines — meaning if you store 10,000,000 objects, you'd statistically expect to lose one roughly once every 10,000 years, because AWS automatically replicates every object across multiple physical facilities).

- **Bucket** — a globally-unique-named container (e.g. `my-data-pipeline-bucket`)
- **Object** — the actual file, plus metadata, identified by a **key** (its full path-like name, e.g. `raw-data/2026/09/logs.csv`)
- **Storage Class** — different price/retrieval-speed tiers for the same durability

#### The industry problem this solves

**1. Unpredictable scale.** A data engineering pipeline ingesting raw logs doesn't know today how many terabytes it'll be storing in a year. With a traditional file server, you'd need to plan disk capacity in advance. S3 needs zero planning — write the file, it's stored, forever, at whatever scale.

**2. Cost of "must keep it, but rarely touch it" data.** This is where **Storage Classes** earn their keep — this is a real, money-saving decision companies make constantly:

|Storage Class|Retrieval|Use case|
|---|---|---|
|**S3 Standard**|Instant|Frequently accessed data|
|**S3 Standard-IA (Infrequent Access)**|Instant, but lower storage cost + retrieval fee|Backups, data accessed a few times a month|
|**S3 Glacier Instant Retrieval**|Instant, much cheaper storage|Archival data that must still be queryable|
|**S3 Glacier Deep Archive**|12+ hours|Long-term compliance archives, almost never touched (cheapest tier)|
A **lifecycle policy** automates moving objects between these tiers based on age — e.g. "after 30 days, move to Standard-IA; after 90 days, move to Glacier." This is exactly what I flagged in your checklist for the "S3 costs are high, data older than 90 days must stay queryable"

#### Code / hands-on

```
# Create a bucket (bucket names must be globally unique across ALL of AWS)
aws s3 mb s3://anubhav-data-pipeline-raw

# Upload a file — this is your object, its key becomes "logs/day1.csv"
aws s3 cp logs.csv s3://anubhav-data-pipeline-raw/logs/day1.csv

# List what's in the bucket
aws s3 ls s3://anubhav-data-pipeline-raw/logs/

# Download it back
aws s3 cp s3://anubhav-data-pipeline-raw/logs/day1.csv ./downloaded.csv
```

A lifecycle rule as JSON (the automation that saves real money):


```json
{
  "Rules": [
    {
      "ID": "MoveOldLogsToGlacier",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 90, "StorageClass": "GLACIER" }
      ]
    }
  ]
}
```

Meaning  - _"Anything under the `logs/` prefix that's older than 90 days, automatically move to Glacier."_ Set once, saves money forever, no human has to remember to do it.

#### Interview questions

1. _"What's the difference between S3 and EBS (Elastic Block Store)?"_ → S3 is object storage, accessed over HTTP(S), not attached to a specific instance — great for files, backups, static assets. EBS is block storage, attached to a single EC2 instance like a virtual hard drive — used for an instance's OS or app data that needs low-latency, filesystem-level access.

2. _"How would you architect storage for data that's frequently accessed for 30 days, then rarely touched after?"_ → S3 Standard for the first 30 days, then a lifecycle policy transitioning it to Standard-IA or Glacier after that — no manual intervention needed.

3. _"Is S3 data automatically backed up / replicated?"_ → Yes — S3 Standard automatically replicates objects across multiple Availability Zones within a region by design (that's where the 11 nines durability comes from), distinct from you manually enabling cross-region replication for disaster recovery.

#### Scenario question

_"Your data pipeline generates 50GB of raw log files daily. They're queried heavily for the first week (for near-real-time dashboards), rarely queried for the next 3 months (occasional audits), and by law must be retained for 7 years but essentially never accessed after month 4. Design the S3 storage class strategy across that lifecycle."_

| Time window                    | Storage class               | Why                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------ | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Day 0–7                        | **S3 Standard**             | Queried heavily for near-real-time dashboards — needs instant retrieval, frequent access doesn't incur extra fees here                                                                                                                                                                                                                        |
| Day 8–90                       | **S3 Standard-IA**          | Rarely queried (occasional audits) but still needs instant access when it _is_ queried — IA is cheaper storage than Standard, with a small per-retrieval fee, which is fine since access is now rare                                                                                                                                          |
| Day 91 onward (through year 7) | **S3 Glacier Deep Archive** | Essentially never accessed after month 4, but legally must be _retained_ (not necessarily _queryable on demand_) for 7 years — Deep Archive is the cheapest possible storage, with a 12+ hour retrieval time, which is acceptable because nothing here suggests you need to pull it back fast, only that you must be _able_ to for compliance |
| Day 2,555 (7 years)            | **Deleted**                 | Add an expiration rule so it doesn't sit there costing money forever once the legal retention window closes                                                                                                                                                                                                                                   |
This is one lifecycle policy with three transitions and one expiration, not three separate rules added "from time to time" — you set it once, on day 0, and AWS executes all four stages automatically as each object ages:

```
{
  "Rules": [
    {
      "ID": "LogLifecycle",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [
        { "Days": 7,  "StorageClass": "STANDARD_IA" },
        { "Days": 90, "StorageClass": "DEEP_ARCHIVE" }
      ],
      "Expiration": { "Days": 2555 }
    }
  ]
}
```
---

#### AWS Global Infrastructure — Regions, Availability Zones, Edge Locations

AWS's infrastructure is like **a country with multiple states, and each state has multiple independent power grids, and every neighborhood has a small local substation.**

#### Definition

- **Region** — a physical geographic location containing multiple, isolated data centers (e.g. `ap-south-1` = Mumbai, `us-east-1` = North Virginia). Regions are fully independent — data doesn't automatically move between regions unless you configure it to.

- **Availability Zone (AZ)** — one or more discrete data centers within a Region, each with independent power, cooling, and networking, but connected to other AZs in the same region via high-speed, low-latency links. Every Region has **at least 2-3 AZs** (this is the actual mechanism behind "fault tolerance" — a term I've been using loosely until now).

- **Edge Location** — a smaller AWS site, much more numerous than Regions, used by **CloudFront (AWS's CDN)** and **Route 53** to cache content and serve requests physically closer to end users, cutting latency.

**The critical relationship to internalize:** Region > AZ > Edge Location, in terms of size, but the _purpose_ differs completely — Regions/AZs are about **where your infrastructure lives and how resilient it is**; Edge Locations are about **how fast your content reaches a user**, and don't run your actual servers/databases.

#### The industry problem this solves

**Two separate real problems:**

**1. Fault tolerance (why AZs exist).** Imagine you run your entire production database on a single EC2 instance in a single data center. A fire, a flood, or even a routine hardware failure in that one building takes your entire product offline. This actually happened to companies before multi-AZ architecture was standard practice. The fix: **deploy across multiple AZs** — e.g., a primary database in AZ-1, a replica in AZ-2. If AZ-1's entire data center goes dark, AZ-2 keeps serving traffic, because they're physically separate enough (different flood zones, different power grids, but still fast-connected) that one failure doesn't cascade to the other.

**2. Latency (why Edge Locations exist).** If your app's servers are in Mumbai but a user is in the US, every request travels halfway around the world — slow, no matter how fast your server itself is. CloudFront caches your static content (images, videos, JS files) at Edge Locations physically near that US user, so the request never has to travel to Mumbai at all for repeat content.

**3. Data residency (why Region choice matters).** A company handling Indian users' data might be legally required to keep it within India — so they deliberately pick `ap-south-1` and never replicate that data to a US region. This connects directly back to Lesson 2's hybrid cloud discussion — Region selection is often a compliance decision, not just a performance one.

#### Code / hands-on

See this for real — every resource you create lives in a specific Region and (for many services) a specific AZ:

```bash
# See every AWS Region currently available to you
aws ec2 describe-regions --output table

# See every AZ within your currently-configured region
aws ec2 describe-availability-zones --output table
```

When you ran your EC2 instance back in Lesson 6, that `--subnet-id` flag was secretly pinning it to one specific AZ. Launching a second instance in a _different_ subnet (mapped to a different AZ) is the actual, literal action behind "deploying for fault tolerance" — it's not a separate exotic feature, it's just "pick a different AZ when you launch."

#### Interview questions

1. _"What's the difference between a Region and an Availability Zone?"_ → Region = a geographic area containing multiple AZs; AZ = an isolated data center (or cluster of them) within that region, with independent power/cooling/networking but fast-connected to sibling AZs. This is one of the single most commonly asked AWS fundamentals questions — nearly guaranteed in some form.

2. _"Why does AWS recommend deploying across multiple AZs instead of just using one?"_ → Fault tolerance — a failure in one AZ (power, cooling, hardware) doesn't take down the others, so your application stays available even if one entire data center goes offline.

3. _"What's the purpose of an Edge Location, and how is it different from a Region?"_ → Edge Locations exist purely to cache content close to end users for lower latency (via CloudFront/Route 53) — they don't host your actual compute/database infrastructure the way Regions and AZs do.

4. _"If a company must keep customer data within a specific country for legal reasons, how does AWS's infrastructure model support that?"_ → By deliberately choosing and pinning all resources to a Region located in that country, and never enabling cross-region replication for that data.

#### Scenario question

_"You're building a data pipeline's database layer. The company says 'we need this to survive a single data-center outage without any downtime.' What AWS Global Infrastructure concept directly addresses this requirement, and how would you architect it?"_

Deploy the database with a **primary instance in one AZ and a synchronously-replicated standby in a second AZ** (this is literally what RDS Multi-AZ does out of the box) — if the primary's AZ goes dark, AWS automatically fails over to the standby, and because the two live in physically separate data centers, one outage can't take both down. That's the "how," not just the "what."

---

##### AWS Well-Architected Framework — The Six Pillars

Think of building a house. You wouldn't just ask "does it have walls and a roof?" A good architect checks it against a mental checklist: _Is it safe? Will it survive a storm? Is it energy-efficient? Can we afford to run it? Can we expand it later? Will it hold up environmentally?_

The **Six Pillars**, with the one-line test for each:

|Pillar|The question it asks|
|---|---|
|**Operational Excellence**|Can we run and monitor this system, and improve it, without heroics?|
|**Security**|Is data and the system protected, and access properly controlled?|
|**Reliability**|Does it recover automatically from failure and meet demand?|
|**Performance Efficiency**|Are we using the _right-sized_ resources for the job, and adapting as needs change?|
|**Cost Optimization**|Are we avoiding unnecessary spend without sacrificing what we actually need?|
|**Sustainability**|Are we minimizing environmental impact (energy/resource use) of running this?|

This isn't a random list — notice it's basically **every lesson we've done so far, given formal names**: Lesson 4 (Shared Responsibility) and Lesson 5 (IAM) _are_ the Security pillar. Lesson 8 (Multi-AZ) _is_ the Reliability pillar. Lesson 7's storage-class lifecycle policy _is_ the Cost Optimization pillar. The Well-Architected Framework isn't new content — it's the **organizing skeleton** that everything else hangs on.

#### The industry problem this solves

Here's the real failure mode this framework exists to prevent: an engineer builds something that _works_ — it's fast, it's live, it demos great — but nobody checked whether it survives an AZ outage (Reliability), whether an intern with S3 access could accidentally leak customer data (Security), or whether it's quietly burning ₹50,000/month because nobody set a lifecycle policy (Cost Optimization). It "works" right up until the one dimension nobody checked becomes an incident.

**Real-world use:** AWS Solutions Architects run actual **Well-Architected Reviews** with enterprise customers — a structured session going through each pillar's specific questions against a customer's real architecture, flagging risks before they become 3 AM incidents. This isn't exam trivia — it's a real, billable AWS professional service, and "run a Well-Architected review" is a genuine sentence used in real jobs.

#### Interview questions

1. _"Name the six pillars of the Well-Architected Framework."_ → Straight recall — but always follow it with a one-line explanation of each, don't just list words, interviewers notice the difference immediately.

2. _"Give an example of a tension between two pillars."_ → Security vs Cost Optimization is the classic one — more encryption, more logging, more redundant security checks all cost more and can add latency. A well-architected system doesn't maximize every pillar blindly, it makes a _deliberate trade-off_ appropriate to the workload (e.g., a healthcare app leans harder into Security even at extra cost; a hobby project leans harder into Cost Optimization).

3. _"How would you apply the Reliability pillar to a database?"_ → Multi-AZ deployment (Lesson 8), automated backups, and testing failure recovery — not just "it hasn't broken yet."

#### Scenario question

_"A junior engineer proudly demos a new feature: it's fast, cheap (single small EC2 instance, no redundancy), and works perfectly in the demo. Using the Well-Architected Framework, what questions would you ask before approving this for production?"_

---
### AWS Cloud Adoption Framework (CAF)

If the Well-Architected Framework is "how do we build this one system well," CAF is "how does an _entire company_ organize itself to move to the cloud at all" — it's zoomed out from a single architecture to a whole organization's transformation plan.
#### Definition

**AWS CAF** breaks an organization's cloud journey into **six perspectives**, grouped into two categories:

**Business-oriented:**

- **Business** — is the cloud migration tied to actual business outcomes/strategy?
- **People** — are staff trained, are roles/org structure ready for cloud ways of working?
- **Governance** — are there policies for how cloud resources get managed, who approves what?

**Technical-oriented:**

- **Platform** — the actual technical architecture (this is where Well-Architected plugs in)
- **Security** — org-wide security posture, not just one system's
- **Operations** — how the business runs and supports cloud workloads day-to-day

#### The industry problem this solves

Companies fail cloud migrations far more often from **people and process problems than technical ones** — nobody trained the ops team on the new tools, there's no policy for who's allowed to spin up a $10,000/month resource, leadership never defined what success even looks like. CAF exists because AWS learned, from thousands of enterprise migrations, that "just move the servers" fails without also addressing the organizational side. It's the difference between a technical migration and a _successful_ one.

#### Interview questions

1. _"What's the difference between the Well-Architected Framework and CAF?"_ → Well-Architected evaluates a specific system's technical architecture (six pillars). CAF is broader — an entire organization's readiness and strategy for cloud adoption (six perspectives, including non-technical ones like People and Governance).

2. _"Why would a company's cloud migration fail even with a technically sound architecture?"_ → Organizational gaps CAF addresses — untrained staff, no governance policy for cost/access control, no clear business objective driving the migration.

#### Scenario question

_"A company's engineering team has built a technically excellent, Well-Architected system on AWS, but six months in, costs are spiraling because any engineer can launch any resource with no approval process, and support tickets pile up because nobody was trained on the new monitoring tools. Which CAF perspectives failed here?"_

**The two that actually failed were Governance and People**

- **Governance** — "any engineer can launch any resource with no approval process" is exactly what Governance is supposed to prevent: policies controlling who can provision what, and cost guardrails
- **People** — "nobody was trained on the new monitoring tools" is a direct People-perspective failure — staff readiness for the new way of working

---

### Encryption Basics — At-Rest, In-Transit, and AWS KMS

Think of a bank locker room. Two very different security problems exist there:

- **The lockers themselves** (data at rest)
- **The armored van moving cash between branches** (data in transit)

#### Definition

**Encryption at rest** — data is encrypted while stored (disk, database, S3 object). If the physical medium or backend storage is compromised, the data is ciphertext without the key.

**Encryption in transit** — data is encrypted while moving across a network (client↔server, server↔server), almost always via TLS/SSL. Stops eavesdropping or tampering mid-flight.

**AWS KMS (Key Management Service)** — AWS's managed service for creating, storing, rotating, and controlling _access_ to the encryption keys that other services (S3, EBS, RDS, etc.) use to actually do the encrypting. KMS is the locksmith and key-vault, not the lock itself.

#### The industry problem this solves

Picture a **fintech app** storing bank statements. Two separate real problems:

1. **Storage compromise (why at-rest encryption exists).** If an attacker gains backend access to the S3 bucket or steals a disk snapshot, without at-rest encryption they read raw PII straight off the storage layer. With KMS-backed encryption, they get ciphertext they can't use without the key — and access to that key is itself logged and restrictable.

2. **Network interception (why in-transit encryption exists).** If the mobile app talks to the backend over plain HTTP, anyone on the same network (public WiFi, a compromised router) can sniff account numbers mid-flight. TLS makes that traffic unreadable even if intercepted.

3. **Compliance-driven key control (why KMS/CMKs exist).** A company under audit needs to prove _only_ the compliance team's role can decrypt PII, and that keys rotate annually. AWS managed keys can't give you that level of policy control — this is exactly when you reach for a CMK with a scoped key policy.

#### Code / hands-on

Enable default at-rest encryption on an S3 bucket using a customer managed key:

```
# Create a customer managed key
aws kms create-key --description "PII data encryption key"

# Enforce default encryption on the bucket using that key
aws s3api put-bucket-encryption \
  --bucket my-company-pii-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "<key-id>"
      }
    }]
  }'
```

`create-key` provisions the CMK. `put-bucket-encryption` makes encryption the _default_ — so even a developer who forgets to specify it on upload still gets encrypted objects.

For in-transit, there's usually no "code" — it's enforced by denying any non-HTTPS request via bucket policy:

```
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::my-company-pii-bucket/*",
  "Condition": { "Bool": { "aws:SecureTransport": "false" } }
}
```

#### Interview questions

**1. At-rest vs in-transit — can you have one without the other?**  
At-rest protects data while stored (disk/DB/S3); in-transit protects it while moving over a network (TLS). Yes, you can absolutely have one without the other — and it's a common misconfiguration. Example: an S3 bucket with KMS encryption enabled (at-rest (no)) but a bucket policy that still allows plain HTTP access (in-transit (yes)) — an attacker on the network path can still intercept the data _before_ it's written to disk. Real systems need both; each protects a different point of vulnerability.

**2. AWS managed vs customer managed key — when do you insist on a CMK?**  
AWS managed keys are auto-created and auto-rotated by AWS, but you can't write a custom key policy on them or control who specifically can use them beyond the service's own IAM permissions. CMKs let you write the exact key policy (who can encrypt/decrypt/disable), choose your own rotation schedule, and produce an audit trail of key usage via CloudTrail. You insist on a CMK whenever there's a compliance requirement naming _specific_ access control or rotation terms — "only role X can decrypt," "keys must rotate annually and we must prove it."

**3. Enforcing "only one IAM role can decrypt" in KMS**  
Write a **key policy** on a CMK that grants `kms:Decrypt` only to that role's ARN (and denies/omits it for everyone else, including other roles that might have broad IAM permissions elsewhere). Key policies are the "final say" for KMS — even if a user has `kms:*` in their IAM policy, the key policy itself must also grant access, so the CMK policy is the authoritative gate.

**4. Does S3 default encryption protect data in transit too?**  
No — default encryption only covers data at rest (what happens to the object once stored). It says nothing about how the object got there. In-transit protection is separate and comes from enforcing HTTPS (e.g., via the `aws:SecureTransport` deny condition in the bucket policy). People conflate these two, which is exactly why interviewers ask this.

**5. What happens if the KMS key is deleted?**  
The data becomes **permanently unrecoverable**. S3 objects encrypted under that key can never be decrypted again — there's no backdoor, no AWS override. This is exactly why KMS enforces a mandatory **7–30 day waiting period** before actually deleting a key (you schedule deletion, it doesn't happen instantly), specifically to prevent this kind of irreversible, catastrophic mistake.

#### Scenario question  
Your company stores customer PII in S3 and RDS. A compliance audit requires: (a) all data encrypted at rest, (b) only the finance team's IAM role can decrypt PII, (c) key usage rotated annually, (d) all API traffic to your app encrypted. Walk through how you'd satisfy each requirement using S3, RDS, KMS, and any other relevant AWS feature.

**(a) All data encrypted at rest — S3 and RDS**

- **S3**: Enable default bucket encryption (`put-bucket-encryption`) using SSE-KMS, so every object written is automatically encrypted without relying on developers remembering to set it per-upload.
- **RDS**: Enable encryption **at creation time** — RDS encryption can't be turned on for an existing unencrypted instance; you'd need to snapshot it, copy the snapshot with encryption enabled, and restore from that encrypted copy. This is a classic gotcha interviewers like to probe.

**(b) Only the finance team's IAM role can decrypt PII**

- Create a **Customer Managed Key (CMK)** in KMS specifically for this PII (don't use the AWS-managed default key — it doesn't give you this level of control).
- Write a **key policy** on that CMK granting `kms:Decrypt` only to the finance team's IAM role ARN. Remember: the key policy is the final gate — even if another role has broad `kms:*` IAM permissions, it still can't decrypt unless the key policy itself names it.
- Use that same CMK for both the S3 bucket's default encryption and the RDS instance's encryption, so one policy governs decrypt access across both data stores.

**(c) Key usage rotated annually**

- Enable **automatic key rotation** on the CMK — AWS rotates the underlying key material yearly while keeping the same Key ID, so nothing downstream (S3, RDS, your app) needs to change. This satisfies the audit requirement with zero application-side work.

**(d) All API traffic to the app encrypted**

- Enforce **TLS/HTTPS only** at the load balancer / API Gateway level — no HTTP listener, or an HTTP listener that only redirects to HTTPS.
- If S3 is accessed directly by clients, add the `aws:SecureTransport: false` **deny** condition to the bucket policy, so even direct S3 calls must use HTTPS.
- For RDS, enforce **SSL/TLS connections** from the application to the database (RDS supports requiring SSL for the DB connection itself, not just the app's external API).

---

### Compliance Programs

Think of a restaurant kitchen. The restaurant itself keeps things clean day-to-day — but a **health inspector** periodically comes in, checks against a standard checklist, and issues a certificate saying "this kitchen meets food-safety standards." Customers don't need to inspect the kitchen themselves — they trust the certificate.

AWS compliance programs work the same way: AWS's data centers and services get independently audited against external standards (SOC, PCI-DSS, HIPAA, ISO 27001, etc.), and AWS hands you the "certificate" — so _you_ don't have to independently verify AWS's physical security, network controls, etc. every time you build something.

**Compliance programs** are third-party audited certifications proving AWS's infrastructure meets specific regulatory or industry security standards. Common ones:

- **SOC 1/2/3** — general controls around security, availability, confidentiality (common baseline ask from any enterprise customer)
- **PCI-DSS** — required if you handle credit card data
- **HIPAA** — required if you handle US healthcare/patient data (AWS signs a BAA — Business Associate Agreement — for eligible services)
- **ISO 27001** — international information security management standard
- **FedRAMP** — US government cloud security standard

Critically: these certifications cover **AWS's side** of the Shared Responsibility Model — the physical data center, the hypervisor, the network. They do **not** automatically make _your_ application compliant. If you store unencrypted PII in a public S3 bucket, AWS being HIPAA-eligible doesn't save you — you still violated HIPAA on your side of the line.

**AWS Artifact** is the actual service/portal where you download these audit reports and certificates on demand (for your own audits or to hand to a customer's compliance team).

#### The industry problem this solves

A healthcare startup wants to launch on AWS but needs to prove to their board — and eventually federal auditors — that patient data is handled to HIPAA standard. Without AWS's compliance programs, that startup would need to independently audit AWS's physical data centers, which is neither possible nor AWS's business model. Instead: AWS provides the HIPAA eligibility + signs a BAA, the startup pulls the relevant audit report from **AWS Artifact**, and the startup's own compliance effort narrows to just _their_ side — encrypting data, controlling access, configuring services correctly. This is the practical, everyday use of Shared Responsibility, not just theory.

#### Interview questions

1. What's the difference between AWS being "compliant" with a standard and your application being compliant?       
- AWS being "compliant" means its physical infrastructure, hypervisor, and underlying services have been independently audited against a standard. It says nothing about how _you_ configure your application on top of it. You inherit AWS's compliant foundation, but your own encryption, access controls, and code are audited separately — compliance isn't automatically inherited end-to-end.

2. What is AWS Artifact, and what problem does it solve for a customer?     
- AWS Artifact is the self-service portal where customers download AWS's audit reports and compliance certificates (SOC 2, PCI-DSS, ISO 27001, etc.) on demand. It solves the problem of proving compliance to _your own_ auditors without needing AWS to individually hand you documents — you just pull the report yourself.

3. If a healthcare company wants to store patient data on AWS, what does AWS provide, and what is still the company's responsibility?     
- AWS provides HIPAA-eligible services and signs a BAA (Business Associate Agreement) covering its side of the infrastructure. The company is still responsible for encrypting the data, controlling who can access it, configuring those specific HIPAA-eligible services correctly, and everything happening in their own application code.

4. Name two compliance programs and what kind of data/industry each one is typically tied to.    
- PCI-DSS — tied to any company processing credit card/payment data. HIPAA — tied to US healthcare providers and anyone handling patient medical records.

#### Scenario question  
A startup handling credit card payments tells their investors "we're PCI-DSS compliant because we use AWS, which is PCI-DSS certified." Is this statement accurate? What's missing from their reasoning, and what would they actually need to do to be truly PCI-DSS compliant?

No, the statement is inaccurate. AWS being PCI-DSS certified only covers the infrastructure layer (data centers, hypervisor, network) — it doesn't make the startup's _application_ compliant. What's missing: they still need to secure how they handle card data themselves — encrypting it, restricting access, never logging raw card numbers, and undergoing their own PCI-DSS assessment (or using a certified payment processor like Stripe to avoid touching card data directly at all). "We use AWS" is necessary but nowhere near sufficient.

---

### Security Services — Shield, WAF, GuardDuty, Config, CloudTrail

Think of a bank branch's physical security setup:

- **Shield** — the reinforced walls and bulletproof glass, built to withstand a battering ram or a mob rushing the door (a flood of traffic trying to overwhelm the building itself)
- **WAF (Web Application Firewall)** — the security guard at the door checking IDs and bags, filtering out specific bad actors and suspicious requests before they even get inside
- **GuardDuty** — the silent alarm system and CCTV analyst, watching behavior patterns inside the branch and flagging "that person's been circling the vault for 20 minutes"
- **Config** — the compliance officer doing daily walkthroughs, checking "is every door locked the way policy says it should be, and when did that change?"
- **CloudTrail** — the security camera footage log — a permanent record of exactly who did what, when, whether it was normal or suspicious

#### Definition

- **AWS Shield** — DDoS protection. Shield Standard is free and automatic for all customers (absorbs common network/transport-layer floods). Shield Advanced is paid, adds larger-scale attack protection, 24/7 DDoS response team access, and cost protection for scaling charges incurred during an attack.

- **AWS WAF** — a web application firewall that filters HTTP/HTTPS requests based on rules you define (block specific IPs, block SQL-injection patterns, rate-limit an IP hitting your API too fast). Sits in front of CloudFront, ALB, or API Gateway.

- **Amazon GuardDuty** — continuous, intelligent threat detection. Analyzes VPC flow logs, DNS logs, and CloudTrail logs using machine learning to flag anomalous behavior (e.g., an EC2 instance suddenly talking to a known crypto-mining IP, or API calls from an unusual geography).

- **AWS Config** — tracks the _configuration state_ of your resources over time and checks them against rules you define (e.g., "no S3 bucket should ever be public"). It's about compliance/drift detection, not attack detection.

- **AWS CloudTrail** — logs every single API call made in your account (who, what action, when, from where) — the foundational audit trail almost every other security service (including GuardDuty) reads from.

#### The industry problem this solves

A gaming company launches a hit mobile game. Three different attacks/risks show up in the same week:

1. A **DDoS attack** floods their servers with junk traffic trying to take the game offline → **Shield** absorbs it at the network layer.
2. Attackers start probing their login API with SQL injection attempts → **WAF** rules block those requests before they reach the app.
3. An engineer's compromised laptop leads to an EC2 instance suddenly making calls to a known malicious IP at 3 AM → **GuardDuty** flags this anomaly automatically, faster than any human watching logs would catch it.
4. Weeks later, they discover an S3 bucket had been accidentally made public — **Config** would have flagged this the moment it happened if the rule was active, and **CloudTrail** lets them forensically trace exactly which API call made it public and when.

#### Code / hands-on
Enable GuardDuty and check for findings:

```bash
# Enable GuardDuty in your account/region
aws guardduty create-detector --enable

# List any findings it's already flagged
aws guardduty list-findings --detector-id <detector-id>
```

Check AWS Config for a specific compliance rule (e.g., is any S3 bucket public):

```bash
aws configservice describe-compliance-by-config-rule \
  --config-rule-names s3-bucket-public-read-prohibited
```

#### Interview questions with answers

1. _"What's the difference between Shield and WAF?"_ → Shield protects against DDoS (volume/availability attacks at the network layer); WAF filters and blocks specific malicious _requests_ based on rules (SQL injection, bad IPs, rate limits) at the application layer. They solve different problems and are often used together.

2. _"How does GuardDuty know something is suspicious without you writing rules for it?"_ → It uses machine learning and threat intelligence feeds against your VPC flow logs, DNS logs, and CloudTrail logs to detect anomalous patterns automatically, rather than relying on manually defined static rules like WAF does.

3. _"What's the difference between Config and CloudTrail?"_ → CloudTrail logs _every API call_ (the raw "who did what, when"). Config tracks the _resulting configuration state_ of resources over time and checks it against rules — Config often uses CloudTrail data under the hood to explain _why_ a resource's state changed.

4. _"If an S3 bucket became public by accident, which two services would you use to detect it and investigate it?"_ → Config to detect the non-compliant state (rule: no public buckets), CloudTrail to trace back exactly which API call and principal made it public and when.

#### Scenario question

A company gets hit with a sudden 10x traffic spike that's crashing their site, and separately their security team notices repeated failed login attempts from a single IP hammering their login API. Which two AWS services address each problem respectively, and why wouldn't one alone cover both?

**Shield** for the traffic-spike DDoS problem (network/availability layer), **WAF** for the repeated failed logins from a single IP (application layer, since you'd write a rate-limiting rule on that IP). One alone wouldn't cover both because Shield doesn't inspect individual request content/patterns, and WAF doesn't defend against raw volumetric floods — they operate at different layers of the stack.

---
### Trusted Advisor

Think of a car's annual inspection report. It doesn't fix anything itself — it just goes through a checklist (brakes, tires, lights, emissions) and tells you "this is wearing thin, fix it before it becomes a real problem." You still decide whether and how to act on it.

**AWS Trusted Advisor** is that inspection report for your entire AWS account — it doesn't change anything automatically, it just scans your setup and flags issues across five categories.
#### Definition

**Trusted Advisor** continuously inspects your AWS account and gives recommendations across five categories:

- **Cost Optimization** — idle/underutilized resources (e.g., an EC2 instance running at 2% CPU for weeks)
- **Performance** — service limits approaching, suboptimal configurations
- **Security** — security groups wide open (0.0.0.0/0), MFA not enabled on root, S3 buckets with open access
- **Fault Tolerance** — resources not backed up, not multi-AZ
- **Service Limits** — approaching account quotas (e.g., close to your EC2 instance limit)

Free tier accounts only get a handful of core checks (mainly security). The **full check list** requires **Business or Enterprise support plans**

#### The industry problem this solves

A startup's AWS bill balloons unexpectedly. Nobody notices because no single engineer is watching every idle resource across dozens of services. Trusted Advisor's Cost Optimization check flags "these 6 EBS volumes are unattached and have been for 45 days" — something that would otherwise sit forgotten and cost money indefinitely. Separately, a security group with SSH open to `0.0.0.0/0` (a real, common misconfiguration) gets flagged before it's exploited rather than after — Trusted Advisor is often the reason a company catches a misconfiguration _before_ an incident, not during a post-mortem.

#### Code / hands-on

```bash
# List all Trusted Advisor checks available to your account
aws support describe-trusted-advisor-checks --language en

# Get the result of a specific check (needs Business/Enterprise support for most checks)
aws support describe-trusted-advisor-check-result --check-id <check-id>
```

#### Scenario question

Your company is on the Basic support plan and a new engineer says "Trusted Advisor will tell us everything wrong with our security setup, we don't need a separate security review." Is this reasoning sound? What's the gap in their assumption?

Basic plan gap means a real security review (IAM audits, pen testing, architecture review) is still necessary; Trusted Advisor is a surface-level automated checklist, not a substitute for depth.

---

### AWS Organizations

Think of a large corporation with a parent company and several subsidiaries — Engineering Corp, Marketing Corp, Finance Corp — each with its own bank account and its own spending limits, but the parent company gets one **consolidated invoice** at month-end, sets company-wide policies (like "no subsidiary can wire money to embargoed countries"), and can see everyone's books. Each subsidiary still operates independently day-to-day, but the parent has centralized control and visibility it couldn't have if these were entirely separate companies.

**AWS Organizations** is that parent-company structure for multiple AWS accounts.

#### Definition

**AWS Organizations** lets you centrally manage multiple AWS accounts under one **management account** (formerly called "master account"). Core pieces:

- **Organizational Units (OUs)** — folders to group accounts (e.g., `Production/`, `Dev/`, `Finance/`), so policies apply to whole groups at once instead of per-account.

- **Service Control Policies (SCPs)** — guardrails applied at the OU or account level that define the _maximum_ permissions available — even if an account's own IAM policy allows something, an SCP can override and block it. Critical nuance: SCPs don't _grant_ permissions, they only restrict the ceiling.

- **Consolidated Billing** — one bill across all accounts, and this unlocks **volume pricing tiers** you wouldn't hit on any single account alone (e.g., combined S3 usage across 10 accounts might cross a discount threshold that no single account would reach).

The reason companies split into multiple accounts at all (rather than one account with lots of IAM roles) is **blast radius isolation** — if the Dev account gets compromised or someone runs a destructive script, Production is a completely separate account boundary, not just a different IAM role within the same account.

#### The industry problem this solves

A mid-size company has three teams: Dev, Staging, Prod. Early on, everything lives in one AWS account, separated only by IAM roles and tags. Then a junior developer, testing a script in what they _thought_ was the dev environment, accidentally deletes a production RDS instance — because IAM permissions, not account boundaries, were the only thing standing between "test" and "real."

With AWS Organizations, Dev/Staging/Prod become **separate accounts** under OUs, each account being a genuinely separate security boundary — not just a permission scope. An SCP on the Dev OU can outright block `rds:DeleteDBInstance` account-wide, regardless of what any individual IAM policy inside that account says. That's a categorically stronger guarantee than "we trust everyone's IAM role scoping."

#### CODE

```
# Create an organization
aws organizations create-organization --feature-set ALL

# Create an OU under the root
aws organizations create-organizational-unit \
  --parent-id <root-id> --name "Production"

# Attach an SCP to that OU denying a specific dangerous action
aws organizations attach-policy \
  --policy-id <scp-id> --target-id <ou-id>
```

#### Interview questions with answers (leveled up)

1.  _"An account admin has `AdministratorAccess` in IAM. An SCP on their OU denies `ec2:TerminateInstance`. Can they terminate an EC2 instance?"_ → No. SCPs set the maximum permission ceiling regardless of what IAM grants — even full admin IAM access is capped by whatever the SCP allows. This is the single most misunderstood SCP behavior and a favorite trick question.

2.  _"Why would a company with strong IAM discipline still adopt multi-account + Organizations instead of relying on IAM roles within one account?"_ → Blast radius isolation — IAM misconfigurations, compromised credentials, or destructive scripts in one account can't cross into another account's resources at all, whereas within a single account, a misconfigured IAM policy is often the _only_ thing separating dev mistakes from production damage.

3.  _"Does an SCP attached to an OU apply retroactively to permissions already granted via IAM, or does it only affect new policy evaluations?"_ → SCPs are evaluated on every single API call in real time — there's no "grandfathering." The moment an SCP is attached, it immediately restricts all matching calls, including ones from existing IAM policies that were already in place.

4.  _"Consolidated billing gives volume discounts. Does it also mean any account in the Organization can see or access another account's resources by default?"_ → No — billing consolidation and resource/permission access are completely separate. Combined billing doesn't grant cross-account resource access; that still requires explicit cross-account IAM roles or resource policies.

#### Scenario question

Your company has Dev, Staging, and Prod accounts under one AWS Organization. Someone proposes: "Let's just write a strict IAM policy in the Prod account that blocks all destructive actions for non-senior engineers — that gives us the same safety as an SCP, so we don't need to bother with SCPs at all." Where does this reasoning break down, and what specific failure scenario would an SCP catch that IAM policy alone would miss?

**Reason** - An IAM policy lives _inside_ the Prod account and can be modified, deleted, or overridden by anyone with sufficient IAM permissions in that account — including the account's own root user or any admin. So "blocks destructive actions for non-senior engineers" only holds as long as nobody with admin rights changes that policy — and a compromised admin credential, a misconfigured new IAM role, or even a well-meaning senior engineer "temporarily" loosening the policy for a hotfix completely defeats it. It's a self-imposed guardrail that the account itself can undo.

**The specific failure scenario an SCP catches that IAM alone misses:** someone (attacker, or a rushed engineer) creates a **brand new IAM role with AdministratorAccess** in the Prod account — bypassing the carefully scoped "non-senior engineer" policy entirely, since the new role was never subject to it. With an SCP attached at the OU level, that new role — no matter how it was created or what IAM permissions it has — is still capped by the SCP's deny on destructive actions, because SCPs are enforced from _outside_ the account, by the Organization's management account, and can't be overridden by anything happening inside Prod itself. That's the real distinction: IAM is a ceiling set from inside; SCP is a ceiling enforced from outside, immune to in-account privilege escalation.

---

### IAM Best Practices (MFA, Root Account)

Think of your house: the **front door key** is your everyday login, but the **master key that unlocks every door, disables the alarm, and can sell the house** is the root account. You don't carry that master key in your pocket day-to-day — you lock it in a safe and only pull it out for the rare situations that truly require it. Using it for daily tasks massively increases the odds you lose it, leak it, or it gets stolen.

#### Definition

-   **Root account** — created when you first open an AWS account, tied to the account owner's email, and has **unrestricted access to everything**, including actions no IAM policy can restrict (billing changes, closing the account, changing support plan). It cannot be permission-limited by any policy — this is the key exam distinction.
-   **MFA (Multi-Factor Authentication)** — requiring a second factor (authenticator app code, hardware key) beyond just a password, so a leaked password alone isn't enough to log in.
-   **IAM best practices** bundle a few concrete rules: never use root for daily work, enable MFA on root (and ideally all human users), create individual IAM users/roles per person (never share credentials), grant least-privilege permissions, use roles instead of long-lived access keys wherever possible.

#### The industry problem this solves

A startup's only AWS login is the root account, shared via a password manager entry the whole team uses. An engineer accidentally commits that root password to a public GitHub repo in a config file. Within hours, an attacker has spun up dozens of expensive GPU instances for crypto mining — and because it's root, there's no IAM policy that could have limited the blast radius; root simply _is_ unrestricted. With MFA enabled on root, the leaked password alone wouldn't have been enough. With individual IAM users instead of a shared root login, the company would also know exactly _whose_ credential leaked, rather than "someone, somewhere, using the one shared login."

#### CODE
```
# Check whether root account has MFA enabled (via IAM credential report)
aws iam generate-credential-report
aws iam get-credential-report --output text --query 'Content' | base64 -d

# Create a dedicated IAM user instead of using root for daily work
aws iam create-user --user-name anubhav-dev

# Enforce MFA requirement via IAM policy condition
```
```
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": { "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" } }
}
```
#### Interview questions

1.  _"Can an SCP restrict what the root user of a _management_ account can do in an AWS Organization?"_ → No — SCPs never apply to the management account's root user, and this is a frequently-tested exception. SCPs restrict member accounts, but the management account root remains fully unrestricted, which is exactly why securing the management account root credentials is treated as the single highest-priority security task in a multi-account org.

3.  _"If root credentials are leaked, does deleting the root user's access keys fully mitigate the risk?"_ → No — root doesn't require access keys to do damage; anyone with the root password (and no MFA) can log into the console directly and take any action, including generating new access keys. Deleting keys helps against programmatic/API abuse but doesn't close the console login vector at all.

4.  _"Why is 'use IAM roles instead of long-lived IAM user access keys' considered a best practice, specifically in terms of attack surface?"_ → Long-lived access keys, once leaked (committed to a repo, logged accidentally), remain valid indefinitely until manually rotated or revoked. Roles issue short-lived, automatically-expiring temporary credentials — so even if one leaks, the exploitation window is measured in hours, not until someone notices.

5.  _"A company has MFA enabled on root but the root user still has active long-term access keys from years ago that nobody remembers creating. What's the actual risk here, given MFA is already on?"_ → MFA protects the _console_ login path, but long-term access keys authenticate purely via key + secret with no MFA check at all for API/CLI calls — so those forgotten keys are a completely separate, MFA-bypassing attack surface. Best practice is root should have **zero** active access keys, period, not just MFA on the console.

#### Scenario question

 A security review finds that your Prod account's root user has MFA enabled, but a developer mentions "we also have a break-glass emergency admin IAM role that anyone on the ops team can assume without MFA, in case root gets locked out." Is this a legitimate design pattern or a hidden vulnerability? Justify your answer and describe how you'd fix it if it's a problem.
 
 This is a **hidden vulnerability**, not a legitimate pattern. An MFA-protected root account means nothing if there's a _side door_ — an IAM role with equal or near-equal power that skips MFA entirely. Anyone who compromises one ops team member's regular (non-MFA-protected) credentials can assume that break-glass role and get admin access, completely bypassing the root MFA control you just spent effort securing.
 **Fix:** require MFA as a _condition_ on the role's trust policy itself (`aws:MultiFactorAuthPresent: true`), so assuming the role requires MFA too — "break glass" should still mean breaking something, not walking through an open door.

---
## Domain 1 + 2 Combined Notes

#### Domain 1 — Cloud Concepts

-   **Cloud computing** = on-demand IT resources over the internet, pay-as-you-go, no upfront hardware.

-   **Deployment models**: Public (AWS itself), Private (dedicated), Hybrid (mix — often for data residency/legacy systems).

-   **Service models**: IaaS (EC2 — you manage OS up), PaaS (Elastic Beanstalk — you manage code only), SaaS (fully managed app, e.g. Gmail).

-   **Global Infrastructure**: Region (geographic area, isolated) → AZ (independent data center(s) within a Region, fault tolerance) → Edge Location (CloudFront/Route 53 caching, latency only, no compute).

-   **Well-Architected Framework**: 6 pillars — Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.

-   **AWS CAF**: helps _organizations_ (not just tech) plan cloud migration — Business, People, Governance, Platform, Security, Operations perspectives.

#### Domain 2 — Security

-   **Shared Responsibility Model**: AWS secures _of_ the cloud (infrastructure, hardware, hypervisor); you secure _in_ the cloud (data, IAM, OS patching on EC2, network config).
 
-   **IAM**: Users (people), Roles (assumable, temporary — preferred over long-term keys), Groups, Policies (JSON, least privilege).

-   **Encryption**: At-rest (stored data) vs in-transit (TLS, moving data) — need both. **KMS** manages keys; AWS-managed (auto) vs CMK (custom policy/rotation control).

-   **Compliance**: AWS's certs (SOC/PCI/HIPAA/ISO) cover _AWS's_ side only — your app compliance is separate. **AWS Artifact** = download portal for these reports.

-   **Security services**: Shield (DDoS/network layer), WAF (app-layer request filtering), GuardDuty (ML-based anomaly detection), Config (configuration compliance/drift), CloudTrail (full API audit log — the source of truth others read from).

-   **Trusted Advisor**: 5 categories (Cost, Performance, Security, Fault Tolerance, Service Limits) — recommends, doesn't auto-fix; full checks need Business/Enterprise support.

-   **AWS Organizations**: Management account + OUs + **SCPs** (permission _ceiling_, don't grant anything, apply even over full IAM admin — except never restrict the _management account's own root_). Consolidated billing = volume discounts, not shared resource access.

-   **Root account**: unrestricted, can't be IAM-limited, no SCP applies to management account root. MFA on root is priority #1; zero access keys on root; use IAM roles (temporary creds) over long-term keys.

----------
## CCP-Style Scenario Practice (Domain 1 + 2)

**Q1.** A company wants to keep customer data within India due to legal requirements but wants disaster recovery. What should they do?  
**A.** Deploy in `ap-south-1` (Mumbai) across **multiple AZs** within that same Region for fault tolerance, and avoid enabling cross-Region replication to any Region outside India. Region choice satisfies data residency; multi-AZ satisfies DR — both without leaving the country.

**Q2.** A media company serves video content globally from servers in Mumbai. US users report slow load times. What's the most cost-effective fix?  
**A.** Use **CloudFront** with Edge Locations to cache static content near US users — no need to duplicate the entire backend infrastructure in a US Region, since it's a latency problem for cacheable content, not a compute-availability problem.

**Q3.** A company's EC2 instance was compromised because SSH (port 22) was open to `0.0.0.0/0`. Which service would have flagged this _before_ the incident, and which policy prevents it going forward?  
**A.** **Trusted Advisor** (Security category) or **AWS Config** (custom rule) would flag the open security group. Going forward, an **SCP** at the OU level could deny overly permissive security group creation account-wide, regardless of individual IAM permissions.

**Q4.** An enterprise wants one consolidated invoice across 15 AWS accounts (Dev/Staging/Prod per team) and wants Prod protected from accidental deletions even by account admins. What two AWS Organizations features solve this?  
**A.** **Consolidated billing** for the single invoice + volume discount, and an **SCP** attached to the Prod OU denying destructive actions (e.g., `rds:DeleteDBInstance`) — enforced regardless of what IAM permissions exist inside that account.

**Q5.** A healthcare startup needs to prove HIPAA compliance to auditors without commissioning their own physical data-center audit. What should they do?  
**A.** Sign a **BAA** with AWS for HIPAA-eligible services, and pull AWS's own third-party HIPAA audit documentation from **AWS Artifact** — this covers AWS's side; the startup still separately secures their own application layer (encryption, access control).

**Q6.** A financial firm requires: PII encrypted at rest, only the compliance team can decrypt it, and keys rotated yearly without app changes. What's the minimal correct setup?  
**A.** A **KMS Customer Managed Key (CMK)** with a key policy scoping `kms:Decrypt` to the compliance team's IAM role, and **automatic annual rotation** enabled — rotation keeps the same Key ID so nothing downstream (S3/RDS) needs reconfiguring.

**Q7.** An account was breached via a leaked root password with no MFA. The team then enables MFA and assumes the account is now fully secured. What's the gap?  
**A.** MFA only protects the **console login path** — if root has any active **long-term access keys**, those bypass MFA entirely for programmatic access. Root should have **zero** active access keys, not just MFA on login.

**Q8.** A junior engineer asks why the company bothers with separate Dev/Staging/Prod AWS _accounts_ instead of just IAM roles within one account. What's the strongest technical justification?  
**A.** **Blast radius isolation** — accounts are a hard security boundary; a compromised credential or destructive script in Dev literally cannot touch Prod resources, whereas within a single account, IAM misconfiguration is the _only_ thing separating environments — a much weaker guarantee.

**Q9.** A retail company's traffic surges 10x during a flash sale, crashing the site, while separately bots are hammering the login page with credential-stuffing attempts. Name the two distinct services addressing each, and why can't one do both?  
**A.** **Shield** for the volumetric traffic surge (network/availability layer, no content inspection), **WAF** for credential stuffing (application layer — rate-limiting/rule-based request filtering). Shield doesn't inspect request patterns; WAF doesn't absorb raw volumetric floods — different layers entirely.

**Q10.** A compliance audit asks: "Show us exactly who made this S3 bucket public, and when." Which service answers this, and which service would have prevented/flagged it happening at all?  
**A.** **CloudTrail** provides the forensic API-call record (who/what/when). **AWS Config** (with a rule like `s3-bucket-public-read-prohibited`) would have flagged the non-compliant state the moment it occurred, ideally triggering an alert before the audit ever needed to ask.

---
### AWS Lambda

Think of hiring a contractor for a single specific task — say, fixing a leaky faucet — versus keeping a full-time plumber on payroll. The full-time plumber (an EC2 instance) sits there whether there's work or not, and you pay for their time regardless. The contractor (Lambda) shows up, does exactly the one job, and leaves — you only pay for that job, and if no faucet leaks that month, you pay nothing.

**AWS Lambda** is compute that exists _only_ for the duration of the task — it doesn't idle, doesn't need provisioning, and someone else (AWS) manages the "plumber's truck and tools" (the underlying server) entirely.

#### Definition

**Lambda** is a **serverless, event-driven compute service** — you upload code (a "function"), and AWS runs it in response to a trigger (an S3 upload, an API Gateway request, a scheduled CloudWatch event, a DynamoDB change) without you provisioning or managing any server.

#### Key characteristics:

-   **You pay per invocation + execution duration** (measured in milliseconds), not for idle time — fundamentally different billing model from EC2.

-   **Max execution time: 15 minutes** — Lambda is built for short, bursty tasks, not long-running processes. This ceiling is a favorite exam trap.
 
-   **Cold starts**: if a function hasn't run recently, the first invocation has extra latency while AWS provisions the execution environment. Frequently-invoked functions stay "warm" and skip this.

-   **Concurrency**: Lambda auto-scales by running many invocations in parallel — but there's an account-level concurrency limit (default 1000, raisable) that can throttle you under sudden massive load if not configured with reserved concurrency.

-   **Stateless by design**: each invocation should assume no memory of the previous one (unless you explicitly use external storage like DynamoDB or S3 for state).

#### The industry problem this solves

An e-commerce company needs to resize product images the moment they're uploaded to S3. The old way: run a dedicated EC2 instance 24/7 polling S3 for new uploads — paying for compute even during the 90% of the day nothing gets uploaded. With Lambda: an S3 upload event directly triggers a Lambda function that resizes the image and exits. No idle server, no capacity planning for "how many images might get uploaded at once" — Lambda scales to handle 1 upload or 10,000 uploads simultaneously, and the company pays only for the actual milliseconds of processing that occurred.

This is also _exactly_ the kind of workload that's a poor fit for Lambda if misunderstood: a company tries to use Lambda to process a 2-hour video transcoding job and hits the 15-minute timeout wall — that's a sign the workload belongs on EC2 or a container service, not Lambda. Knowing _when not_ to use Lambda is as testable as knowing when to use it.

#### CODE

```
# Create a Lambda function from a zipped code package
aws lambda create-function \
  --function-name resize-image \
  --runtime python3.12 \
  --role arn:aws:iam::<account-id>:role/lambda-execution-role \
  --handler resize.handler \
  --zip-file fileb://function.zip

# Set up S3 to trigger it on new object upload
aws lambda add-permission \
  --function-name resize-image \
  --statement-id s3-trigger \
  --action "lambda:InvokeFunction" \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-uploads-bucket

# Invoke manually to test
aws lambda invoke --function-name resize-image output.json
```

The IAM role attached (`lambda-execution-role`) is critical — Lambda's permissions come entirely from that role, not from whoever deployed it. — Lambda is one of the most common real-world examples of "a service assuming a role" rather than a human doing so.

#### Interview questions with answers (leveled up)

1. _"A company wants to migrate a video transcoding job that regularly takes 40 minutes to Lambda for cost savings. What's wrong with this plan, and what would you recommend instead?"_ → Lambda has a hard 15-minute maximum execution timeout — a 40-minute job cannot run on Lambda at all, regardless of memory/config tuning. The right fit is a container-based service (ECS/Fargate) or EC2, which support long-running processes without an artificial time ceiling.
2. _"Your Lambda function processes payment webhooks and occasionally receives a burst of 5,000 simultaneous requests. What could go wrong, and how would you prevent it?"_ → If account-level concurrency limits (default 1000) are shared across other functions/services, this burst could throttle invocations — some webhooks would fail to process. Fix: configure **reserved concurrency** on this specific function to guarantee it capacity, and/or use an SQS queue in front of Lambda to buffer and smooth the burst instead of hitting Lambda directly.
3. _"Why is a cold start a bigger concern for a user-facing API than for a nightly batch job, even though the actual added latency might be the same number of milliseconds?"_ → A user-facing API's cold start directly delays a real person waiting for a response — degrading perceived performance/UX in the moment. A nightly batch job has no human waiting in real time, so the same few hundred milliseconds of added latency is functionally invisible to anyone.
4. _"You're told 'Lambda functions should be stateless.' A developer stores a counter variable at the global scope of the function code, outside the handler, and it seems to persist across some invocations but not others. Why does this happen, and why is relying on it dangerous?"_ → AWS may reuse a "warm" execution environment for consecutive invocations, which coincidentally lets global-scope variables persist — but this is an unreliable implementation detail, not a guarantee. A cold start, a scale-out event, or AWS recycling the environment wipes it with zero warning, silently corrupting logic that assumed persistence. State must live in an external store (DynamoDB, S3, ElastiCache) to be reliable.