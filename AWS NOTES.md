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


#### Scenario question

A startup's API is built entirely on Lambda + API Gateway. During a viral traffic spike, some users start seeing failed requests, but CloudWatch shows the functions themselves aren't erroring — they're just not being invoked at all for some requests. What's the most likely underlying cause given what you now know about Lambda, and what are two different ways you could architect around it?

The most likely cause is hitting the **account-level Lambda concurrency limit** (default 1000) — once that ceiling is hit, API Gateway requests that can't get a Lambda execution slot are **throttled (HTTP 429)**, not passed through as a function error. That's exactly why CloudWatch shows no errors _inside_ the functions — the functions never got invoked in the first place; the request was rejected before reaching them.

**Two architectural fixes:**

1. **Request a concurrency limit increase** from AWS and set **reserved concurrency** on this specific critical function so it always has guaranteed capacity, immune to other functions in the account eating the shared pool.
2. **Put an SQS queue in front of Lambda** — API Gateway writes the request to the queue instantly (fast, always succeeds), and Lambda drains the queue at whatever pace it can sustain. This decouples "accepting the request" from "processing it immediately," smoothing spikes instead of dropping requests during them.

---

### Elastic Beanstalk

Imagine you want to open a restaurant. You could buy the building, install plumbing, wire the electrics, hire an architect, get permits, build the kitchen from scratch (that's raw EC2 — you manage everything). Or you could lease a fully-fitted commercial kitchen space where the plumbing, gas lines, and electrical are already done — you just bring your recipes and staff and start cooking (that's Elastic Beanstalk). You still _own_ the restaurant and can walk into the kitchen and rewire something if you really want to — but you don't _have_ to touch the infrastructure to get started.

### Definition

Elastic Beanstalk is a **Platform-as-a-Service (PaaS)** orchestration layer built on top of underlying AWS services (EC2, Auto Scaling, ELB, S3, CloudWatch). You upload your application code (Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker), and Beanstalk automatically handles:

- Provisioning the EC2 instances to run it
- Setting up load balancing and auto scaling
- Health monitoring and basic auto-recovery
- Deployment (including rolling updates, blue/green via swapping environment URLs)

Critically: **the underlying resources are not hidden from you** — unlike a fully-managed service like Lambda, Beanstalk provisions real EC2 instances, a real ELB, a real Auto Scaling group, and you can go into the EC2 console and see them, SSH into them, and modify them directly if needed. Beanstalk is an _orchestration convenience layer_, not an abstraction that removes the servers.

**you pay for the underlying resources (EC2, ELB, etc.) — Beanstalk itself is free.** It's not a separate billed service; it's automation on top of ones you already pay for.

### The industry problem this solves

A small startup has a Node.js web app ready to ship. The founders don't have a dedicated DevOps engineer. The "correct" manual path is: launch EC2 instances, configure an Auto Scaling group, set up an Application Load Balancer, configure health checks, set up a CI/CD deployment pipeline for rolling updates, wire CloudWatch alarms — easily a week of infra work for a two-person team who just want their app live.

With Beanstalk: `eb init`, `eb create`, `eb deploy` — and within minutes they have a load-balanced, auto-scaled, monitored environment running their code, with none of that manual wiring. They can focus on the app; Beanstalk handles the operational scaffolding.

The flip side (equally testable): a team that _needs_ fine-grained control over their networking topology, custom AMIs with heavy pre-baked configuration, or non-standard architectures finds Beanstalk's opinionated structure limiting — that's when you drop to raw EC2 + your own orchestration, or move to ECS/Fargate for containerized workloads with more control.

#### CODE 

```
# Initialize a Beanstalk application in your project directory
eb init -p node.js my-app --region us-east-1

# Create an environment and deploy — this provisions EC2, ELB, ASG, security groups, etc.
eb create my-app-env

# Deploy a new version of your code to the existing environment
eb deploy

# Check environment health
eb health

# Open the running app in your browser
eb open
```

Behind the scenes, `eb create` is essentially doing the EC2 + ELB + Auto Scaling Group work, using a config you can still override via a `.ebextensions/` folder (YAML/JSON config files) if you need to customize instance type, environment variables, or scaling triggers.

### Interview questions with answers

1. **"Your team says 'Elastic Beanstalk is a PaaS, so we don't need to think about EC2 costs — Beanstalk pricing covers it.' What's wrong with this statement?"**  
→ Elastic Beanstalk itself has **no additional charge** — you pay only for the underlying resources it provisions (EC2 instances, ELB, EBS volumes, data transfer). Thinking of it as a separately-priced service is a fundamental misunderstanding; cost optimization on a Beanstalk app means the exact same EC2/ELB cost levers as a manually-built environment (right-sizing instances, Reserved/Spot pricing, etc.).

2. **"A developer deploys an app via Beanstalk, then manually goes into the EC2 console and changes the instance type directly on the running instances. Two days later, Beanstalk performs a routine environment update and the change is gone. Why?"**  
→ Beanstalk treats its environment configuration as the source of truth and can reprovision or update instances to match that stored configuration during health checks, scaling events, or deployments — manual out-of-band changes to the underlying EC2 instances aren't tracked by Beanstalk and get overwritten. The correct way to make the change persistent is through Beanstalk's own configuration (console settings or `.ebextensions`), not by hand-editing the EC2 resources it manages.

3. **"When would you choose Beanstalk over plain EC2, and when would you choose ECS/Fargate over Beanstalk?"**  
→ Beanstalk over EC2: when you want fast time-to-deploy for a standard web app/API without hand-building the ASG/ELB/monitoring stack, and your app fits a supported platform. ECS/Fargate over Beanstalk: when your workload is containerized and you need more granular control over container orchestration, task placement, or you're running multiple microservices that don't fit Beanstalk's single-application-environment model as cleanly.

### Scenario question (answer next chat)

A team deploys their app on Elastic Beanstalk with default settings. During a load test, they notice the app becomes unresponsive under high traffic, but when they check, only **one EC2 instance** is running the whole time — no new instances were launched despite CPU sitting at 95%+. Beanstalk is supposed to handle scaling automatically. What's the most likely misconfiguration, and where would you go to fix it?

The most likely cause: the Beanstalk environment was created as a **"Single Instance" environment type** (or, if load-balanced, the Auto Scaling Group has **Min = Max = 1**) — either way, there's no headroom for Beanstalk to scale into, so no matter how high CPU climbs, there's nowhere for a second instance to go.

Where to fix it: **Beanstalk Console → Configuration → Capacity**. There you'd:

1. Change the environment type from "Single instance" to "Load balanced, auto scaling" if it wasn't already
2. Raise Max instances above 1 (e.g., to 4)
3. Verify a scaling trigger exists (e.g., CloudWatch alarm on average CPU > 70% for N minutes) — because even with Max raised, if no trigger is configured, the ASG still won't know _when_ to add instances

This is the classic gotcha: people assume "Beanstalk = auto scaling by name" and never check that the ASG's own Min/Max and trigger policy were actually set up for growth.

---

### Auto Scaling (EC2 Auto Scaling Groups)

Think of a restaurant kitchen staffing plan. On a slow Tuesday afternoon, one cook is enough. On Friday dinner rush, you need five. A bad manager either keeps five cooks on payroll all week (wasteful) or keeps one cook all week and lets Friday dinner collapse into chaos (unreliable). A good manager has a **staffing policy**: "if the ticket queue backs up past X, call in another cook; if it's been quiet for 20 minutes, send someone home." Auto Scaling is that staffing policy, automated, for EC2 instances.
#### Definition

An **Auto Scaling Group (ASG)** is a logical collection of EC2 instances that AWS automatically grows or shrinks based on defined rules, so capacity matches actual demand instead of being fixed.

Three core numbers define every ASG:

- **Minimum** — the floor; ASG never scales below this, even at zero traffic
- **Maximum** — the ceiling; ASG never scales above this, even under extreme load (a cost/safety guardrail)
- **Desired** — the current target the ASG is actively trying to maintain; scaling actions move this number up or down between Min and Max

Scaling happens via **scaling policies**, the most common being:

- **Target tracking** — "keep average CPU at 50%" — you set a target metric value, AWS handles the math of how many instances that requires (this is the modern default, least manual)
- **Step scaling** — explicit rules like "if CPU > 80% for 5 min, add 2 instances; if CPU > 90%, add 4"
- **Scheduled scaling** — "every weekday at 8:59am, set desired = 10" for predictable traffic patterns (e.g., a B2B app that's busy during business hours and dead at night)

ASGs also handle **health checks and self-healing**: if an instance fails a health check (EC2 status check or an attached ELB health check), the ASG terminates it and launches a replacement automatically — this is a distinct benefit from scaling itself.

### The industry problem this solves

A ticket-booking site sells concert tickets that go on sale at exactly 10:00am. At 9:59am, traffic is near zero. At 10:00am, it's 50x normal traffic for 15 minutes, then drops back down. Provisioning for peak 24/7 means paying for 50x capacity almost all day for nothing. Provisioning for average traffic means the site falls over exactly when it matters most — at the moment of the sale.

With Auto Scaling: scheduled scaling can pre-warm capacity right before 10:00am (since target tracking alone might react a few minutes too slowly to a sudden 50x spike), then target tracking takes over to fine-tune during the rush, and the ASG scales back down afterward. The company pays for high capacity only during the ~20 minutes it's actually needed.

#### CODE

```
# Create a launch template (defines what an instance in the ASG looks like)
aws ec2 create-launch-template \
  --launch-template-name my-app-template \
  --version-description v1 \
  --launch-template-data '{"ImageId":"ami-0abcd1234","InstanceType":"t3.micro"}'

# Create the Auto Scaling Group itself
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-app-asg \
  --launch-template LaunchTemplateName=my-app-template,Version='1' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-abc123,subnet-def456" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/my-tg

# Attach a target tracking scaling policy (keep avg CPU near 50%)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-app-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
      "PredefinedMetricSpecification": {"PredefinedMetricType": "ASGAverageCPUUtilization"},
      "TargetValue": 50.0
    }'
```

Notice `--vpc-zone-identifier` takes **multiple subnets across different AZs** — this is deliberate. An ASG that only spans one AZ has no fault tolerance if that AZ goes down; spreading across AZs (from Lesson 8's Region/AZ hierarchy) is how Auto Scaling contributes to high availability, not just cost efficiency.

#### Interview questions with answers 

1. **"Your ASG has Min=2, Max=10, target tracking on CPU at 50%. During a traffic spike, CPU hits 95% but the ASG only ever scales to 4 instances, then stalls — even though Max is 10. What could be limiting it, and it's not the ASG config itself?"**  
→ Something _outside_ the ASG's own settings is capping it — most commonly: 
(1) an **EC2 service quota** (vCPU limit per account/region) being hit, 
(2) a **subnet running out of available IP addresses** in the AZs the ASG is deployed into, or 
(3) an attached Elastic Load Balancer or target group having its own capacity/registration limits. This is a favorite curveball because candidates assume Min/Max is the only ceiling — in reality the ASG can be blocked by quiet failures elsewhere in the account that don't show up in the ASG's own dashboard, only in its Activity History/scaling activity error messages.

2. **"Why would you use Scheduled Scaling _in addition to_ Target Tracking rather than just Target Tracking alone, for a known traffic pattern like a lunch rush?"**  
→ Target tracking is _reactive_ — it scales only after the metric (e.g., CPU) has already crossed the threshold, meaning there's a lag between demand rising and capacity catching up, during which users can experience degraded performance. Scheduled scaling is _proactive_ — it pre-provisions capacity ahead of a known spike (e.g., set desired capacity higher at 11:45am for a 12pm lunch rush) so instances are already warm and ready the moment demand hits, then target tracking handles any additional fine-tuning beyond the predictable baseline.

3. **"A cost-conscious manager sets Min=0 on an ASG serving a production API to 'save money during quiet periods.' What's the risk?"**  
→ If desired capacity ever scales down to 0, there are zero running instances — meaning the API is completely unreachable, not just slow, until a scaling trigger fires and a new instance boots (which itself takes time — no instant recovery). For a production-facing API, Min should be set to whatever the minimum instance count is that can still serve baseline traffic (often 1–2, spread across AZs), never 0; Min=0 is only appropriate for batch/worker ASGs where "no work available" genuinely means "nothing should be running."

#### Scenario question

An ASG is configured correctly with Min=2, Max=8, target tracking at 50% CPU, spread across 3 AZs. One day, one entire AZ goes down (a rare but real AWS event). What happens to the ASG's running instances and its ability to maintain desired capacity, and what's the one config choice from _today's_ lesson that determines whether the app stays available or not?

If one AZ goes down, the ASG **loses whatever instances were running in that AZ** — EC2 health checks (or ELB health checks) detect them as unreachable, and the ASG terminates them and tries to launch replacements to get back to desired capacity. Because the ASG spans 3 AZs, it can launch replacement instances in the **remaining healthy AZs** — so overall desired capacity can still be maintained, just redistributed.

The one config choice that determines survival: **whether Min (2) is high enough to be met using only the surviving AZs, and whether those AZs have subnet capacity to absorb the shifted load.** If, say, both of the 2 minimum instances happened to be in the AZ that failed, there's a brief window with **zero running instances** until the ASG detects the failure and launches replacements elsewhere — meaning the app's actual availability during an AZ outage depends on instances already being _spread across_ AZs at the moment of failure, not just configured to span multiple AZs in theory. This is exactly why ELB health checks (next lesson) matter — they detect failure faster and more accurately than basic EC2 status checks alone.

---

### Elastic Load Balancing (ELB)

Picture a busy restaurant with multiple kitchens in the back, all capable of cooking the same menu. Instead of customers walking randomly into whichever kitchen door they see first (some kitchens get slammed, others sit idle), there's a host at the front who looks at which kitchen has capacity right now and sends the order there. If a kitchen catches fire, the host stops sending orders to it entirely until it's fixed. That host is the Load Balancer — it doesn't cook anything itself, it just intelligently distributes incoming orders (requests) across the kitchens (instances) that can handle them.

### Definition

**Elastic Load Balancing (ELB)** is a managed service that distributes incoming application traffic across multiple targets (EC2 instances, containers, IP addresses, Lambda functions) in one or more Availability Zones, and continuously performs **health checks** to route traffic only to healthy targets.
Three types, each for a different layer of traffic:

-   **Application Load Balancer (ALB)** — operates at Layer 7 (HTTP/HTTPS). Understands the content of requests — can route based on URL path (`/api` → one target group, `/images` → another), hostname, or headers. This is the default choice for modern web apps and microservices.
- 
-   **Network Load Balancer (NLB)** — operates at Layer 4 (TCP/UDP). Doesn't inspect request content — routes purely on IP/port, at extremely high throughput and low latency. Used for use cases needing raw speed or non-HTTP protocols (e.g., gaming servers, financial trading systems).
- 
-   **Gateway Load Balancer (GWLB)** — used for deploying and scaling third-party virtual network appliances (firewalls, intrusion detection) transparently in front of traffic — a more specialized, less commonly tested type.

ELB is what an ASG plugs into to actually receive and distribute traffic — the ASG decides _how many_ instances exist, the ELB decides _which requests go to which instance_. They're complementary, not overlapping.

### The industry problem this solves

A company runs an e-commerce app across 6 EC2 instances behind Auto Scaling for redundancy and capacity. Without a load balancer, how does a customer's browser know which of the 6 instances to hit? Hardcoding one instance's IP defeats the whole purpose of having 6 — if that one instance goes down, the site appears down to anyone using that IP, even though 5 healthy instances are sitting idle.

ELB gives customers **one stable DNS endpoint** to hit, and behind that single endpoint, it silently spreads traffic across all healthy instances, pulls unhealthy ones out of rotation the moment a health check fails, and adds newly-launched ASG instances into rotation automatically as they come up. Users never know or care how many instances exist behind it — that's the whole point of "elastic."

#### CODE 

```
# Create an Application Load Balancer
aws elbv2 create-load-balancer \
  --name my-app-alb \
  --subnets subnet-abc123 subnet-def456 \
  --security-groups sg-0123456 \
  --type application

# Create a target group (the pool of instances traffic gets routed to)
aws elbv2 create-target-group \
  --name my-app-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-0123456 \
  --health-check-path /health \
  --health-check-interval-seconds 30

# Create a listener — the rule for "traffic arriving on port 443 goes to this target group"
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/my-app-alb/... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:...:certificate/... \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/my-app-tg
```

The `--health-check-path /health` matters more than it looks — it means your app needs to expose an actual `/health` endpoint that returns 200 OK only when the app is genuinely ready to serve traffic (e.g., its database connection is live), not just "the server process is running." A shallow health check that only proves the process exists, not that it works, is a classic real-world bug.

### Interview questions with answers (leveled up)

1. **"A team switches their app from an ALB to an NLB for 'better performance' without checking anything else, and suddenly their path-based routing rules (`/api/*` vs `/static/*` going to different target groups) stop working. Why?"**  
→ NLB operates at Layer 4 (TCP) and has no visibility into HTTP-level content like URL paths or headers — it can only route based on IP/port. Path-based routing is a Layer 7 (HTTP-aware) feature exclusive to ALB. Swapping load balancer types isn't a drop-in performance upgrade; it changes what routing capabilities are even available, and the team needs to redesign their routing strategy (e.g., separate NLBs per service, or stay on ALB) rather than assume feature parity.

**"An ALB's target group shows all instances as 'healthy,' but users still report intermittent 502 errors. What are two things worth investigating beyond the health check status itself?"**  
→ (1) The health check interval/threshold might be too lenient — an instance can fail _between_ health check intervals and still show healthy until the next check catches it, meaning real users can hit a briefly-dead instance during that gap. (2) The issue could be at the instance's application layer itself (e.g., the app's connection pool to a downstream database is exhausted under load, timing out real requests) even though the shallow health check endpoint itself still returns 200 — reinforcing the earlier point that a health check is only as good as what it actually verifies.

**"Why does an ALB typically sit in public subnets while the EC2 instances behind it sit in private subnets — walk through the traffic flow and why this is more secure."** 
→ The ALB is the only component that needs a public-facing presence to accept internet traffic; it's purpose-built and hardened by AWS to be internet-exposed safely. The EC2 instances doing the actual app logic and holding data have zero need for direct internet exposure — placing them in private subnets means they're unreachable directly from the internet even if someone knows their IP, and all traffic _must_ pass through the ALB's security group rules and health-checked routing first. This is defense-in-depth: even a compromised ALB config doesn't automatically expose the instances, since the private subnet's own routing (no direct internet gateway route) is a second barrier.

#### Scenario question 

A company's ALB is configured with a target group containing instances across 2 AZs. During a deploy, all instances in **AZ-1** get replaced at once (old ones terminated, new ones launched) while AZ-2's instances are untouched. Users start reporting a short but noticeable burst of errors during the deploy window, even though AZ-2 was never touched and had "enough" capacity on paper. What's the most likely mechanism causing errors, and what would you change about the deployment approach?

The most likely mechanism: **replacing all of AZ-1's instances at once meant AZ-2 alone had to absorb 100% of live traffic during the transition — but new AZ-1 instances weren't instantly ready to take load either.** Two compounding effects:

1. **No warm-up grace period** — newly launched instances in AZ-1 might pass a basic health check before the app is _actually_ ready (JVM warm-up, connection pools initializing, caches cold), so the ALB starts routing to them prematurely and those early requests fail or are slow.
2. **No connection draining (deregistration delay)** — when the _old_ AZ-1 instances were terminated, if in-flight requests weren't allowed to finish first, those requests got cut off mid-response, producing errors even though "on paper" AZ-2 had capacity.

**What to change:** Don't replace an entire AZ's instances simultaneously. Use a **rolling deployment** (e.g., ASG **Instance Refresh** with a `MinHealthyPercentage` like 90%, replacing instances a few at a time across _both_ AZs rather than one AZ wholesale) combined with:

- A proper **target group health check grace period** so new instances aren't sent traffic until truly ready
- **Deregistration delay** configured on the target group so terminating instances finish in-flight requests before being pulled from rotation

This turns a "capacity cliff" into a gradual, zero-visible-impact transition.

---
### EBS (Elastic Block Store)

Think of an EC2 instance as a laptop you rent by the hour. The laptop itself is powerful, but when you shut it down and return it, everything on its internal hard drive is gone — that's EC2's default **instance store** behavior for ephemeral data. EBS is like plugging in an **external hard drive** to that rented laptop instead: the drive persists independently of the laptop, you can unplug it and plug it into a _different_ rented laptop, and if the laptop itself is returned/destroyed, your external drive and everything on it survives untouched.

### Definition

**EBS (Elastic Block Store)** is persistent, block-level storage designed to attach to a single EC2 instance at a time (like a virtual hard drive), independent of the instance's own lifecycle.

Key properties:

- **Persists independently of the instance** — stopping or terminating an EC2 instance doesn't automatically delete its attached EBS volume (unless you explicitly set "delete on termination")

- **AZ-locked** — an EBS volume lives in a specific Availability Zone and can only attach to instances in that _same_ AZ. To move data to another AZ, you must snapshot it first, then restore the snapshot into a volume in the target AZ.

- **Snapshots** — point-in-time, incremental backups of an EBS volume stored in S3 behind the scenes. Incremental means only _changed blocks_ since the last snapshot are saved, making repeated snapshots cheap and fast compared to a full copy each time.

- **Volume types** trade off IOPS, throughput, and cost:
    - `gp3` (General Purpose SSD) — the modern default for most workloads, balanced price/performance
    - `io2`/`io2 Block Express` (Provisioned IOPS SSD) — for high-performance databases needing guaranteed, consistent IOPS
    - `st1` (Throughput Optimized HDD) — cheap, for large sequential workloads like big data/log processing, not for random small reads
    - `sc1` (Cold HDD) — cheapest, for rarely-accessed data

### The industry problem this solves

A company runs a database on an EC2 instance. The instance itself occasionally needs to be resized (say, from `m5.large` to `m5.xlarge` for more CPU) or replaced after a hardware failure. If the database's data lived only on the instance's local ephemeral storage, resizing or replacing the instance would **wipe the entire database**.

With EBS: the database's data volume is a separate EBS volume attached to the instance. When you need to resize, you stop the instance, detach nothing (EBS survives instance stop), resize the instance type, restart — the EBS volume is still attached with all data intact. If the instance itself dies entirely, you can detach the EBS volume and attach it to a brand-new replacement instance, and the database is back online with zero data loss, because storage and compute lifecycle are decoupled.

### CODE

```
# Create an EBS volume in a specific AZ
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3

# Attach it to a running EC2 instance
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-0123456789abcdef0 \
  --device /dev/sdf

# Take a point-in-time snapshot (incremental, stored in S3 behind the scenes)
aws ec2 create-snapshot \
  --volume-id vol-0123456789abcdef0 \
  --description "Pre-migration backup"

# Restore a snapshot into a new volume in a DIFFERENT AZ (the only way to "move" EBS data across AZs)
aws ec2 create-volume \
  --availability-zone us-east-1b \
  --snapshot-id snap-0123456789abcdef0 \
  --volume-type gp3
```

Notice the last command — this is the actual mechanism for the "EBS is AZ-locked" rule from above. There's no "move volume to another AZ" API call; snapshot-then-restore-elsewhere is the only path, and it's a common exam trick question phrased as "how do you migrate data across AZs."

### Interview questions with answers

**"A team terminates an EC2 instance expecting to keep their data since 'EBS is persistent,' but the volume is gone afterward too. What happened?"**  
→ Every EBS volume has a **"Delete on Termination"** flag, and for the **root volume** specifically, this defaults to `true` unless explicitly changed — so terminating the instance deleted the root EBS volume along with it. "EBS is persistent" is true _in principle_, but that persistence is only guaranteed if the delete-on-termination setting was explicitly turned off for volumes you intend to survive instance termination; non-root/additional volumes default to `false`, but the root volume's default catches people off guard.

**"Why can't you just detach an EBS volume from an instance in `us-east-1a` and attach it directly to an instance in `us-east-1b`?"**  
→ EBS volumes are physically tied to the Availability Zone's underlying storage infrastructure they were created in — they're not a "file" that can move freely, they're a replicated block storage volume bound to that AZ's hardware. Cross-AZ attachment isn't supported at all; the only path is to create a snapshot (which _is_ stored in S3, a regional/multi-AZ service) and restore a new volume from that snapshot in the target AZ — which is effectively a copy, not a move, of the underlying data.

**"Your team provisions an `io2` volume for a database expecting top-tier performance, but a teammate says the _instance type_ also matters, not just the volume type. Are they right, and why?"**  
→ Yes — an EC2 instance has its own **EBS-optimized bandwidth limit** (how much total throughput it can push to/from all attached EBS volumes combined), independent of any single volume's provisioned IOPS. Attaching a high-performance `io2` volume to an instance type with a low EBS bandwidth ceiling means the _instance_ becomes the bottleneck, not the volume — you'd be paying for IOPS the instance physically can't deliver to the application. Matching volume performance tier to an instance type that can actually utilize it (or scaling up the instance) is necessary, not optional.

### Scenario question 

A company's finance database runs on EC2 with a `gp3` EBS volume. One morning, the volume's performance suddenly degrades badly, but CloudWatch shows the instance's CPU is low and there's no unusual traffic. A teammate checks and finds the application has been doing far more random small-write operations than usual due to a recent code change. What's the most likely EBS-specific explanation for the slowdown, and what would you check or change?

The most likely EBS-specific cause is **running out of IOPS (or throughput) the `gp3` volume is provisioned for.**
A `gp3` volume comes with a baseline of **3,000 IOPS and 125 MB/s** regardless of size. Anything above that has to be provisioned (and paid for) separately. A code change that suddenly generates far more random small writes pushes the IOPS demand past that ceiling. Once that happens, I/O requests queue up and latency spikes, even though CPU is idle and traffic looks normal. The CPU is low precisely _because_ it’s sitting around waiting on storage.

**What I’d check:**

- CloudWatch EBS metrics: `VolumeQueueLength` (a persistently high queue means the volume can’t keep up), `VolumeReadOps`/`VolumeWriteOps` against the provisioned IOPS, and `VolumeThroughputPercentage`/`BurstBalance`-style indicators depending on the volume type.

**What I’d change:**

- Raise the provisioned IOPS/throughput on the `gp3` volume (it can be modified live with no downtime via `modify-volume`), or
- Move to `io2`/`io2 Block Express` if the workload needs guaranteed, consistent high IOPS (typical for a finance database), and
- Confirm the instance’s EBS-optimized bandwidth limit isn’t the next bottleneck (from yesterday’s interview question).

Note that `gp2` is the classic version of this trap. It uses a burst-credit model, and once credits are exhausted performance collapses to baseline. `gp3` removed credits in favor of a fixed baseline, but the “demand exceeded provisioned performance” principle is the same.

---

### Lesson 22: EFS (Elastic File System)

EBS (last lesson) was an external hard drive plugged into **one** laptop. Now imagine instead a **shared network drive in an office**: dozens of employees’ laptops, all connected at the same time, all reading and writing the same folders. Someone saves a file from their desk and a colleague across the floor sees it instantly. The drive also grows automatically as people add files, so nobody ever has to ask IT to “increase the size.” That’s EFS.

#### Definition

**EFS (Elastic File System)** is a fully managed, **shared, elastic file storage** service that uses the **NFS protocol** and can be mounted by **many EC2 instances simultaneously**, across **multiple AZs**.

Key properties:

- **Shared access:** thousands of instances can mount the same file system at once (contrast with EBS, which attaches to one instance at a time in one AZ).

- **Multi-AZ by design:** data is stored redundantly across multiple AZs in a region. Instances in different AZs all see the same files.

- **Elastic:** no capacity provisioning. It grows and shrinks automatically as you add or delete files, and you pay for what you actually store.

- **Linux-oriented (POSIX):** EFS is built for Linux workloads. For Windows file shares, the equivalent is **Amazon FSx for Windows File Server**, a favorite exam distinction.

- **Storage classes and lifecycle management:** EFS Standard for frequently accessed files, **EFS Infrequent Access (IA)** for rarely accessed ones at lower cost, and a lifecycle policy can automatically move files that haven’t been touched for N day

**Quick comparison of the three storage types:**

![[Pasted image 20261012120107.png|581]]

#### The industry problem this solves

A media company runs a **content management system (CMS)** on an Auto Scaling group of 6 web servers behind a load balancer (Lessons 19 and 20). Editors upload images and documents through the site. With EBS, each instance has its **own private disk**, so an upload that lands on Server A isn’t visible on Server B. The next page load, routed by the ELB to Server B, shows a broken image. You’d have to build custom syncing logic between servers, a fragile and slow mess.

With EFS: every web server mounts the **same shared file system**. An upload written by Server A is instantly readable by Servers B through F, and when the ASG launches a brand-new instance during a traffic spike, it mounts the same EFS and immediately sees all existing content with no copying step. Shared state lives in EFS while the instances stay disposable.

#### CODE 

```
# Create an EFS file system
aws efs create-file-system \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --tags Key=Name,Value=shared-cms-files

# Create a mount target in each AZ's subnet (how instances in that AZ reach EFS)
aws efs create-mount-target \
  --file-system-id fs-0123456789abcdef0 \
  --subnet-id subnet-abc123 \
  --security-groups sg-0123456

# On each EC2 instance: install the helper and mount it
sudo yum install -y amazon-efs-utils
sudo mkdir /mnt/shared
sudo mount -t efs fs-0123456789abcdef0:/ /mnt/shared
```

the **mount target** step. EFS isn’t reached by “attaching” like EBS. Each AZ gets a mount target (a network endpoint inside your VPC), and a **security group on that mount target must allow NFS traffic (port 2049)** from your instances. A missing rule there is the most common real-world reason a mount hangs and times out.

PORT- a numbered “door” on a server for one kind of traffic (80 = web, 443 = secure web, 2049 = NFS, 3306 = MySQL)

#### Interview questions with answers (leveled up)

**“A team mounts EFS on their EC2 instances and it works perfectly in `us-east-1a`, but instances launched by their ASG in `us-east-1b` hang forever when trying to mount. Same file system ID, same command. What’s the most likely cause?”**

→ A mount target must exist **in each AZ** where instances need to reach EFS (instances reach EFS through the mount target in their own AZ’s subnet). If a mount target was only created in `us-east-1a`, or the security group on the `us-east-1b` mount target doesn’t allow inbound NFS (TCP 2049) from those instances, the mount request in 1b has nowhere to land and times out. The file system itself is multi-AZ, but network access to it is configured per-AZ.

**“Why would you NOT use EFS as the storage for a high-performance transactional database, even though it’s shared and durable?”**  

→ EFS is network-based file storage, so every operation has network latency, and it’s designed for shared file access, not the ultra-low-latency, high-IOPS(input/output operations per second, i.e. how many reads/writes a disk handles each second) random block access databases need. A database on EFS would face higher latency and weaker performance consistency than on EBS (especially `io2`). Also, most databases are single-writer by design, so EFS’s main advantage (many concurrent mounts) goes unused. Match the tool to the access pattern: databases on EBS, shared file content on EFS.

**“A company’s Windows-based application servers need a shared file system. A junior engineer proposes EFS. What’s wrong, and what’s the right service?”**  

→ EFS speaks **NFS**(the protocol (set of rules) a computer uses to mount a shared folder over a network) and is built for **Linux (POSIX)** clients, so it doesn’t natively support Windows SMB file shares or Windows-specific features like NTFS permissions and Active Directory integration. The right service is **Amazon FSx for Windows File Server**, which provides native SMB shares with AD integration, essentially the managed equivalent of a Windows file server.

#### Scenario question

A company stores 20 TB of data on EFS Standard. Cost review shows the bill is much higher than expected, but an analysis shows roughly 85% of those files haven’t been accessed in over 90 days. Meanwhile, a different team proposes moving the whole thing to S3 to save money. What’s the better first step using EFS’s own features, and what would have to change in the applications if you _did_ migrate to S3 instead?

**Better first step: turn on EFS lifecycle management.** With 85% of the files untouched for 90+ days, a lifecycle policy that moves files not accessed for 90 days (the period is configurable) into **EFS Infrequent Access (IA)** is the lowest-risk fix. IA costs a fraction of Standard per GB. In exchange, you pay a small fee each time an IA file is read, which is fine for files that are almost never read. Applications see no difference: same mount, same file paths, same commands. Only about 3 TB stays on Standard.

**If you to S3 instead, the applications must change.** EFS is a _file system_: apps open, append to, rename, and lock files at paths like `/mnt/shared/report.pdf`, as if it were a local disk. S3 is _object storage_: apps call an API to upload or download whole objects (PUT/GET), and there’s no mounting it like a disk, no editing part of a file in place, and no file locking. Every piece of code that touches those files would need rewriting to use the S3 SDK, and permissions move from file permissions to IAM and bucket policies. S3 may be cheaper per GB, but that rewrite cost is why lifecycle-to-IA comes first, and S3 makes sense only if the data is write-once, read-rarely and you’re willing to change the apps.

---

### VPC (Virtual Private Cloud)

AWS is a giant city shared by millions of customers. A **VPC** is your own **fenced private estate** inside that city. Nobody else can walk in, and you decide the layout:

- **Subnets** are separate buildings on your estate.
- The **Internet Gateway** is the main gate to the public road. If a building has no path to that gate, it’s effectively indoors-only.
- **Route tables** are the signposts that say “traffic going to the internet, take this path.”
- **Security groups** are bouncers at each room’s door. They check who may enter that specific server.

#### Definition

A **VPC** is a logically isolated virtual network that you define inside an AWS Region. You choose its private IP address range, split it into subnets, and control what can talk to what.

Key building blocks:

- **CIDR block**: the range of private IP addresses your VPC owns, written like `10.0.0.0/16`. The `/16` says how big the range is (about 65,000 addresses). A bigger number after the slash means a _smaller_ range.

- **Subnet**: a slice of the VPC’s range that lives in **one single AZ**. A subnet can’t span AZs, which is why Lesson 19’s ASG listed several subnets (one per AZ) for fault tolerance.
    - **Public subnet**: its route table has a route to the Internet Gateway.
    - **Private subnet**: no direct route to the internet. Instances here can’t be reached from outside.

- **Internet Gateway (IGW)**: the door connecting your VPC to the public internet.

- **Route table**: a list of rules, “traffic for destination X goes to target Y,” attached to a subnet. **A subnet is public only if its route table points to an IGW.** The name or tag doesn’t matter.

- **NAT Gateway**: lets instances in a _private_ subnet make outbound requests (e.g., download software updates) without letting the internet start connections _to_ them. It sits in a public subnet.

- **Security group**: a firewall attached to an instance (or load balancer). It contains _allow_ rules only, and it’s **stateful**: if inbound traffic is allowed, the reply is automatically allowed back out.

- **Network ACL (NACL)**: a firewall at the _subnet_ boundary. It supports allow _and_ deny rules, and it’s **stateless**: replies need their own explicit rule.

- **Safe access into private resources**: a **bastion host** (a small, hardened server in a public subnet that you connect through) or a **VPN**

This is the picture behind Lesson, the ALB sits in public subnets, and the app servers sit in private subnets. and EFS mount targets were network endpoints placed in your subnets, which is why they needed a security group allowing port 2049.

#### The industry problem this solves

A company runs a web app and a customer database. Without network isolation, the database would sit on an address reachable by anyone on the internet, protected only by a password. One weak password or an unpatched bug, and customer data is exposed.

With a VPC, the team builds three layers: a **public subnet** (load balancer only), a **private subnet** for app servers, and another **private subnet** for the database. The database’s security group allows port 3306 **only from the app servers’ security group**. Even a leaked database password is useless to an attacker on the internet, because there’s no network path to the database at all.

#### CODE

```
# 1. Create the VPC (your private network, ~65k addresses)
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# 2. Create a subnet in one AZ (256 addresses)
aws ec2 create-subnet \
  --vpc-id vpc-0123456 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# 3. Create an Internet Gateway and attach it to the VPC
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-0123456 --vpc-id vpc-0123456

# 4. Create a route table, add a route "all internet traffic (0.0.0.0/0) -> IGW"
aws ec2 create-route-table --vpc-id vpc-0123456
aws ec2 create-route --route-table-id rtb-0123456 \
  --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0123456

# 5. Associate the route table with the subnet: THIS step makes the subnet public
aws ec2 associate-route-table --route-table-id rtb-0123456 --subnet-id subnet-0123456
```

`0.0.0.0/0` means “any address on the internet.” Steps 4 and 5 are the whole difference between a public and a private subnet: the same subnet without that route is private.

#### Interview questions with answers (leveled up)

**“Instances in a private subnet need to download OS patches from the internet, but must never be reachable _from_ the internet. How do you set this up, and why not just route them to the Internet Gateway?”**  
→ Put a **NAT Gateway** in a _public_ subnet and add a route in the _private_ subnet’s route table sending `0.0.0.0/0` to the NAT. Instances can start outbound connections, but the internet can’t start connections to them. Routing to the IGW directly would make the subnet public, and with a public IP the instances would be reachable from outside.

**“A team’s EC2 instance is in a subnet they tagged ‘public,’ has a security group allowing port 80 from anywhere, yet the site is unreachable from the internet. Name three things to check.”**  
→ (1) The subnet’s **route table** must have a route to an **Internet Gateway** (the tag means nothing). (2) The IGW must be **attached** to the VPC. (3) The instance needs a **public IP or Elastic IP**, since a private IP alone isn’t reachable from outside. Also check the NACL isn’t denying the traffic.

**“A security group allows inbound port 443, and nobody wrote an outbound rule for the replies, yet HTTPS works. Why? Would the same setup work with a NACL?”** 
→ Security groups are **stateful**: if inbound traffic is allowed, the response is automatically allowed out. A NACL is **stateless**, so replies are treated as new traffic and need their own outbound rule (for the high-numbered “ephemeral” ports the client uses, 1024-65535). With only an inbound rule, a NACL would block the replies and the connection would fail.

#### Scenario question

A company launched its MySQL database in a **public subnet** “so developers can connect from home,” and the database’s security group allows port 3306 from `0.0.0.0/0`. A security reviewer flags this as critical. What exactly is wrong, and how would you redesign the network so developers can still reach the database safely?

**What’s wrong:** two separate problems stack on top of each other.

1. **The database is in a public subnet.** That subnet’s route table points to the Internet Gateway, so the database sits on a network path that anyone on the internet can reach.
2. **The security group allows port 3306 from `0.0.0.0/0`** (“any address in the world”). Port 3306 is MySQL’s door, and the rule leaves it open to everyone. Bots scan the internet for open database ports constantly, so the only thing standing between attackers and customer data is a password.

**The redesign:**

- Move the database into a **private subnet** (no route to the IGW, no public IP).
- Change its security group so port 3306 is allowed **only from the app servers’ security group**, not from IP ranges.
- For developers working from home, give them a controlled way in:
    - **Bastion host**: a small, hardened server in a public subnet. Its security group allows SSH only from known IP ranges (e.g., the company’s), and developers tunnel through it to reach the database.
    - **VPN**: a private encrypted connection into the VPC, so developers’ laptops behave as if they’re inside the network. This is the cleaner long-term option.

Now even a leaked password is useless without a network path to the database.

---

### RDS (Relational Database Service)

Running your own database on EC2 is like owning a **restaurant and also being the building’s maintenance crew**: you cook (manage the data) _and_ fix the plumbing, replace burnt-out lights, and do the nightly security checks. **RDS** is leasing space in a managed building: the landlord handles plumbing, power, and maintenance (hardware, operating system, patching, backups, failover), and you just run the restaurant.

#### Definition 

**RDS** is a managed service for running relational databases. You choose an **engine**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, or Amazon’s own **Aurora** (its own lesson later). AWS runs the servers and database software, and you manage your data and who can reach it.

**DNS**: the internet’s phone book, translating a name (`mydb.abc.us-east-1.rds.amazonaws.com`) into the current IP address behind it.

Connect this to **Lesson 4 (Shared Responsibility)**: with RDS, AWS secures and patches the hardware, operating system, and database software. You’re responsible for your data, schema (table design), queries, user access, and the security group. You **cannot SSH into an RDS server**; the OS is AWS’s job.

Key features:

- **Automated backups**: daily snapshots plus continuous transaction logs (a running record of every change), kept 1-35 days. This enables **point-in-time recovery**: restore to any second within the retention window.

- **Manual snapshots**: backups you trigger yourself; they stay until you delete them.

- **Multi-AZ**: AWS keeps a **standby copy** of your database in a _different AZ_, updated synchronously (every write is copied before it’s confirmed). If the primary fails, RDS **automatically fails over**: it flips the DNS name to point at the standby, typically within a minute or two. This is for **availability**, not speed. In the standard setup, the standby isn’t used for queries.

- **Read Replicas**: extra copies updated **asynchronously** (a small delay) that your apps can query for **reads**. They offload read-heavy traffic from the primary for **performance**. They can live in another Region and can be **promoted** to become standalone databases.

- **DB subnet group**: the set of subnets (in at least 2 AZs) where RDS is allowed to place your instances.

- **Storage autoscaling** and **encryption at rest** (using AWS KMS, the key management service) are also available.

**The two most-confused features:** Multi-AZ = survive failure (availability). Read Replica = handle more reads (performance).

#### The industry problem this solves

A company self-manages MySQL on EC2. Their one database admin must schedule backups, test restores, apply security patches, and write failover scripts. At 3 a.m. the server dies, someone is paged, and the app is down for an hour while the database is rebuilt from backup.

With RDS Multi-AZ, the standby takes over automatically in about a minute or two, patching and backups are handled by AWS, and the team spends its time on the application instead.

#### CODE

```
# 1. Tell RDS which subnets it may use (private subnets in 2 AZs, from Lesson 23)
aws rds create-db-subnet-group \
  --db-subnet-group-name my-db-subnets \
  --db-subnet-group-description "Private subnets for DB" \
  --subnet-ids subnet-aaa111 subnet-bbb222

# 2. Create a Multi-AZ MySQL database, private, with 7 days of backups
aws rds create-db-instance \
  --db-instance-identifier my-store-db \
  --engine mysql \
  --db-instance-class db.t3.micro \
  --allocated-storage 20 \
  --master-username admin \
  --manage-master-user-password \
  --db-subnet-group-name my-db-subnets \
  --vpc-security-group-ids sg-0123456 \
  --multi-az \
  --no-publicly-accessible \
  --backup-retention-period 7 \
  --storage-encrypted

# 3. Add a Read Replica to offload read traffic
aws rds create-db-instance-read-replica \
  --db-instance-identifier my-store-db-replica \
  --source-db-instance-identifier my-store-db
```

`--multi-az` adds the failover standby, `--no-publicly-accessible` keeps it private, and `--manage-master-user-password` lets AWS store the password securely instead of you typing it into scripts.

#### Interview questions with answers

**“A team enables Multi-AZ to fix slow queries during peak hours. Nothing improves. Why, and what should they do?”**  
→ In the standard Multi-AZ setup the standby exists only for failover and doesn’t serve queries, so it adds no capacity. Multi-AZ solves _availability_, not _performance_. For read-heavy slowness, add **Read Replicas** and send read queries there, or use a larger instance class or add caching.

**“At 2:14 p.m. a developer runs a `DELETE` with no filter and wipes a table. Backups run nightly at 2 a.m. Can you recover data as of 2:13 p.m.?”**  
→ Yes, via **point-in-time recovery** (automated backups plus transaction logs, within the retention window). One catch: the restore creates a **new database instance** with a new endpoint. It doesn’t undo the damage in place, so you either repoint the app to the restored instance or copy the lost data back.

**“After a Multi-AZ failover, an application keeps failing because it was configured with the primary’s IP address. What’s the lesson?”**  
→ During failover, RDS updates the **DNS name** (the endpoint) to point at the standby; the old IP stops being the primary. Apps must always connect using the **endpoint hostname**, never a hardcoded IP, so they follow the failover automatically.

#### Scenario question

A company’s RDS database runs in **us-east-1** (Multi-AZ). They now have many users in Europe who need fast reads, and leadership also wants a plan in case the **entire us-east-1 Region** goes down. Which RDS features would you use for each goal, and what are the limits of that plan (what could still go wrong)?

**Goal 1: fast reads for European users.** Create a **cross-Region Read Replica** in a European Region (e.g., Ireland or Frankfurt). European apps send their read queries to the replica’s endpoint, so reads travel a short distance instead of crossing the Atlantic. The replica is read-only, so all **writes** still go to the primary in us-east-1.

**Goal 2: surviving the loss of the entire us-east-1 Region.** Use that same replica as your disaster plan: if us-east-1 goes down, you **promote** the replica to a standalone, writable database and point the application at it. Copying snapshots/backups to another Region is a good extra safety net.

**The limits of this plan:**

- **Possible data loss.** Replication is **asynchronous** (a small delay), so the last few seconds or minutes of writes may not have reached Europe when the Region fails. The “how much data can we lose” number is called the **RPO** (recovery point objective), and here it’s above zero.

- **Not automatic.** Unlike Multi-AZ failover, promoting a cross-Region replica is a manual step (or one you script). Downtime lasts from detection through the decision, the promotion, and repointing the apps. That downtime number is the **RTO** (recovery time objective).

- **Multi-AZ doesn’t help here.** Its AZs are all inside the same Region (Lesson 8), so a Region-wide failure takes out both the primary and the standby.

- **Stale reads and latency.** European reads can be slightly behind the primary, and European _writes_ still travel to the US until a failover happens.

- **Cost.** You pay for the extra instance and cross-Region data transfer.

---

### DynamoDB

RDS is a **well-organized filing cabinet**: linked folders, and you can ask complex questions like “all customers in Delhi who ordered twice last month.” DynamoDB is a **giant coat check**. You hand over your ticket number (the key) and get your exact coat back instantly. It takes the same time whether the room holds 100 coats or 100 million. But if you ask “show me every red coat,” the attendant has to check every coat, unless you planned for that question in advance.

### Definition

**DynamoDB** is a fully managed, **serverless** NoSQL database. It stores **key-value and document data** and delivers consistently fast responses (single-digit milliseconds) at almost any scale.

Key points:

- **Serverless**: no instances to size, patch, or place in subnets. Unlike RDS, you don’t choose a server class. You create a **table** and call it through an API, with access controlled by **IAM** (Lesson 5), not security groups.

- **Built-in resilience**: data is automatically replicated across multiple AZs in the Region.

- **Primary key**: every item needs one. It’s either a **partition key** alone, or a partition key plus a **sort key**. The partition key decides where the item is stored.

- **Flexible schema**: apart from the key, each item can have different attributes.

- **Two capacity modes**:
    - **On-demand**: you pay per request and DynamoDB absorbs spikes. Best for unpredictable traffic.
    - **Provisioned**: you set read and write capacity in advance, optionally with auto scaling. Cheaper for steady, predictable traffic.

- **Global secondary index (GSI)**: a second way to look up the same data by a different key.

- **DynamoDB Streams**: a time-ordered log of changes to items. It can trigger a **Lambda** function (Lesson 17) whenever data changes.

- **DAX**: an in-memory cache for DynamoDB, serving reads in microseconds.

- **Global Tables**: replicate a table across multiple Regions, with writes accepted in each Region. Replication is asynchronous, and if the same item is written in two Regions at once, **the last write wins**.

- **TTL (time to live)**: automatically deletes items after an expiry time, useful for sessions.

- **Backups**: on-demand backups and point-in-time recovery. Encryption at rest is on by default.

**Limits to remember:** an item can be at most **400 KB**; there are **no joins**; reads are eventually consistent by default, with a **strongly consistent** option that costs more.

#### The industry problem this solves

An online store runs a **flash sale**. Its shopping-cart service runs on one RDS database. When traffic jumps 20x, the database runs out of connections and slows down, and someone has to resize it before the next sale. A cart is a simple lookup: “get the cart for user 4821.” It doesn’t need joins.

On DynamoDB, with the table keyed on `userId`, each lookup stays fast whether 1,000 or 10 million shoppers arrive. With on-demand mode there is no capacity planning, and the team pays per request rather than for idle servers.

#### CODE

```
# 1. Create a table: partition key = userId, sort key = productId
aws dynamodb create-table \
  --table-name Carts \
  --attribute-definitions AttributeName=userId,AttributeType=S AttributeName=productId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH AttributeName=productId,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST

# 2. Add an item (S = string, N = number)
aws dynamodb put-item --table-name Carts \
  --item '{"userId":{"S":"4821"},"productId":{"S":"p-100"},"qty":{"N":"2"}}'

# 3. Fast lookup by exact key
aws dynamodb get-item --table-name Carts \
  --key '{"userId":{"S":"4821"},"productId":{"S":"p-100"}}'

# 4. Query: everything in one user's cart (efficient, uses the partition key)
aws dynamodb query --table-name Carts \
  --key-condition-expression "userId = :u" \
  --expression-attribute-values '{":u":{"S":"4821"}}'
```

`HASH` is the partition key and `RANGE` is the sort key. `PAY_PER_REQUEST` is on-demand mode. A **Query** goes straight to one user’s items. A **Scan** (reading the whole table) is the slow, expensive operation to avoid.

#### Interview questions with answers (leveled up)

**“A team migrates an orders database from RDS to DynamoDB ‘for scale.’ Now the report ‘all orders from last week across every customer’ is slow and expensive. Why?”**  
→ DynamoDB is built for **key-based access patterns**. If the table is keyed by customer, a report across all customers has no efficient path and falls back to a **Scan** of the whole table, which reads (and bills for) every item. The fix is to design around access patterns up front: add a **GSI** keyed on order date, or export the data to an analytics store for reporting. DynamoDB asks you to know your questions _before_ you design the table, unlike RDS.

**“Total table capacity is plentiful, yet one very popular product’s page gets throttled. What’s happening?”**  
→ A **hot partition**. Items with the same partition key live together, and capacity is spread per partition, so a flood of requests for one key can overwhelm its partition even when the table overall is idle. Fixes: choose a key with better spread, split the hot key (for example by adding a random suffix), or put **DAX** in front to absorb repeated reads.

**“A user updates their profile, then immediately reloads and sees the old value. Is DynamoDB broken?”**  
→ No. Reads are **eventually consistent** by default: for a brief moment after a write, a read may hit a copy that hasn’t caught up. If the app needs the latest value, request a **strongly consistent read** for that call. It costs more read capacity and has slightly higher latency, so use it only where freshness matters.

#### Scenario question

A mobile game stores player sessions and leaderboards in RDS MySQL. At a new-season launch, traffic jumps 50x: the database runs out of connections and latency soars. The company also has players in three continents who need fast access. Which DynamoDB features would you use for the scale problem and the global problem, and what trade-offs or risks come with that design?

**For the scale problem (50x traffic, connections maxing out):**

- Move player sessions and leaderboard entries to **DynamoDB in on-demand mode**. There are no connections to exhaust, because apps call it through an API. On-demand absorbs a sudden 50x spike with no capacity planning.
- Key the sessions table on `playerId` so every session lookup is a fast key-based read.
- For the leaderboard, don’t scan the table to rank players. Add a **GSI** (a second key, e.g. `seasonId` as partition key and `score` as sort key) so “top players this season” is an efficient query.
- Put **DAX** (the in-memory cache) in front for the hottest reads, such as the global top 100 that thousands of players refresh constantly. It also protects against a hot partition.
- Use **TTL** to expire old sessions automatically.

**For the global problem (players on three continents):**

- Use **Global Tables**, so each Region has its own replica that accepts both reads and writes. Players connect to the nearest Region and get low latency.

**Trade-offs and risks:**

- **Conflicting writes:** replication between Regions is asynchronous, and if the same item is updated in two Regions at nearly the same moment, **the last write wins**. A player’s score could be silently overwritten. Design so conflicts are rare (for example, route each player to a “home” Region) or so writes are safe to merge.

- **Eventual consistency:** a player may briefly see a slightly stale leaderboard, especially across Regions. That’s fine for rankings, but not for something like a purchase balance.

- **Access patterns must be designed up front:** no joins and no flexible ad-hoc queries. A new question like “players who bought item X” needs a new index or an export to an analytics system.

- **Hot keys:** a celebrity player or a “global” leaderboard key can still overload one partition. Spread the key (for example, shard it) or cache with DAX.

- **Cost:** Global Tables multiply write costs across Regions, and on-demand is pricier than provisioned for steady traffic. Once the launch spike settles, consider switching to provisioned with auto scaling.

---

