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
## ARN (Amazon Resource Name)

An **ARN is the unique, full address of any resource in AWS**. Names like `orders-queue` can repeat across accounts and Regions, so AWS needs an unambiguous identifier, and the ARN is it. IAM policies, SNS subscriptions, Lambda triggers and cross-service permissions all refer to resources by ARN.

A real example, from the SQS lesson:

```
arn:aws:sqs:ap-south-1:123456789012:orders-queue
```

| Part       | Value here     | Meaning                                                                             |
| ---------- | -------------- | ----------------------------------------------------------------------------------- |
| `arn`      | `arn`          | Always starts with this                                                             |
| partition  | `aws`          | Normally `aws`. Special ones exist for China (`aws-cn`) and GovCloud (`aws-us-gov`) |
| service    | `sqs`          | Which AWS service                                                                   |
| region     | `ap-south-1`   | Which Region (Mumbai)                                                               |
| account-id | `123456789012` | The 12-digit AWS account that owns it                                               |
| resource   | `orders-queue` | The resource itself (its format varies by service)                                  |
**Some variations to know:**

- **Global services leave the Region blank.** IAM and S3 are examples:
    - An S3 bucket: `arn:aws:s3:::my-bucket` (no Region and no account ID, because bucket names are globally unique).
    - An S3 object: `arn:aws:s3:::my-bucket/reports/report.pdf`.
    - An IAM role: `arn:aws:iam::123456789012:role/RedshiftS3ReadRole` (no Region).
- **The resource part can have a type prefix**, like `role/...`, `instance/...`, or `function:my-function`.

**Where ARNs matter in practice:**

1. **IAM policies** say which resources an action applies to:

```json
   "Resource": "arn:aws:sqs:ap-south-1:123456789012:orders-queue"
```

A `*` wildcard can cover many resources (`arn:aws:s3:::my-bucket/*` means every object in that bucket).  
2. **Linking services together**: when you subscribed the SQS queue to the SNS topic, you passed the topic’s ARN and the queue’s ARN.  
3. **Conditions** like `aws:SourceArn`, which we used in the CloudFront and SQS policies to say “only this specific distribution or topic may do this”. That is a security best practice, because it stops a different resource, possibly in another account, from using the permission.  
4. **Finding the ARN**: it’s shown on the resource’s page in the console, and CLI `create-*` or `describe-*` commands return it.

**A quick way to remember it:** the **name** is what you call the resource, and the **ARN** is its full postal address, including country (partition), city (Region) and house owner (account).

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

### Amazon Aurora

Think of a **normal RDS database** as a shop where each branch keeps its own full copy of the ledger. When the head office records a sale, it must copy that entry to every branch’s ledger. If a branch is slow, everyone waits. If a branch burns down, you rebuild its ledger from backups.

**Aurora** is the same chain with **one shared, fireproof vault** that holds the ledger. The branches (database servers) are just counters that read from and write to that same vault. The vault keeps six copies of every page, spread across three buildings. Adding a counter takes minutes, because nothing needs to be copied. If a counter dies, another one starts serving the same vault.
#### Definition

**Amazon Aurora** is AWS’s own relational database engine. It is **compatible with MySQL and PostgreSQL**, so your existing code and drivers work unchanged. It is part of the RDS family, but its storage layer was redesigned for the cloud.

Key facts:

- **Storage is separated from compute.** The database servers (instances) are one layer, and a distributed storage layer is another.
- **Six copies of your data across 3 AZs** (two per AZ). Writes need 4 of 6 copies to acknowledge, and reads need 3 of 6. This is why Aurora can lose a whole AZ and still write.
- **Storage auto-grows** in 10 GB chunks up to 128 TiB. You never pre-provision it.
- **One writer instance plus up to 15 Aurora Replicas**, all reading the same shared storage. Replica lag is typically milliseconds, because replicas don’t copy data, they only read it.
- **Failover is fast**, usually around 30 seconds or less. Aurora promotes a replica to writer.
- AWS markets it as up to **5x faster than standard MySQL** and **3x faster than standard PostgreSQL**, at about **1/10th the cost of commercial databases** (Oracle, SQL Server). Treat these as AWS claims.

#### The endpoints

| Endpoint                      | Points to                         | Use it for                                                              |
| ----------------------------- | --------------------------------- | ----------------------------------------------------------------------- |
| **Cluster (writer) endpoint** | Current writer                    | All writes. After a failover it automatically points to the new writer. |
| **Reader endpoint**           | Load-balances across all replicas | Read-heavy traffic (reports, dashboards, search)                        |
| **Custom endpoint**           | A subset you choose               | Sending heavy analytics to big replicas only                            |
| **Instance endpoint**         | One specific instance             | Debugging and special cases (avoid in app code)                         |
#### Aurora variants

- **Aurora Serverless v2**: capacity scales up and down automatically in small increments, and you pay for what you use. Good for unpredictable or spiky workloads, dev/test, and new apps whose load is unknown.

- **Aurora Global Database**: one primary Region plus up to 5 read-only secondary Regions, with storage-level replication that usually lags under a second. Used for disaster recovery (a promoted secondary Region can take over within about a minute) and for low-latency local reads worldwide.

- **Backtrack** (MySQL-compatible only): rewind the database to an earlier point in time **in place, without restoring from backup**.

- **Aurora Parallel Query, Cloning and Backups**: cloning creates a fast, space-efficient copy of a big database using copy-on-write. Continuous backups go to S3 with point-in-time restore.

#### The industry problem it solves

**Example: a food-delivery or ticketing platform.**

Traffic is normal most of the day, then spikes 20x during dinner rush or a ticket drop. On standard RDS you face three pains:

1. **Read replicas lag.** Each one has to replay every write, so under heavy load customers see stale order status.
2. **Failover is slower.** A Multi-AZ promotion on standard RDS typically takes 60-120 seconds, which is a visible outage during dinner rush.
3. **Scaling storage and replicas is operationally heavy.** You pre-provision storage, and every new replica needs a full data copy first.

With Aurora, replicas read the shared storage, so lag is tiny. Failover is quick, storage grows by itself, and you can add replicas in minutes. For a company running **Oracle or SQL Server licenses costing millions**, migrating to Aurora PostgreSQL is a standard cost-cutting move, and AWS provides the Schema Conversion Tool and Database Migration Service for it.

**Where it is a poor fit:** simple, small, predictable workloads where plain RDS is cheaper, and non-relational data (use DynamoDB).

#### CODE

```
# 1. Create the cluster (the shared storage + the endpoints)
aws rds create-db-cluster \
  --db-cluster-identifier shop-aurora \
  --engine aurora-postgresql \
  --master-username admin_user \
  --manage-master-user-password \
  --storage-encrypted

# 2. Add the writer instance (the first instance in a cluster becomes the writer)
aws rds create-db-instance \
  --db-instance-identifier shop-aurora-writer \
  --db-cluster-identifier shop-aurora \
  --engine aurora-postgresql \
  --db-instance-class db.r6g.large

# 3. Add a reader (an Aurora Replica)
aws rds create-db-instance \
  --db-instance-identifier shop-aurora-reader-1 \
  --db-cluster-identifier shop-aurora \
  --engine aurora-postgresql \
  --db-instance-class db.r6g.large
```

- Step 1 creates the **cluster**. This is the key idea: in Aurora you create a _cluster_ first (storage and endpoints), then add _instances_ into it. In plain RDS, the instance is the database.

- `--manage-master-user-password` makes AWS store the password in **Secrets Manager** (a later lesson). You never write the password anywhere.

- `--storage-encrypted` encrypts the data on disk with KMS (Domain 2).

- Steps 2 and 3 add compute. The first instance becomes the writer, and any later one becomes a reader.

#### How the application uses it (Python):

```
import psycopg2

# Writes go to the cluster (writer) endpoint
writer = psycopg2.connect(host="shop-aurora.cluster-xxxx.ap-south-1.rds.amazonaws.com", ...)

# Reads go to the reader endpoint (note the "cluster-ro-")
reader = psycopg2.connect(host="shop-aurora.cluster-ro-xxxx.ap-south-1.rds.amazonaws.com", ...)

with writer.cursor() as cur:
    cur.execute("INSERT INTO orders (user_id, total) VALUES (%s, %s)", (42, 499))
writer.commit()

with reader.cursor() as cur:
    cur.execute("SELECT status FROM orders WHERE user_id = %s", (42,))
```

Because of the split, the writer stays free for transactions and the readers absorb the read load. If the writer fails over, the `cluster-` hostname follows the new writer and your code doesn’t change.

#### Interview question answers

**Q1. “Aurora replicas copy data from the writer, just like RDS read replicas.”**

That statement is wrong. A standard RDS read replica has its **own copy of the data** and must **replay every write** that the primary makes, so it falls behind when write load is heavy. An Aurora replica has **no copy of its own**. It reads the **same shared storage volume** as the writer. When the writer commits a change, the change is already in storage that the replica reads from. The replica only has to refresh its in-memory cache.

This is why Aurora replica lag is typically in **milliseconds**, and why adding a replica takes minutes: no data has to be copied first.

**Q2. The AZ holding the writer goes down. What happens?**

- **(a) Data:** Nothing is lost. Aurora keeps **6 copies across 3 AZs** (2 per AZ). Losing one AZ removes 2 copies, and 4 remain. Writes need 4 of 6 and reads need 3 of 6, so the storage layer can still write and read.
- **(b) Writes:** The writer instance is gone, so Aurora **promotes one of the replicas** (in a surviving AZ) to be the new writer. Writes are interrupted for roughly **30 seconds or less**, then resume. Replicas have a failover priority (tier), and you can set which one is preferred.
- **(c) Connection string:** It doesn’t change, if the app uses the **cluster (writer) endpoint**. That DNS name automatically points to the new writer. The app might see a short burst of connection errors, so it should have **retry logic** and short DNS caching.

A subtlety worth mentioning: if you had **no replicas at all**, Aurora would need to create a new instance, which is much slower. That’s why production clusters should have at least one replica in a different AZ.

**Q3. The hard-coded instance endpoint of the writer.**

An **instance endpoint** points to **one specific instance**, not to “whoever is the writer”. After the failover, the old writer instance either is down or has become a **reader** (read-only). The app keeps connecting to it and every write fails with a read-only error.

They should have used the **cluster (writer) endpoint**, which always follows the current writer. Instance endpoints are for debugging and special cases only.

**Q4. Analytics team running huge queries without hurting production reads.**

Use a **custom endpoint**. Steps:

1. Add one or two **larger replicas** (for example `db.r6g.4xlarge`) to the cluster.
2. Create a custom endpoint that includes **only those replicas**.
3. Give the analytics team that endpoint. Production traffic continues to use the standard reader endpoint, which maps to the other replicas.

Their heavy queries now land on dedicated replicas, so production reads are unaffected, and there’s no second database to maintain, because they all read the same storage. For extremely heavy analytical workloads, **Aurora Parallel Query** can also push query processing down into the storage layer.

**Q5. `DELETE FROM customers` with no WHERE clause, 10 minutes ago.**

Two options:

1. **Backtrack** (MySQL-compatible only): rewind the **existing cluster** to a point just before the mistake. It happens **in place**, takes minutes, and doesn’t create a new cluster. This is the faster option. The caveat is that it rewinds the **whole database**, so any legitimate writes made in the last 10 minutes are also undone. It has to be enabled on the cluster before the incident happens, and it has a limited window.
2. **Point-in-time restore (PITR)** from the continuous backup: this creates a **new cluster** at the chosen time. It’s slower, because you then have to switch your application to the new cluster endpoint or copy the lost rows back. Its advantage is that the original cluster stays untouched, so you can export just the deleted rows and merge them back without losing the last 10 minutes of other changes.

For a surgical fix, a good alternative is to **create a clone** from before the delete (or restore with PITR), copy the `customers` table back into production, and delete the clone.

---

### Scenario answer: the ticketing company

**Problem 1: browsing traffic (90% reads) slows the site.**

Move to **Aurora (MySQL-compatible, to keep their existing code)** with **multiple Aurora Replicas** (up to 15). Send all browsing queries (event pages, search, listings) to the **reader endpoint**, which load-balances across the replicas. Only the booking and payment steps use the **writer endpoint**. Since replicas read shared storage, there’s almost no lag, so event availability stays fresh. For extra relief on the hottest data (event details), a cache like **ElastiCache** in front of the database would help. We’ll cover this in a later lesson, so for now it’s just a pointer.

**Problem 2: over a minute of downtime when the primary crashed.**

Aurora failover is typically **about 30 seconds or less**, compared with 60 to 120 seconds on standard RDS Multi-AZ. Place the replicas in **different AZs** and set the **failover priority** so a well-sized replica is promoted first. Apps connect via the **cluster endpoint**, so no connection string change is needed, and they should have retry logic. For the sale, a replica of the **same size as the writer** is important, because a smaller replica that gets promoted would become the bottleneck.

**Problem 3: European users are slow, and the company wants to survive a Region outage.**

Use **Aurora Global Database**:

- Primary Region (for example, Mumbai) takes all writes.
- A secondary Region (for example, Frankfurt) holds a **read-only copy** with typical replication lag **under a second**. European users read from their local Region, which fixes latency for browsing.
- If the primary Region fails entirely, you **promote the secondary Region** to be the new primary, which typically takes about a minute. That covers the **Region-outage** requirement.

An honest limitation: European users’ **writes** (purchases) still go to the primary Region, unless you use write forwarding, which has some limitations. For a ticketing site this is usually acceptable, since most traffic is browsing.

**Serverless v2 or provisioned for sale day?**

I’d choose **provisioned instances** for the **writer and the main replicas**, sized in advance for the expected peak. The reasons:

- The sale day is a **known, scheduled** event, so you can plan capacity (and pre-scale replicas before the sale starts). Predictable load is what provisioned instances are good at.
- A sudden 20x spike can outrun automatic scaling in the first moments, while a pre-sized provisioned instance is already warm and ready.
- Running at steady high load, provisioned (especially with **Reserved Instances** for the baseline) is usually cheaper than Serverless.

A sensible hybrid is to use **Serverless v2 replicas for the extra, unpredictable headroom** beyond what you sized, since they scale up and down on their own and you pay only while they’re busy. Serverless v2 would be my first choice for **dev/test environments** and for apps with **unpredictable or spiky** load, which this one isn’t on sale day.

---

### Amazon Redshift

**Aurora is a cashier** at a supermarket: serves one customer at a time, very fast, thousands of times a day.

**Redshift is the head-office analyst** who must answer: _“What was our total revenue per product category, per state, for the last 3 years?”_ A cashier can’t do that, because it needs scanning every receipt ever printed.

To make the analyst fast, the company does two things:

1. It splits all receipts across **many analysts working in parallel**, each handling a slice (this is **massively parallel processing**).
2. It files receipts **by column** rather than by receipt: all the “price” values together, all the “date” values together. To compute total revenue, the analyst reads only the price pile and ignores everything else (this is **columnar storage**).

#### Definition

**Amazon Redshift** is AWS’s fully managed **data warehouse**. It’s a database designed for **OLAP**: _online analytical processing_, meaning complex queries that scan and aggregate huge amounts of data for reports, dashboards and business intelligence. It speaks **SQL** and is based on PostgreSQL, but it is not a drop-in PostgreSQL replacement for transactional apps.

#### Why it’s fast: three ideas

1. **Columnar storage**: data is stored by column. A query touching 3 of 100 columns reads only those 3, so it reads far less data from disk. Similar values sit together, so they also compress very well.
2. **MPP (massively parallel processing)**: the cluster has one **leader node** and several **compute nodes**. The leader receives your SQL, builds a plan and distributes the work. Each compute node processes its own slice of the data in parallel, then results are combined.
3. **Compression and zone maps**: Redshift compresses columns automatically and skips blocks of data that can’t match your filter.

#### Key terms

- **Cluster**: leader node plus compute nodes.
- **Leader node**: the front door. Your tools connect here, and it plans the query.
- **Compute nodes**: store the data and do the heavy work.
- **Redshift Serverless**: no cluster to size. You run queries and pay for the compute used (measured in RPUs, Redshift Processing Units), and it scales automatically. Good for unpredictable or occasional analytics.
- **Redshift Spectrum**: query data that stays in **S3** directly, without loading it into Redshift first. This is handy for huge, rarely-queried data (a “data lake” approach).
- **COPY command**: the standard way to bulk-load data from S3 into Redshift. It loads in parallel and is far faster than inserting row by row.
- **Distribution style / sort key**: choices about how rows are spread across nodes and in which order they’re stored. Good choices make joins and filters much faster. This is tuning, which the CCP exam won’t test deeply but interviews do.
- **Snapshots**: backups stored in S3, which can be copied to another Region for disaster recovery.
- **Concurrency Scaling**: temporarily adds capacity when many users query at the same time.
- **Data sharing**: share live data between Redshift clusters without copying it.

#### Honest limitations

- It is **not** for transactions. Single-row inserts and updates are slow compared with Aurora.
- Redshift clusters are mostly **single-AZ**, with **Multi-AZ** available for certain node types (RA3), so check the current option when designing for high availability.

#### The industry problem it solves

**Example: an e-commerce company** with 5 years of orders, clicks and returns.

The company’s order database (Aurora) holds the live data. The analytics team runs a monthly query: _“revenue and return rate per product category, per city, with year-over-year comparison.”_ On Aurora this query:

- scans hundreds of millions of rows,
- **locks up resources and slows down the live shop**,
- takes hours.

The standard fix is a **data warehouse pipeline**:

1. The app writes to **Aurora** (OLTP).
2. A nightly job copies the data into **S3**, then into **Redshift** via `COPY`. (A managed way to build these pipelines is **AWS Glue**, and a way to move database data is **DMS**, the Database Migration Service.)
3. Analysts and dashboards (Tableau, QuickSight, Power BI) query **Redshift**, so the live shop is never touched.

The same query now runs in seconds to minutes. This separation of “operational database” and “analytics database” is one of the most important patterns in data engineering.

#### CODE

```
-- 1. Create a table for sales data
CREATE TABLE sales (
    order_id    BIGINT,
    product     VARCHAR(100),
    category    VARCHAR(50),
    city        VARCHAR(50),
    amount      DECIMAL(10,2),
    order_date  DATE
)
DISTSTYLE AUTO
SORTKEY (order_date);

-- 2. Bulk-load CSV files from S3 using COPY
COPY sales
FROM 's3://my-shop-data/sales/2026/'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftS3ReadRole'
FORMAT AS CSV
IGNOREHEADER 1
REGION 'ap-south-1';

-- 3. An analytical query
SELECT category, city, SUM(amount) AS revenue
FROM sales
WHERE order_date >= '2026-01-01'
GROUP BY category, city
ORDER BY revenue DESC;
```

- `SORTKEY (order_date)` stores rows ordered by date, so the `WHERE order_date >= ...` filter can skip large blocks of data.
- `DISTSTYLE AUTO` lets Redshift pick how to spread rows across nodes, which is a safe default.
- `COPY` reads all files under that S3 prefix **in parallel**. It uses an **IAM role** so no access keys appear in your SQL. The role must allow reading that bucket.
- The final query only needs the `category`, `city`, `amount` and `order_date` columns, so columnar storage helps.

Query S3 directly with Spectrum (SQL):

```
CREATE EXTERNAL SCHEMA lake
FROM DATA CATALOG DATABASE 'shop_lake'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftS3ReadRole';

SELECT COUNT(*) FROM lake.old_clickstream WHERE year = 2022;
```

The old data never moves into Redshift. It stays in cheap S3 and is queried when needed.

#### Interview questions with answers

**Q1. A teammate proposes replacing the company’s Aurora order database with Redshift “because it’s faster.” What’s wrong with this idea?**

**Answer:** “Faster” depends on the workload. Redshift is fast at **scanning and aggregating huge datasets**, but slow at **many small, single-row reads and writes**, which is what an order system does. Columnar storage means updating or inserting one row touches many column blocks. Redshift also has weaker transactional concurrency for that pattern. The right design uses **both**: Aurora for live transactions, Redshift for analytics, with a pipeline between them.

**Q2. A query that touches 3 columns of a 200-column, 10-billion-row table is running quickly in Redshift, but the same query on a row-store database is very slow. Explain exactly why.**

**Answer:** A row store keeps each full row together on disk, so reading 3 columns still drags all 200 columns of every row from disk into memory, which is roughly 60 times more data than needed. Redshift’s columnar layout reads **only the 3 columns’ blocks**. Those blocks also **compress heavily**, since values in one column are similar, which reduces disk reads further. On top of that, **MPP** splits the work across nodes, and **zone maps** skip blocks that can’t match the filter.

**Q3. Loading 500 GB of data row by row with `INSERT` statements takes days. What should be done instead, and why is it faster?**

**Answer:** Put the data into **S3** (split into multiple files, ideally compressed, and ideally around equal-size files) and use the **`COPY` command**. `COPY` loads files **in parallel across all compute slices**, so each slice reads its own files simultaneously. Single-row `INSERT`s go through the leader node one at a time and don’t use that parallelism. File count matters: a multiple of the number of slices gives the best balance, and a single huge file loads on just one slice.

**Q4. The company has 10 years of clickstream logs (200 TB) that are queried only a few times a year. Loading everything into Redshift would be very expensive. What’s your approach?**

**Answer:** Keep the data in **S3** (in a columnar format like **Parquet**, ideally partitioned by date) and query it with **Redshift Spectrum**. Frequently used, recent data (say, the last 90 days) can live inside Redshift for speed, while old data stays in cheap storage. Pay for Spectrum scans only when queries run. You can also move old S3 data to cheaper storage classes with lifecycle policies, keeping in mind that Glacier-class data isn’t directly queryable.

**Q5. Your Redshift dashboards are fast all day, but at 9 AM when 200 analysts log in at once, queries queue up and slow down. Which Redshift feature addresses it, and what’s another approach?**

**Answer:** **Concurrency Scaling** adds temporary extra clusters to handle bursts of concurrent queries, then removes them. This suits a predictable 9 AM spike. Another approach is **Redshift Serverless**, which scales compute automatically with demand, or using **workload management (WLM)** queues to give dashboards priority over heavy ad-hoc queries. For repeated identical dashboard queries, **materialized views** can precompute the results.

#### Scenario question

**Scenario:** You are the cloud engineer at a streaming-video company.

- Viewing events (play, pause, stop, search) arrive at roughly **2 TB per day** and are stored as files in **S3**.
- The product team wants a **daily dashboard** of watch time by genre, country and device, and the finance team wants a **monthly report** that joins viewing data with subscription data from the **Aurora** database.
- The data scientists occasionally need to query the **last 4 years** of raw events, but only a few times a year.
- Right now, the analysts run these queries directly on the Aurora database, and the live app slows down whenever they do.
- The CTO wants this fixed and wants costs controlled.

QUESTION - 

1. Where do the analytics queries run instead, and why?

On **Redshift**, not Aurora. The workload is OLAP: scans and aggregations over billions of rows (watch time by genre, country and device). Aurora is built for OLTP and stores rows together, so these queries read far more data than needed and compete with the live app for resources. Redshift’s **columnar storage and MPP** are built for exactly this, and moving analytics there **isolates the live app** from analyst queries. This also fixes the CTO’s main complaint.

2. How does data get from Aurora and S3 into Redshift?

- **From S3 (viewing events):** use the **`COPY` command** to load recent data in parallel. Store the files as **compressed Parquet or split CSV files**, and schedule the load daily (for example with an orchestrated job or AWS Glue). Load only the **new day’s partition** each time, not everything.
- **From Aurora (subscription data):** the finance report needs subscription data next to viewing data. Use **AWS DMS (Database Migration Service)** for ongoing replication, or a nightly export to S3 followed by `COPY`. Subscription tables are small compared with 2 TB/day of events, so a daily refresh is usually enough.
- Once both are in Redshift, the monthly finance report is a normal **SQL `JOIN`** inside Redshift, with zero load on Aurora.

3. How do you handle the 4-year-old raw data without overspending?

Four years at 2 TB/day is roughly **2,900 TB (about 2.9 PB)** of raw data. Loading all of it into Redshift would be very expensive when it’s queried only a few times a year. So:

- Keep **recent data** (for example, the last 60 to 90 days, which powers the daily dashboard) **inside Redshift** for speed.
- Keep **older raw events in S3** as **partitioned Parquet** (partitioned by date), queried through **Redshift Spectrum** when the data scientists need it. Parquet plus partitioning means Spectrum scans only the date ranges and columns a query needs, which keeps both time and cost down.
- Apply **S3 lifecycle policies** (Lesson 7) to move very old data to cheaper storage classes. One caution: data in Glacier-type classes can’t be queried directly, so choose a class (like Standard-IA or Glacier Instant Retrieval) that still allows direct reads, or restore the data before a query.

 4. Redshift provisioned or Serverless?

I’d choose **Redshift Serverless**, with a **usage limit** to cap spending, for these reasons:

- The workload is **uneven**: a daily dashboard refresh, a monthly finance report and occasional data-science runs. Serverless suits intermittent workloads because compute scales automatically and you pay only for the compute used while queries run.
- No cluster sizing or node-count guessing is needed, which reduces operational effort.
- Cost control is built in through **RPU usage limits**.

**When provisioned would win instead:** if the dashboard were queried **constantly all day** with a **steady, predictable load**, a provisioned cluster with **Reserved Nodes** could become cheaper than paying per-use. So the honest answer is: start Serverless, monitor actual usage, and switch if the load turns out to be steady and heavy.

---

### Amazon Route 53

- **IP address**: the numeric address of a server on the internet, like `203.0.113.25`. Computers find each other with these.
- **Domain name**: a human-friendly name, like `example.com`. People remember names, not numbers.
- **DNS (Domain Name System)**: the internet’s phonebook. It translates a domain name into an IP address. When you type `example.com`, your device asks DNS “what’s the IP address?”, gets the answer, then connects.
- **ELB (Lesson 20)** gives you a load balancer with its own long, ugly DNS name (like `my-alb-123456.ap-south-1.elb.amazonaws.com`). You don’t want customers typing that.
- **Region and AZ : locations of AWS infrastructure. Health and location matter for DNS routing decisions.
- **VPC** : your private network in AWS, which can also have private names.

DNS is a **phonebook**, and **Route 53 is a smart receptionist**.

A normal phonebook says “Rahul → 98xxxxx”. A smart receptionist does more: “Rahul is on holiday, so I’ll connect you to his backup.” “You’re calling from Europe, so I’ll connect you to our Frankfurt office.” “Send 10% of callers to the new office to test it out.” Route 53 does exactly this for internet traffic, using the **health** of your servers, the **location** of the caller, and **weights** you choose.

### Definition

**Amazon Route 53** is AWS’s **highly available and scalable DNS service**. It does three things:

1. **Domain registration**: you can buy and manage domain names (like `mycompany.com`).
2. **DNS routing**: it answers “what IP or resource does this name point to?”
3. **Health checking**: it monitors your endpoints and stops sending traffic to unhealthy ones.

The name “53” comes from **port 53**, the port DNS uses.
#### Key terms

- **Hosted zone**: a container for the DNS records of one domain. A **public hosted zone** answers queries from the internet. A **private hosted zone** answers only inside your VPC(s), for internal names like `db.internal.mycompany.com`.

- **Record**: one entry in the phonebook. Common types:
    - **A**: name → IPv4 address.
    - **AAAA**: name → IPv6 address.
    - **CNAME**: name → another name (not allowed at the root domain, like `example.com` itself).
    - **Alias**: a Route 53 special record that points a name to an **AWS resource** (ELB, CloudFront, S3 website, API Gateway, etc.). It works at the root domain, is **free** for queries to AWS resources, and automatically follows the resource’s changing IPs. Prefer Alias over CNAME for AWS targets.
    - **MX**: mail servers. **TXT**: text data, often for domain verification.

- **TTL (time to live)**: how long (in seconds) other systems may cache an answer. Low TTL means changes spread quickly but with more queries. High TTL means fewer queries but slow changes.

- **Health check**: Route 53 periodically probes an endpoint (HTTP, HTTPS or TCP). If it fails, Route 53 stops returning that endpoint in answers (when the record is tied to the health check).

#### Routing policies 

| Policy                | What it does                                                                       | Typical use                                             |
| --------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **Simple**            | One record, one or more values returned                                            | A single web server                                     |
| **Weighted**          | Splits traffic by percentage                                                       | Canary release: 5% to the new version, 95% to the old   |
| **Latency-based**     | Sends users to the Region with the lowest latency                                  | Multi-Region app for global users                       |
| **Failover**          | Primary and secondary; switches to secondary when the primary’s health check fails | Disaster recovery (active-passive)                      |
| **Geolocation**       | Routes by the user’s country or continent                                          | Legal or language requirements, content restrictions    |
| **Geoproximity**      | Routes by geographic distance, with an adjustable bias                             | Shift more load toward one Region                       |
| **Multivalue answer** | Returns up to 8 healthy records at random                                          | Simple client-side load distribution with health checks |
| **IP-based**          | Routes by the client’s IP range (CIDR)                                             | Send specific ISPs or networks to specific endpoints    |
**Key distinction :** **Latency** routing optimizes for _speed_. **Geolocation** routes by _where the user is_, regardless of speed (for compliance or language). They are different goals.

#### Other things worth knowing

- Route 53 is a **global service** (not tied to one Region), and AWS states a **100% availability SLA** for it.
- It can do **DNS-level failover only**. It does not forward traffic itself. It just answers DNS queries, and the client then connects directly to the endpoint.
- **Route 53 Resolver** handles DNS queries between your VPC and your on-premises network (hybrid setups).

### The industry problem it solves

**Example: a food-delivery company** running its app in two Regions (Mumbai and Singapore), behind load balancers.

Problems without Route 53:

1. Customers must reach `app.foodco.com`, not the long ELB hostname.
2. Customers in Southeast Asia should reach Singapore, and Indian customers should reach Mumbai (lower delay).
3. If the Mumbai Region has an outage at 8 PM on a Friday, traffic must **automatically** move to Singapore, without an engineer manually editing DNS.
4. For a new app version, the team wants to send 5% of users to it first, to catch bugs before everyone sees them.

Route 53 solves all four: an Alias record for friendly naming, **latency-based routing** for speed, **health checks plus failover** for resilience, and **weighted routing** for the gradual rollout.

#### CODE

```
# 1. Create a public hosted zone for your domain
aws route53 create-hosted-zone \
  --name foodco.com \
  --caller-reference "foodco-2026-10-13"

# 2. Create a health check for the Mumbai endpoint
aws route53 create-health-check \
  --caller-reference "mumbai-hc-1" \
  --health-check-config '{
      "Type": "HTTPS",
      "FullyQualifiedDomainName": "mumbai-alb-123.ap-south-1.elb.amazonaws.com",
      "Port": 443,
      "ResourcePath": "/health",
      "RequestInterval": 30,
      "FailureThreshold": 3
  }
```

Latency-based records with health checks (change batch, JSON):

```
{
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.foodco.com",
        "Type": "A",
        "SetIdentifier": "mumbai",
        "Region": "ap-south-1",
        "HealthCheckId": "<mumbai-health-check-id>",
        "AliasTarget": {
          "HostedZoneId": "<ALB hosted zone id for ap-south-1>",
          "DNSName": "mumbai-alb-123.ap-south-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    },
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.foodco.com",
        "Type": "A",
        "SetIdentifier": "singapore",
        "Region": "ap-southeast-1",
        "AliasTarget": {
          "HostedZoneId": "<ALB hosted zone id for ap-southeast-1>",
          "DNSName": "sg-alb-456.ap-southeast-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }
  ]
}
```

```
aws route53 change-resource-record-sets \
  --hosted-zone-id <your-hosted-zone-id> \
  --change-batch file://latency-records.json
```

**What this does:**

- Step 1 creates the **hosted zone** (the container for your domain’s records). It gives you four **name servers**, and you must set these at your domain registrar so the internet knows Route 53 is in charge.
- The health check probes `/health` every 30 seconds. After 3 failures it marks the endpoint unhealthy.
- Both records have the **same name** (`app.foodco.com`) and **different `SetIdentifier`s**, which is how Route 53 tells records of the same name apart in routing policies.
- `Region` makes this **latency-based**: a user is answered with the record for the Region that gives them the lowest latency.
- **Alias** records point at the load balancers, and `EvaluateTargetHealth: true` makes Route 53 skip a load balancer whose targets are all unhealthy.
- If Mumbai is unhealthy, Route 53 stops returning it, and everyone gets Singapore. No human action is needed.

**Weighted example (the 5% canary), as a concept:** two records for the same name, one with `Weight: 95` pointing to the stable version and one with `Weight: 5` pointing to the new version. Set the new one to `0` to turn it off, or raise it step by step.

#### Interview questions with answers

**Q1. Your site is `example.com` (the root domain, also called the zone apex), and you want it to point to your Application Load Balancer. A teammate creates a CNAME record and it’s rejected. Why, and what’s the fix?**

**Answer:** DNS rules **do not allow a CNAME at the root (zone apex)**, because the root already needs other record types (like NS and SOA) and a CNAME can’t coexist with them. The fix is a Route 53 **Alias record** (type A or AAAA with the alias flag) pointing to the ALB. Alias works at the root, is free for queries to AWS resources, and automatically tracks the ALB’s changing IP addresses.

**Q2. You changed an A record to point to a new server, but half your users still reach the old server an hour later. What’s going on, and how should you plan the next change?**

**Answer:** It’s **TTL caching**. Resolvers and browsers cached the old answer for the old record’s TTL (if it was set to, say, 24 hours, some users keep the old IP for up to a day). Planning ahead, **lower the TTL (for example to 60 seconds) well before the change**, wait for the old TTL to expire, make the change, then raise the TTL back afterward. Lowering TTL only helps once the old, longer TTL has expired everywhere.

**Q3. A customer in Germany is being routed to your Mumbai Region even though you have a Frankfurt Region, and the Frankfurt endpoint is healthy. You’re using latency-based routing. Is this a bug?**

**Answer:** Not necessarily. Latency-based routing uses **network latency measurements between the user’s resolver location and AWS Regions**, which is not the same as physical distance. Occasionally a path to another Region is measurably faster, due to how networks are connected. Also check that the Frankfurt record **actually exists with the right Region value** and that its **health check is passing**, because a failing health check removes it from answers. If the goal is “German users must always go to Frankfurt for data-residency reasons,” latency routing is the wrong tool. Use **geolocation routing**, which routes by user location regardless of speed.

**Q4. Explain the difference between geolocation and latency routing with an example where choosing the wrong one causes a real problem.**

**Answer:** **Latency** routing answers “which Region is fastest for this user?” **Geolocation** routing answers “where is this user, and what should users there get?” Example: a streaming service licensed to show a show only in India. Using latency routing, an Indian user may be served from Singapore (faster), which is fine for speed, but it **cannot enforce** the licensing rule, so a user in another country might reach the content. Geolocation lets you serve the licensed catalog only to Indian users and a different catalog elsewhere. You can also set a **default** geolocation record for locations that match no rule, otherwise those users get no answer.

**Q5. Your primary Region fails at night. Route 53 failover is configured with a health check, but the switch took 5 minutes. Name at least three reasons it wasn’t instant, and how you’d tighten it.**

**Answer:** The delay comes from stacked timers:

1. **Health check detection time**: the check interval (30 seconds, or 10 seconds for fast checks) times the failure threshold (for example 3) means detection can take 30 to 90 seconds.
2. **TTL caching**: resolvers keep serving the old answer until the TTL expires.
3. **Client and ISP caching**: some resolvers and apps cache longer than the TTL.

To tighten it: use **fast health checks (10-second interval)** with a lower failure threshold, set a **low TTL (such as 30 to 60 seconds)** on failover records, use **Alias records with `EvaluateTargetHealth`** (Alias answers can react to target health without waiting for a separate check), and consider pairing DNS failover with a layer that doesn’t depend on DNS caching at all, such as a global accelerator or CDN-level failover (we’ll cover CloudFront next). Also test failover regularly, since an untested failover plan often doesn’t work.

#### Scenario question

**Scenario:** You are the cloud engineer for an online education company.

- The main app runs in **Mumbai** (primary) and a standby copy runs in **Singapore**, both behind Application Load Balancers.
- The website is `learnco.com`, and the course videos site is `videos.learnco.com`.
- Last month, Mumbai had an outage for 40 minutes and the team had to **manually edit DNS** while students couldn’t log in.
- Students from the **UK** and **Gulf countries** complain the site is slow, so the company is adding a **London** Region.
- Legal says that **students in the EU must have their account data served only from the London Region**.
- For the next release, the team wants to **send 10% of traffic to the new version first**, then ramp up.
- Developers also want internal names like `db.learnco.internal` that only work **inside the VPC**, so nobody hard-codes private IPs.

Design the Route 53 setup. For each requirement, name the **record type or routing policy**

#### 1. Automatic recovery from the Mumbai outage

Use **failover routing with a health check**.

- Create a **health check** on the Mumbai ALB’s `/health` endpoint.
- Create two records for the same name: Mumbai as **PRIMARY** (tied to the health check) and Singapore as **SECONDARY**.
- When Mumbai fails its checks, Route 53 stops returning it and answers with Singapore. No human has to edit DNS, which is exactly what went wrong last month.
- Use **Alias records with `EvaluateTargetHealth`**, a **low TTL (30 to 60 seconds)** and a **fast (10-second) health check** to shorten detection and caching delays, as discussed in Q5.

#### 2. Fast access for UK and Gulf students

Use **latency-based routing**, adding a **London** record alongside Mumbai (and Singapore). UK users will usually be answered with London. Gulf users will be answered with whichever Region measures lowest latency for them, which could be Mumbai or London, and Route 53 decides that from network measurements, so you don’t have to guess. Each latency record gets its own health check so an unhealthy Region is skipped.

#### 3. The EU data rule

Use **geolocation routing**, not latency. This is a **legal requirement based on where the user is**, so speed must not decide it. Create a geolocation record for **Europe (or the specific EU countries)** pointing to London. Also create a **default** geolocation record for everyone else, because users who match no rule would otherwise get no answer.

One honest caveat: DNS routing only controls **which endpoint a user is sent to**. Real data-residency compliance also requires that EU data is **stored only in London** (database and backups) and that the application doesn’t replicate it elsewhere. Route 53 alone doesn’t guarantee that.

#### 4. The 10% gradual rollout

Use **weighted routing**: one record with weight **90** for the stable version and one with weight **10** for the new version. Each should have a health check. Ramp up in steps (10 → 25 → 50 → 100), and set the new version’s weight to **0** to roll back instantly (subject to TTL, so keep the TTL low during the rollout).

#### 5. Internal names for the database

Use a **private hosted zone** for `learnco.internal`, associated with the VPC(s) that need it, and create an **A record** (or an alias/CNAME to the database endpoint) named `db.learnco.internal`. These names resolve only inside the associated VPCs. This also means that when the database endpoint changes after a failover or rebuild, you update one record and nothing is hard-coded in application code. For Aurora, point to the **cluster endpoint** (Lesson 26), which already follows the writer.

#### 6. Does `learnco.com` need a CNAME?

No. The root (zone apex) **cannot** have a CNAME. Use an **Alias record (type A and AAAA)** pointing at the ALB. It’s free for queries to AWS resources and tracks IP changes. `videos.learnco.com` is a subdomain, so a CNAME would be allowed there, but an Alias is still better if it points to an AWS resource such as CloudFront (our next lesson).

##### Can two policies be combined on the same name?

Yes, through **nested records (a routing tree)**. You can’t attach two policies to a single record, but one record can point to another record set. Example for this scenario:

- At the top, a **geolocation** record: Europe → London, default → a “global” name.
- The “global” name is a **latency** record set that picks the fastest Region.
- Under each Region’s record, a **weighted** set handles the 90/10 rollout, and **failover** can sit below that.

Alias records are what make this chaining possible, since an Alias can point to another record in the same hosted zone. Keep trees shallow, because complexity makes debugging DNS harder.

---

### Amazon CloudFront

Think of a publisher with one big printing press (the **origin**, your S3 bucket or server in Mumbai).

Without a CDN, every reader worldwide orders directly from that press. A reader in Brazil waits for the book to be shipped across the world, every single time.

With **CloudFront**, the publisher sets up **local bookshops (edge locations)** in many cities. The first time someone in Brazil orders a book, the local shop fetches it from the press and **keeps a copy**. The next Brazilian customer is served from the shelf in minutes. The press is also less busy, because the shops absorb most of the demand.

### Definition

**Amazon CloudFront** is AWS’s **content delivery network (CDN)**. It caches copies of your content at **edge locations** around the world, and serves users from the nearest one. This lowers latency, reduces load on your origin and can lower data-transfer costs.

#### How a request flows

1. A user requests `https://cdn.example.com/logo.png`. DNS (Route 53) routes them to the **nearest CloudFront edge location**.
2. **Cache hit**: the edge already has `logo.png` (and it hasn’t expired), so it returns it immediately.
3. **Cache miss**: the edge fetches it from the **origin**, stores it in cache (per the TTL), and returns it to the user.
4. Later users near that edge get a cache hit.

There is also a **regional edge cache** layer between the edge locations and the origin, which holds content longer and reduces origin fetches.

#### Key terms

- **Distribution**: the CloudFront configuration object. It has its own domain name, like `d111111abcdef8.cloudfront.net`, and you can attach a custom domain.

- **Origin**: where the original content lives. It can be an **S3 bucket**, an **ALB/EC2**, API Gateway, or any HTTP server (even on-premises).

- **Edge location**: a CloudFront point of presence (hundreds worldwide).

- **Cache behavior**: rules that say “for paths matching `/images/*`, use origin A, cache for 1 day; for `/api/*`, use origin B and don’t cache”.

- **TTL**: how long an object stays cached. You can control it with cache headers from the origin and distribution settings.

- **Invalidation**: manually removes objects from the cache before their TTL expires (for example after you update a file). The first batch of paths each month is free, with charges after that. A better practice is **versioned file names** (like `app.v42.js`), which avoids the need to invalidate.

- **OAC (Origin Access Control)**: lets CloudFront access a **private S3 bucket**, so the bucket doesn’t have to be public, and users can reach files only through CloudFront. This replaces the older OAI (Origin Access Identity).

- **Signed URLs / signed cookies**: give time-limited access to private content (paid videos, subscriber downloads).

- **HTTPS**: CloudFront supports HTTPS with free certificates from **AWS Certificate Manager (ACM)**. For CloudFront, the certificate must be created in **us-east-1**.

- **AWS WAF and Shield**: attach **WAF** (web application firewall) to a distribution to filter malicious requests, and **Shield Standard** provides DDoS protection by default. Absorbing attacks at the edge protects your origin.

- **Lambda@Edge and CloudFront Functions**: run small pieces of code at the edge (rewrite URLs, add headers, redirect, simple auth). CloudFront Functions are lighter and cheaper but more limited.

- **Geo restriction**: allow or block countries.

- CloudFront isn’t only for static files. It can also accelerate **dynamic content**, because requests ride AWS’s private backbone network between the edge and the origin, and connections are reused.

- A CloudFront distribution is a **global service**, but its origin is in a specific Region.

CloudFront **caches content** at the edge. **Global Accelerator** gives you **static anycast IP addresses** and routes TCP/UDP traffic over the AWS network to your endpoints with fast failover, but it **doesn’t cache**. Choose CloudFront for cacheable web content, and Global Accelerator for non-HTTP traffic or when you need fixed IPs.

#### The industry problem it solves

**Example: a news and video website** hosted in Mumbai.

Without a CDN:

1. Readers in the US and Europe wait seconds for images and videos to cross the world.
2. During a breaking-news spike, millions of requests hit the origin servers, which can crash.
3. The company pays for a lot of outbound data transfer from the origin Region.
4. Premium video must only be viewable by paying subscribers, but if the S3 bucket is public, anyone with the link can share it.
5. Attackers send floods of junk requests directly at the origin.

CloudFront addresses these: users are served from nearby edges, the spike is absorbed by edge caches (the origin sees only the misses), the S3 bucket stays private behind **OAC**, premium content uses **signed URLs**, and **WAF/Shield** blocks bad traffic before it reaches the origin.

#### CODE

Create a private S3 bucket served through CloudFront (CLI, simplified):

```
# 1. Create a bucket (stays private) and upload a test file
aws s3 mb s3://learnco-static-demo-12345 --region ap-south-1
echo "<h1>Hello from the edge</h1>" > index.html
aws s3 cp index.html s3://learnco-static-demo-12345/

# 2. Create an Origin Access Control (OAC) for CloudFront
aws cloudfront create-origin-access-control \
  --origin-access-control-config '{
    "Name": "learnco-oac",
    "SigningProtocol": "sigv4",
    "SigningBehavior": "always",
    "OriginAccessControlOriginType": "s3"
  }'
```

```
# 3. Create the distribution (config file holds the origin, default behavior, and OAC id)
aws cloudfront create-distribution --distribution-config file://dist-config.json
```

**Bucket policy that lets only that CloudFront distribution read the bucket:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowCloudFrontServicePrincipalReadOnly",
    "Effect": "Allow",
    "Principal": { "Service": "cloudfront.amazonaws.com" },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::learnco-static-demo-12345/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::123456789012:distribution/EDFDVBD6EXAMPLE"
      }
    }
  }]
}
```

**Invalidate cached files after an update:**

```bash
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/index.html" "/css/*"
```

**What this does:**

- The bucket is **not public**. The **bucket policy** trusts only the CloudFront service acting on behalf of **your specific distribution** (the `SourceArn` condition), so nobody can bypass CloudFront with a direct S3 URL.
- The **OAC** makes CloudFront sign its requests to S3.
- Visiting `https://<distribution-domain>.cloudfront.net/index.html` returns your page from the nearest edge. The first request is a cache miss, and later ones are hits (check the `X-Cache` response header: `Miss from cloudfront` vs `Hit from cloudfront`).
- Invalidation clears the edge copies so users get the updated file before the TTL ends.

#### Interview questions  with answers

**Q1. You updated `style.css` on your origin, but users still see the old styles hours later. What’s happening, and what are your options to fix it now and to prevent it next time?**

**Answer:** The old file is **cached at the edge** and its **TTL hasn’t expired**. To fix now, run an **invalidation** for that path (`/style.css`). To prevent it, use **versioned file names** (such as `style.v43.css` or content hashes like `style.a1b2c3.css`) so each release is a new URL, and the old cached copy is irrelevant. This is faster, free of per-path invalidation costs and avoids any lag between edges. Also set sensible TTLs and keep `index.html` on a short TTL, since it references the versioned assets.

**Q2. A company hosts premium course videos in a private S3 bucket. A student shares a CloudFront link on social media and now everyone can watch the videos for free. How do you stop this?**

**Answer:** Make the distribution **require signed URLs or signed cookies**, so links work only with a valid signature and an **expiry time** (and optionally restricted to an IP range). The application generates a short-lived signed URL after verifying the student has paid. A shared link then stops working when it expires. Keep the S3 bucket private with **OAC** so there’s no way around CloudFront. Signed **cookies** are better when many files need access (a whole course), and signed **URLs** suit single files. For stronger protection against sharing, add short expiry times and consider DRM for high-value content, because a signed URL alone can still be shared during its valid window.

**Q3. Your origin is a dynamic API (an ALB), and responses differ per logged-in user. A developer enables caching with a long TTL on the API path, and users start seeing each other’s data. Explain the bug and the fix.**

**Answer:** The cache key didn’t include what distinguishes users, so CloudFront served **one user’s cached response to another**. That’s a serious data leak. Fixes: for personalized endpoints, set the cache behavior to **not cache** (TTL 0, or the caching-disabled policy), or include the **Authorization header or the session cookie in the cache key** so each user gets a separate cache entry (usually inefficient). Better still, cache only what is truly shared (public product lists, static assets) on separate paths, with `Cache-Control: private` or `no-store` set by the origin for personalized responses. CloudFront can still help here by accelerating the request over the AWS backbone, even with no caching.

**Q4. A teammate says “CloudFront is just for static files like images, so it’s no use for our dynamic website.” Do you agree?**

**Answer:** Only partly. It’s best at caching static and cacheable content, but it helps dynamic sites too. Requests travel from the edge to the origin over the **AWS global network**, which is usually faster and more stable than the public internet, and the edge keeps **persistent, reused connections** to the origin and terminates TLS close to the user, reducing handshake time. CloudFront also provides **WAF, Shield and edge functions**, so it protects the origin even when nothing is cached. And many “dynamic” pages have cacheable parts (public pages for a few seconds or minutes can absorb big spikes). For non-HTTP traffic or when fixed IPs are required, Global Accelerator is the better fit.

**Q5. Your S3 bucket is public so CloudFront can read it. A security reviewer objects. What’s the problem and what’s the right design?**

**Answer:** A public bucket lets anyone bypass CloudFront with a direct S3 URL, so signed URLs, WAF rules, geo restrictions and caching controls can all be circumvented, and it exposes data and increases costs. The right design: keep the bucket **private with Block Public Access on**, and use **Origin Access Control (OAC)** with a bucket policy that allows only the CloudFront service principal for the **specific distribution ARN**. If the bucket uses SSE-KMS encryption, the KMS key policy must also allow CloudFront to decrypt. Mention that OAC replaces the older **OAI**, which doesn’t support all newer features (like SSE-KMS and all Regions).

#### Scenario question

**Scenario:** You are the cloud engineer for a regional fashion e-commerce company, **StyleKart**, whose website is hosted in **Mumbai** (ALB and EC2 for the app, S3 for product images).

- A flash sale is planned. Last sale, product images loaded slowly for customers abroad, and the origin servers nearly crashed because every image request hit them.
- Product images are in an S3 bucket that is **currently public**.
- Customers log in, and the **cart and account pages** are personalized.
- **Invoices** (PDFs) are stored in S3 and each must be downloadable **only by its owner, for 10 minutes** after clicking “Download”.
- Bots have been hammering the site with junk requests.
- The marketing team updates product images and prices several times a day, and wants changes to appear quickly.

Design the CloudFront setup. For each point, name the feature

#### 1. Speeding up images and protecting the origin from the spike

Put a **CloudFront distribution** in front of everything, with **S3 as the origin for `/images/*`** and the **ALB as the origin for the app**, using separate **cache behaviors** per path.

- Images are cacheable, so edges around the world serve them after the first request (a cache miss). Customers abroad get nearby copies, which fixes the slow loading.
- During the flash sale, millions of image requests are absorbed by edge caches and the **regional edge cache**. The origin sees only misses, which protects it from the spike.
- Use a **long TTL** for images, combined with versioned file names (see point 6).
- Serving cacheable pages (like public product listing pages) with a **short TTL** (even 30 to 60 seconds) can also take significant pressure off the app servers during the sale.

#### 2. Securing the S3 buckets

- **Turn on Block Public Access** and make the image bucket **private**. The current public bucket lets anyone bypass CloudFront with direct S3 URLs, which defeats WAF, caching and cost control.
- Use **Origin Access Control (OAC)** and a bucket policy that allows only the CloudFront service principal, restricted with a **`SourceArn` condition for this specific distribution**.
- If the bucket uses SSE-KMS, update the **KMS key policy** so CloudFront can decrypt.
- Do the same for the invoices bucket (or serve both through the same distribution under different paths).

#### 3. Personalized cart and account pages

- **Do not cache** the cart and account pages, using a behavior for paths like `/cart/*` and `/account/*` with the **caching-disabled policy** (TTL 0), forwarding the cookies and headers the app needs. Caching them with a shared cache key would show one customer’s data to another, which is the leak we discussed in Q3.
- The origin should also send `Cache-Control: private, no-store` on personalized responses, as a second safety net.
- **Do cache** the genuinely shared content: images, CSS and JS, public product pages and category listings (short TTL).
- Requests to the uncached paths still benefit from the edge: TLS terminates near the user, connections to the origin are reused, and traffic travels over the AWS backbone.

#### 4. Delivering invoices

Keep the invoices bucket private and use **signed URLs** (or **S3 pre-signed URLs**, which are the S3 equivalent):

- When a logged-in customer clicks “Download”, the **application first checks that the invoice belongs to that user**.
- It then generates a **CloudFront signed URL with a 10-minute expiry**. After 10 minutes the link stops working, so sharing it is of limited use.
- A signed URL fits here because it’s one file per click. **Signed cookies** are better when a user needs access to many files (like a whole course).
- The signing private key must be kept safe (for example in **Secrets Manager**, which is coming up in our list), and CloudFront is set up with a trusted key group.

Honest caveat: within the 10-minute window, the signed URL could still be shared by its owner. You can narrow this by also restricting to the customer’s IP address, if that’s acceptable, though IP restriction can break for mobile users whose IP changes.

#### 5. Dealing with the bots

- Attach **AWS WAF** to the distribution. Use **rate-based rules** (block an IP making too many requests in 5 minutes), **AWS managed rule groups** (known bad inputs, bot control) and, if needed, CAPTCHA challenges on sensitive paths like login.
- **Shield Standard** is included automatically and covers common network and transport-layer DDoS attacks. Shield Advanced is optional for higher-risk workloads.
- Blocking at the edge means bad requests are rejected **before they reach the origin**, which also keeps the servers available for real customers during the sale.
- Make sure the ALB only accepts traffic from CloudFront, for example with a **custom secret header** that CloudFront adds and the ALB checks (or the managed prefix list for CloudFront in the ALB’s security group). Otherwise bots can bypass the edge and hit the ALB directly.

#### 6. Letting marketing see updates quickly without causing problems

- For images: use **versioned file names** (for example `shirt-red.v17.jpg` or a hash), so each update is a **new URL** and no cache clearing is needed. This is the preferred approach.
- For pages that reference those images (and for prices): keep a **short TTL** on the HTML and price data so changes appear within seconds to minutes, and use **invalidations** for urgent fixes (for example a wrong price), keeping paths specific rather than invalidating `/*` repeatedly, because invalidations beyond the free monthly allowance are charged.
- Better still, render prices **dynamically from an API (uncached or short TTL)** rather than baking them into cached images or pages, so a price change doesn’t depend on cache clearing at all.

#### 7. Which Region for the HTTPS certificate?

**us-east-1 (N. Virginia).** CloudFront is a global service, and it only accepts **ACM certificates created in us-east-1**. If you create the certificate in Mumbai, it won’t appear in the CloudFront certificate list. (The ALB in Mumbai needs its own certificate in **ap-south-1** if you want HTTPS between CloudFront and the ALB, which you should.) Also set the distribution’s **alternate domain name (CNAME)** to `www.stylekart.com` and point Route 53 to it with an **Alias record**.

---
## EXTRA - Boto3

**Boto3 is the official AWS SDK for Python.** An SDK (software development kit) is a library that lets your code talk to a service. Instead of writing raw HTTP requests to AWS and signing them yourself, you call Python functions, and boto3 handles the authentication, request signing, retries and response parsing.

Everything you can click in the AWS Console, or run with the AWS CLI, can also be done from Python:

```python
import boto3

# Create a "client" for a service
s3 = boto3.client("s3", region_name="ap-south-1")

# Call a function that maps to an AWS API action
s3.upload_file("report.pdf", "my-bucket", "reports/report.pdf")
```

**The pieces you’ll see in every lesson:**

- **`boto3.client("service")`** gives you a low-level interface where each method maps directly to an AWS API call (`sqs.send_message`, `sns.publish`). The responses are plain dictionaries.
- **`boto3.resource("service")`** gives a higher-level, object-style interface for some services (like S3 and DynamoDB). AWS has stopped adding new features to resources, so most new code uses clients.
- **Credentials**: you never hard-code keys in your script. Boto3 looks for credentials automatically, in this order of common sources: environment variables, the `~/.aws/credentials` file (created by `aws configure`), and, when your code runs on AWS itself (EC2, Lambda), the **IAM role** attached to that resource (Lesson 5). On AWS, always prefer the role.
- **Permissions still apply**: boto3 doesn’t bypass IAM. If your role isn’t allowed to call `sqs:SendMessage`, boto3 raises an `AccessDenied`-type error (`botocore.exceptions.ClientError`).

Install it with `pip install boto3`. The name comes from a freshwater dolphin in the Amazon river, which is the same naming spirit as the original library.

**Where you’ve seen it:** the SQS worker loop (`sqs.receive_message`, `sqs.delete_message`) and the SNS publish call in the last two lessons were boto3. The CLI commands in the other examples do the same job from the terminal.

---
### AWS Direct Connect

- **VPC** is your private network inside AWS. It has subnets, route tables and gateways. Traffic leaves it through an **Internet Gateway (IGW)** or a **NAT gateway**.
- **On-premises** means servers in your own office or data center, outside AWS.
- **Hybrid cloud (Lesson 2)** means running part of a workload on-premises and part in AWS.
- **Region and AZ (Lesson 8)**: AWS infrastructure is grouped into Regions made of AZs.
- **Latency** is the delay of a request. **Bandwidth** is how much data can flow per second. 
- **Encryption in transit**: scrambling data while it travels, so interceptors can’t read it. **VPN** (virtual private network) is an encrypted tunnel, usually over the public internet.
- **Route 53 (Lesson 28)** has a Resolver that helps on-premises and VPC networks resolve each other’s names, which matters in hybrid setups.

You have a head office (your **data center**) and a warehouse (your **AWS VPC**), and trucks carry goods between them every day.

- **Site-to-Site VPN** is sending the trucks over the **public highway** with locked containers. It’s quick to set up and cheap, but traffic is stuck in the same jams as everyone else, so speed and travel time vary.
- **Direct Connect** is building a **private dedicated road** between your head office and the warehouse. Nobody else uses it, so travel time is steady and predictable, you can run many trucks at once, and the data transfer rates are lower. It takes weeks to build and costs more upfront.

### Definition

**AWS Direct Connect (DX)** is a service that creates a **dedicated, private network connection** between your on-premises network (data center, office or colocation facility) and AWS, **bypassing the public internet**.

#### What you get

- **Consistent, lower latency** and more predictable performance than internet-based connections.
- **High bandwidth**: dedicated connections come in **1 Gbps, 10 Gbps and 100 Gbps** (and 400 Gbps in some locations). Hosted connections from partners can be smaller (from 50 Mbps up).
- **Lower data-transfer-out cost** than sending the same data over the internet, since DX outbound data is charged at a reduced rate.
- **Private connectivity** to your VPCs and to AWS public services (like S3) without crossing the public internet.

#### Key terms

- **Direct Connect location**: a colocation facility where AWS has equipment. Your physical link ends here. You must get your network to this location (or use a partner who is already there).

- **Dedicated connection**: a physical Ethernet port allocated to **you** (1, 10, 100 Gbps).

- **Hosted connection**: a smaller slice of capacity provisioned by an **AWS Direct Connect Partner**. Often quicker and cheaper if you don’t need a full port.

- **Virtual interface (VIF)**: a logical connection over the physical link. You create one VIF per destination type:
    - **Private VIF**: reaches resources inside a **VPC** (private IPs), through a Virtual Private Gateway or Direct Connect Gateway.
    - **Public VIF**: reaches **AWS public services** (S3, DynamoDB, etc.) using public IP addresses, without going over the internet.
    - **Transit VIF**: reaches many VPCs through a **Transit Gateway**.

- **Direct Connect Gateway**: lets one Direct Connect connection reach **VPCs in multiple Regions** (and multiple accounts).

- **Virtual Private Gateway (VGW)**: the VPC-side endpoint for a VPN or Direct Connect private connection.

- **Transit Gateway (TGW)**: a central hub that connects many VPCs and on-premises networks, so you don’t need a web of one-to-one links.

- **BGP**: the routing protocol Direct Connect uses to exchange routes between your network and AWS.

- **Link Aggregation Group (LAG)**: bundles several physical connections into one logical connection, for more bandwidth and resilience.

- **MACsec**: optional Layer 2 encryption on dedicated 10, 100 and 400 Gbps connections.

#### The security point that trips people up

**Direct Connect is private, but it is NOT encrypted by default.** Private means the traffic doesn’t traverse the public internet. It does not mean the data is scrambled. If you need encryption (compliance often requires it), you have options:

- Run an **IPsec VPN over the Direct Connect public VIF or transit VIF** (a common pattern), or
- Use **MACsec** on supported dedicated connections, or
- Encrypt at the application layer (TLS).

##### Resilience: it’s a physical cable

A single Direct Connect link is a **single point of failure** (a cable cut, a router failure, a problem at one location). AWS recommends redundancy:

- **Two connections at two different Direct Connect locations** for maximum resilience (this is the highest-resiliency model AWS documents).
- Or a **backup Site-to-Site VPN** over the internet, which is cheaper but has lower performance during failover.

#### Direct Connect vs Site-to-Site VPN 

![[Pasted image 20261014192645.png]]

#### The industry problem it solves

**Example: a bank** moving part of its systems to AWS but keeping core ledgers in its own data center for now (hybrid).

Problems with using only the internet:

1. **Unpredictable latency**: the trading and reporting systems in AWS call the on-premises ledger thousands of times per second. Internet jitter causes timeouts.
2. **Huge data transfers**: the bank moves terabytes of transaction data every night to AWS for analytics (for example into Redshift, Lesson 27). Over a VPN this is slow and the internet data-transfer charges add up.
3. **Compliance**: regulators prefer sensitive financial traffic not to traverse the public internet.
4. **Reliability**: the internet path can degrade without warning.

A Direct Connect link (two links at two locations for resilience, with encryption added over it) gives the bank stable latency, lots of bandwidth, lower transfer costs and a private path. A common supporting setup is a **Site-to-Site VPN as a backup** in case both links fail.

**Where it is a poor fit:** a small startup with light traffic, a short-term project, or a deployment that has to be up next week. A VPN is the better, quicker and cheaper choice there.

#### CODE
**Create the AWS-side objects (CLI):**

```bash
# 1. Order a dedicated connection at a Direct Connect location (this starts a real, billable request)
aws directconnect create-connection \
  --location EqDC2 \
  --bandwidth 1Gbps \
  --connection-name bank-dx-primary

# 2. Create a Direct Connect Gateway (so one link can reach VPCs in several Regions)
aws directconnect create-direct-connect-gateway \
  --direct-connect-gateway-name bank-dx-gw \
  --amazon-side-asn 64512

# 3. Create a private virtual interface on the connection
aws directconnect create-private-virtual-interface \
  --connection-id dxcon-xxxxxxxx \
  --new-private-virtual-interface '{
      "virtualInterfaceName": "bank-private-vif",
      "vlan": 101,
      "asn": 65000,
      "authKey": "<BGP-auth-key>",
      "amazonAddress": "169.254.255.1/30",
      "customerAddress": "169.254.255.2/30",
      "directConnectGatewayId": "<dx-gateway-id>"
  }'
```

**What this does:**

- Step 1 requests the physical port. AWS then issues a **Letter of Authorization (LOA-CFA)** that you give to the colocation provider so they can run the **cross-connect** (the physical cable) to your equipment. This is the slow, human part.

- Step 2 creates the gateway object that can attach to Virtual Private Gateways or Transit Gateways in multiple Regions.

- Step 3 creates the **logical VIF** with a **VLAN** (a tag that separates traffic on the shared cable), **BGP** details (your ASN, the two link-local peering addresses and an authentication key). BGP then advertises your on-premises routes to AWS and AWS’s VPC routes to you.

- Finally you must add **route table entries** in the VPC pointing on-premises address ranges to the gateway, and allow that traffic in **security groups** and **NACLs**. A very common cause of “the link is up but nothing works” is a missing route or a blocked security group.

**Practice-friendly version: Site-to-Site VPN (works in any account):**

```bash
aws ec2 create-vpn-gateway --type ipsec.1
aws ec2 attach-vpn-gateway --vpn-gateway-id vgw-xxxx --vpc-id vpc-xxxx
aws ec2 create-customer-gateway --type ipsec.1 --public-ip <your-router-public-ip> --bgp-asn 65000
aws ec2 create-vpn-connection --type ipsec.1 \
  --customer-gateway-id cgw-xxxx --vpn-gateway-id vgw-xxxx
```

This produces the same **VGW + customer gateway + routing** concepts that Direct Connect uses, over an encrypted internet tunnel.

#### Interview questions with answers

**Q1. A security team states: “We’re using Direct Connect, so our data is encrypted in transit.” Is that correct? What would you tell them, and what are the fixes?**

**Answer:** No. Direct Connect is **private**, not **encrypted by default**. The traffic avoids the public internet, but the data on the wire isn’t scrambled, and anyone with access to the path (for example inside the colocation facility or the carrier network) could in principle read it. To meet an encryption requirement, either run an **IPsec VPN over Direct Connect** (over a public or transit VIF), use **MACsec** on supported dedicated connections (hop-by-hop Layer 2 encryption between your router and the AWS device), or encrypt at the application layer with **TLS**. Many compliance frameworks accept a layered combination.

**Q2. Your company needs connectivity to AWS by next Friday, but Direct Connect takes weeks. The data volumes are modest at first, but will grow big. What do you do?**

**Answer:** Start with a **Site-to-Site VPN**, which can be created within hours, so the project isn’t blocked. In parallel, order Direct Connect. When the physical link is ready, **keep the VPN as a backup path**. With BGP, AWS prefers Direct Connect routes over VPN routes when both exist, so traffic shifts to the dedicated link automatically and falls back to the VPN if the link fails. This is an extremely common phased rollout.

**Q3. A single Direct Connect link has been running for a year with no problems. A construction crew cuts the fiber, and the company’s AWS-hosted app loses all access to the on-premises database for 6 hours. What was the design flaw, and what would you build?**

**Answer:** The flaw was a **single point of failure**: one physical connection at one location. The fix depends on how critical the workload is. For maximum resilience, use **two Direct Connect connections at two different Direct Connect locations** (and ideally different carriers and paths), so even a whole-location failure doesn’t take the service down. A cheaper alternative for less critical workloads is **one Direct Connect link plus a Site-to-Site VPN backup**. Whichever you pick, **test the failover** regularly, because untested failover often doesn’t work, and verify that BGP is configured so the backup path really takes over.

**Q4. You have the same on-premises data center and 12 VPCs spread across 3 Regions and several accounts. How do you connect them without creating 12 separate Direct Connect setups?**

**Answer:** Use **a single Direct Connect connection with a Direct Connect Gateway**, which lets one set of virtual interfaces reach VPCs in **multiple Regions and accounts**. Where there are many VPCs, a **Transit Gateway** acts as the hub, so you attach the VPCs to the Transit Gateway and connect Direct Connect to it through a **transit VIF**, avoiding a mesh of one-to-one connections. Plan address ranges carefully so **VPC CIDRs don’t overlap** with each other or with the on-premises network, because overlapping ranges can’t be routed properly.

**Q5. The Direct Connect link shows “up” and BGP shows established, but your application servers in the VPC still can’t reach the on-premises database. List the checks you’d make.**

**Answer:** A healthy link only means the network layer is up. The problem is usually elsewhere, so I’d check in this order:

1. **VPC route table**: is there a route for the on-premises CIDR pointing at the VGW or Transit Gateway? (Enable **route propagation** or add it manually.)
2. **Security groups** on the instances: do they allow traffic from the on-premises CIDR on the right port?
3. **NACLs** on the subnets (Lesson 23): they are stateless, so both inbound and outbound rules (including return ports) must allow the traffic.
4. **On-premises firewall and routing**: is there a return route back to the VPC CIDR, and does the firewall allow it?
5. **BGP advertisements**: is the on-premises prefix actually being advertised to AWS, and is the VPC prefix being received by your router (and are any route filters or prefix limits blocking it)?
6. **Overlapping CIDRs** between the VPC and on-premises.
7. **MTU issues**: large packets dropped due to mismatched maximum transmission unit settings, especially with VPN over Direct Connect.

#### Scenario question

**Scenario:** You are the cloud engineer at **MediCore**, a healthcare-analytics company.

- Patient records live in a **data center in Pune**. For compliance, the main patient database **must stay on-premises**.
- A new analytics platform runs in **AWS Mumbai** (EC2 and Redshift) and must read about **2 TB of data from Pune every night**, and the app servers in AWS also call the Pune database in real time during the day, where latency spikes cause timeouts.
- A regulator requires that **patient data must not travel over the public internet**, and that **data in transit must be encrypted**.
- The business says the platform must **not go down** because of a single network failure.
- The company will soon add two more VPCs in **Mumbai** and one in **Hyderabad (ap-south-2)**, which must also reach Pune.
- The project must **start next week**, but a Direct Connect link normally takes 4 to 6 weeks to provision.

Design the connectivity. For each point, name the AWS features and explain your reasoning:

1. How do you get started next week, before Direct Connect is ready?
2. What is the target connectivity design (connection type, locations, interfaces) and why?
3. How do you satisfy both **“no public internet”** and **“encrypted in transit”**?
4. How do you make it resilient to a single failure, including a full location outage?
5. How do you connect the extra VPCs in Mumbai and Hyderabad without building many separate links?
6. What checks would you do if the link is up but the servers can’t reach Pune?

#### 1. Getting started next week, before Direct Connect is ready

Use a **Site-to-Site VPN** as a bridge. Create a **Virtual Private Gateway** (or a Transit Gateway, see point 5) attached to the VPC, a **Customer Gateway** representing the Pune router, and a **VPN connection**. It can be running in hours and gives two encrypted IPsec tunnels. Use **BGP (dynamic routing)** so that when Direct Connect arrives, AWS prefers the Direct Connect routes and the VPN becomes the backup automatically.

**Does this violate the regulator’s rule?** Honestly, **it depends on how the rule is read, and you must not assume**. A VPN is **encrypted**, which satisfies “encrypted in transit”, but its packets **travel across the public internet**, which conflicts with a strict reading of “must not travel over the public internet.” I would handle it this way:

- **Ask the compliance team or regulator** (in writing) whether an encrypted VPN is acceptable as a **temporary** measure.
- If it is **not** acceptable, don’t move real patient data over it. Start the project with **synthetic or de-identified test data** during the first weeks, and build and test the pipeline and applications. Real patient data starts flowing only when Direct Connect is live.
- Document the decision either way, since this is a compliance matter and not just a technical one.

I wouldn’t quietly run patient data over the VPN and hope nobody notices.

#### 2. Target connectivity design

- **Dedicated Direct Connect connections** (for example 10 Gbps, since the nightly 2 TB transfer needs to complete in a reasonable window and real-time traffic needs headroom), terminating at **Direct Connect locations in or near Mumbai**, with a carrier path from the Pune data center to those locations.
- A **private VIF** (or a **transit VIF** if using a Transit Gateway) for reaching VPC resources by private IP. Nightly data and daily app calls both use this private path.
- A **Direct Connect Gateway** so one connection can reach VPCs in more than one Region.
- **BGP** for route exchange, with a **Site-to-Site VPN kept as a backup path**.

Why: Direct Connect gives **stable low latency** (fixing the daytime timeouts), plenty of **bandwidth** for 2 TB nightly, and **lower data-transfer cost** than the internet.

Note: sizing should be checked against reality. 2 TB in a 4-hour window needs roughly 1.1 Gbps sustained (2 TB ≈ 16,000 Gb, divided by 14,400 seconds), so a 1 Gbps link would be too tight and 10 Gbps leaves comfortable headroom. A hosted connection of a few Gbps from a partner could also work if a full port is more than needed, so compare costs.

#### 3. Satisfying both “no public internet” and “encrypted in transit”

These are **two separate requirements, and Direct Connect only meets the first**. It is private but **not encrypted by default**. To add encryption, choose one or combine:

- **IPsec VPN over Direct Connect** (the VPN runs over a public or transit VIF, so traffic is encrypted and still stays off the public internet). This is a common pattern. Mind that VPN throughput per tunnel is limited (about 1.25 Gbps), so for the 2 TB nightly job you may need multiple tunnels, ECMP with a Transit Gateway, or a different option.
- **MACsec** (Layer 2 encryption) on a **dedicated 10 Gbps or higher** connection, if the locations and your equipment support it. It’s highly performant, but it protects only the link between your router and the AWS device.
- **Application-level TLS** to the database and to the data transfer jobs. This is the simplest to apply and is usually done in addition to the network-level controls (defense in depth).

I’d combine **TLS end to end** with a network-level control chosen with the compliance team, and verify exactly what the regulator accepts as proof

#### 4. Resilience to a single failure, including a full location outage

- Provision **two Direct Connect connections at two different Direct Connect locations**, ideally with **separate carriers and physical paths** from Pune. This covers a cable cut, a device failure and a full location outage. This is the maximum-resiliency pattern AWS recommends.
- Keep the **Site-to-Site VPN as a third path** (lower performance, used only if both Direct Connect links fail).
- Configure **BGP routing preferences** (for example, local preference and AS-path prepending) so the primary link carries traffic normally and the others take over on failure.
- **Test failover regularly** (for example, deliberately shutting down one link during a maintenance window), since untested failover often fails when needed.
- On the AWS side, the application should be in **multiple AZs** (Lessons 8 and 19), and the on-premises side shouldn’t be a single point of failure either (redundant routers).

Honest note: the question says the platform shouldn’t go down from a single network failure. Direct Connect resilience covers the **network path**, but the Pune database itself is still a single data center, so a data-center outage would still break the real-time calls. That’s worth flagging to the business as a separate risk.

#### 5. Connecting the extra VPCs in Mumbai and Hyderabad

- Use a **Transit Gateway (TGW)** in Mumbai as the hub. Attach all the Mumbai VPCs to it, and attach the Direct Connect through a **transit VIF** via a **Direct Connect Gateway**.
- For the **Hyderabad VPC (a different Region)**, create a Transit Gateway there too and use **Transit Gateway peering** between the two Transit Gateways, or associate the Direct Connect Gateway with the Transit Gateways in both Regions (a Direct Connect Gateway can associate with Transit Gateways in multiple Regions), so Pune can reach both Regions through the same link.
- Plan **non-overlapping CIDR ranges** across all VPCs and Pune from the start, because overlapping ranges can’t be routed.
- Be careful that if TGW is used for VPC-to-VPC traffic, you control which VPCs can talk to each other through **TGW route tables**, because for a healthcare workload you don’t want every VPC to reach every other VPC by default.

One more honest point: patient data **leaving the Mumbai Region for Hyderabad** may have its own compliance implications, so confirm that moving data between Regions is allowed.

#### 6. Checks if the link is up but the servers can’t reach Pune

In this order:

1. **VPC route tables**: is there a route to the Pune CIDR pointing at the VGW or Transit Gateway (propagation enabled)? For TGW, check the **TGW route tables** as well.
2. **Security groups**: do they allow the Pune CIDR on the database port?
3. **NACLs**: stateless, so check both directions including ephemeral return ports.
4. **BGP**: is the session established, is Pune advertising its prefix, and is AWS advertising the VPC prefixes back? Check for prefix limits or filters.
5. **Pune firewall and routing**: is there a return route to the VPC CIDRs, and are they allowed?
6. **Overlapping CIDRs** and **MTU mismatches** (especially when a VPN runs over Direct Connect).
7. If a **Direct Connect Gateway** is used, check the **allowed prefixes** on the gateway association.

---

### Amazon SQS (Simple Queue Service)

- **Lambda**  runs your code in response to an event, without you managing servers. It scales by running more copies in parallel.
- **EC2 and Auto Scaling** servers you manage, and the feature that adds or removes them with demand.
- **DynamoDB** and **RDS/Aurora**: databases that can slow down or reject requests when they’re overloaded.
- **IAM roles**: how one AWS service gets permission to call another.
- **Synchronous call**: you ask and **wait** for the answer (like a phone call). **Asynchronous call**: you leave a message and **move on** (like a text message).
- **Tight coupling**: component A calls component B directly, so if B is slow or down, A breaks too. **Loose coupling**: they communicate through something in between, so each can fail or scale independently.

A busy **restaurant**. The waiters (web servers) take orders from customers. Imagine they shout each order straight at the chef (the processing service). During the dinner rush, the chef is overwhelmed, orders get lost, and waiters stand around waiting.

**SQS is the order rail** (the ticket holder above the kitchen pass). Waiters clip order tickets onto the rail and immediately go back to serving customers. The chef takes tickets one at a time at their own pace. If the chef takes a break, the tickets wait safely on the rail. If the rush is huge, you hire more chefs, and they all pull from the same rail.

Nobody waits on anybody. The rail absorbs the difference between how fast orders arrive and how fast they can be cooked.

### Definition

**Amazon SQS** is a fully managed **message queue** service. One part of your application (the **producer**) sends **messages** to a queue, and another part (the **consumer**) reads and processes them later. The queue stores messages until they are processed.

It **decouples** components, smooths out traffic spikes, and makes systems more reliable. There are no servers to manage, and it scales automatically.

#### How it works

1. **Producer** sends a message to the queue (`SendMessage`).
2. The message is stored redundantly across multiple AZs.
3. **Consumer** asks for messages (`ReceiveMessage`). This is a **pull** model: consumers poll the queue.
4. When a consumer receives a message, the message **isn’t deleted**. It becomes **invisible** to other consumers for a period called the **visibility timeout**.
5. After successfully processing it, the consumer must **delete** the message (`DeleteMessage`). If it doesn’t (because it crashed, for example), the visibility timeout expires and the message **reappears** in the queue for another consumer to try.

Step 5 is the key idea. It gives SQS **at-least-once** behavior: nothing is lost, but a message might be **delivered more than once**, so your processing should be **idempotent** (safe to do twice).

#### Key terms

- **Message**: up to **1 MiB** of text (the limit was raised from 256 KiB in 2025). For larger payloads, store the data in S3 and send a pointer in the message.

- **Visibility timeout**: how long a received message is hidden from other consumers (default 30 seconds, up to 12 hours). Set it **longer than your processing time**, or another consumer will pick up the same message while the first is still working on it, causing duplicates.

- **Retention period**: how long unprocessed messages are kept (default **4 days**, up to **14 days**).

- **Delay queue / message timers**: postpone delivery of new messages (up to 15 minutes).

- **Short polling vs long polling**: short polling returns immediately (often empty); **long polling** (`WaitTimeSeconds` up to 20) waits for messages to arrive. Long polling is **cheaper and more efficient**, so it’s the usual choice.

- **Dead-letter queue (DLQ)**: a separate queue where messages go after failing to process a set number of times (`maxReceiveCount`). It prevents a “poison message” (one that always fails) from being retried forever, and lets you inspect failures later.

- **Batching**: send or receive up to 10 messages per call, to cut cost and improve throughput.

#### Two queue types

![[Pasted image 20261015102829.png]]

FIFO uses **message group IDs**: ordering is guaranteed **within a group**, while different groups can be processed in parallel.

#### Security and other features

- **Encryption**: server-side encryption is on by default (SQS-managed keys), with the option of KMS keys. TLS in transit.
- **Access control**: IAM policies and **queue policies** (resource policies, for cross-account or cross-service access).
- **Pricing**: pay per request (and payload size in 64 KiB chunks), with a free tier. There’s no cost for an empty, unused queue beyond requests.

#### What SQS is NOT

- It’s **not pub/sub**. In a queue, each message is processed by **one** consumer (competing consumers). If you need one event delivered to **many** subscribers, that’s **SNS**, and the two are often combined.
- It’s **not a database or a long-term log**. Messages are temporary (14 days at most) and are deleted once processed.
- It doesn’t **push** messages. Consumers pull (although Lambda’s integration polls for you).

### The industry problem it solves

**Example: an online store during a flash sale.**

The checkout flow does a lot after a customer clicks “Pay”: charge the card, update inventory, send a confirmation email, generate an invoice PDF, notify the warehouse. If checkout does all of this **synchronously**:

1. The customer waits many seconds for the page to respond.
2. During a spike, the email service or the invoice generator slows down, and **checkout itself slows or fails**, even though the payment succeeded.
3. If the invoice service crashes mid-task, the order data for that step is **lost**.
4. To survive the peak, the whole system must be scaled for the worst case.

With SQS, the checkout service does the **essential** work (take the payment, save the order), puts a message on a queue (“order 8841 needs invoice and email”), and responds to the customer immediately. A fleet of workers (Lambda or EC2) drains the queue at its own pace. If the spike is 10x bigger than workers can handle, the **queue grows** instead of the system failing. Auto Scaling can add workers based on the **queue depth** (the number of waiting messages). Failed messages go to a **DLQ** for investigation.

**Where it is a poor fit:** when the caller needs the answer immediately (a synchronous request/response, like “is this password correct?”), when you need real-time fan-out to many receivers (use SNS), or when you need to replay a long history of events (use a streaming service like Kinesis).

## CODE 

**Create a queue with a dead-letter queue, then send and receive (CLI):**

```bash
# 1. Create the dead-letter queue first
aws sqs create-queue --queue-name orders-dlq

# Get its ARN
aws sqs get-queue-attributes \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/orders-dlq \
  --attribute-names QueueArn

# 2. Create the main queue, pointing at the DLQ after 3 failed attempts
aws sqs create-queue --queue-name orders-queue \
  --attributes '{
    "VisibilityTimeout": "60",
    "ReceiveMessageWaitTimeSeconds": "20",
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:ap-south-1:123456789012:orders-dlq\",\"maxReceiveCount\":\"3\"}"
  }'

# 3. Send a message
aws sqs send-message \
  --queue-url https://sqs.ap-south-1.amazonaws.com/123456789012/orders-queue \
  --message-body '{"order_id": 8841, "task": "send_invoice"}'
```

**A worker in Python (boto3):**


```python
import boto3, json

sqs = boto3.client("sqs", region_name="ap-south-1")
queue_url = "https://sqs.ap-south-1.amazonaws.com/123456789012/orders-queue"

while True:
    resp = sqs.receive_message(
        QueueUrl=queue_url,
        MaxNumberOfMessages=10,
        WaitTimeSeconds=20          # long polling
    )
    for msg in resp.get("Messages", []):
        try:
            order = json.loads(msg["Body"])
            process_order(order)    # your logic; must be safe to run twice
            sqs.delete_message(     # delete ONLY after success
                QueueUrl=queue_url,
                ReceiptHandle=msg["ReceiptHandle"]
            )
        except Exception as e:
            print("Failed, will retry after visibility timeout:", e)
```

**What this does:**

- The **DLQ** is created first, because the main queue’s **redrive policy** needs its ARN. After 3 failed receives (`maxReceiveCount`), a message moves to the DLQ instead of looping forever.
- `VisibilityTimeout: 60` gives a worker 60 seconds to finish before the message reappears. It should be longer than your slowest normal processing.
- `ReceiveMessageWaitTimeSeconds: 20` makes **long polling** the default, which reduces empty responses and cost.
- In the worker, `delete_message` is called **only after success**. If `process_order` raises an error, the message isn’t deleted, becomes visible again after the timeout, and is retried (and eventually lands in the DLQ).
- Because delivery is at-least-once, `process_order` should be **idempotent**: for example, check whether invoice 8841 already exists before creating it.

**With Lambda (no polling code needed):** you attach the queue as an **event source mapping**, and Lambda polls, invokes your function with a batch of messages, and deletes them when the function succeeds. If the function fails, the messages return to the queue. Set the queue’s visibility timeout to **at least 6 times the function timeout**, which AWS recommends for this integration.

### Interview questions with answers

**Q1. Your workers process each order and charge a customer. After a deployment, customers report being **charged twice**. The queue is a Standard queue. What happened, and how do you fix it?**

**Answer:** Standard queues guarantee **at-least-once** delivery, so the same message can legitimately be delivered **more than once**. It can also happen when processing takes **longer than the visibility timeout**, so the message becomes visible again while the first worker is still working, and a second worker picks it up. The fixes: make the processing **idempotent** (for example, store a unique order or payment ID, and skip the charge if it was already done, or pass an idempotency key to the payment provider), **increase the visibility timeout** above the maximum processing time (and extend it dynamically with `ChangeMessageVisibility` for long jobs), and, if strict de-duplication is needed, consider a **FIFO queue** with a deduplication ID (while still keeping idempotent logic, since de-duplication only covers a 5-minute window).

**Q2. One message in your queue contains malformed data that crashes the worker every time. The queue keeps retrying it, workers keep crashing, and the queue’s backlog is growing behind it. What’s the problem and the standard solution?**

**Answer:** That is a **poison message**. It fails, becomes visible again, and is retried endlessly, wasting capacity (and in a FIFO queue it can even **block the entire message group** behind it). The solution is a **dead-letter queue**: set a **redrive policy** with a sensible `maxReceiveCount` (for example 3 to 5), so after repeated failures the message moves to the DLQ. Then **set a CloudWatch alarm** on the DLQ depth so someone is alerted, inspect and fix the message or the bug, and use **DLQ redrive** to send fixed messages back to the source queue. Also validate the input in the worker, and make the DLQ’s retention period longer than the source queue’s so the messages aren’t expired before anyone looks at them.

**Q3. Your queue is a Standard queue, and an engineer insists “SQS keeps messages in order, so it’s safe to process ‘create account’ then ‘update account’ in sequence.” Is that true?**

**Answer:** No. Standard queues give **best-effort ordering only**, so “update account” could be delivered before “create account,” especially with multiple consumers. If ordering matters, use a **FIFO queue**, with the same **message group ID** for all messages about the same account (ordering is guaranteed **within a group**, while different accounts can be processed in parallel in different groups). Note the trade-off: FIFO has lower throughput than Standard and a message group is processed **sequentially**, so one slow or failing message holds up its group. An alternative is to **design the messages to be order-independent** (for example, each message carries a version number, and the consumer ignores stale updates).

**Q4. Messages are piling up during a traffic spike, and the on-call engineer wants to add EC2 workers. How would you automate this, and what metric would you scale on?**

**Answer:** Use **EC2 Auto Scaling** (Lesson 19) driven by queue metrics. The raw `ApproximateNumberOfMessagesVisible` isn’t ideal on its own, because the right worker count depends on **how fast each worker processes**. The AWS-recommended approach is a **“backlog per instance”** custom metric: queue depth divided by the number of running instances, and a **target tracking** policy that keeps it at an acceptable level (derived from your processing time and the latency you can tolerate). If using Lambda as the consumer, it scales out automatically with the queue (subject to concurrency limits, which can also be set to protect downstream databases from being overwhelmed). Also set a **maximum** on scale-out so a spike doesn’t overwhelm the database downstream, which is the same problem the queue was meant to prevent.

**Q5. A teammate wants to put a 50 MB video file into an SQS message body. Is that possible, and what should you do instead?**

**Answer:** Not directly, because the message size limit is **1 MiB**. The standard pattern (the **claim-check pattern**) is to **upload the file to S3** and send a small message containing the **S3 bucket and key** (and any metadata). The consumer reads the message, downloads the object from S3, and processes it. AWS also provides an **Extended Client Library** (for Java and Python) that automates this approach. This also keeps queue costs down and avoids pushing large payloads through the queue.

### Scenario question

**Scenario:** You are the cloud engineer at **QuickPrint**, a service where customers upload photos and order printed photo books.

- When a customer clicks “Place order”, the web app currently does everything in one request: charges the card, resizes and watermarks up to 200 photos, generates a PDF proof, emails the customer, and tells the print shop’s system. Sometimes this takes **90 seconds**, and the browser times out. On festival weekends, order volume is **20 times** normal and the servers crash, **losing orders**.
- The **card charge** must happen exactly once per order, and customers must never be double-charged.
- Photo processing is heavy, takes **1 to 3 minutes per order**, and the print shop’s system can accept only **10 requests at a time** before it starts rejecting them.
- Some orders contain corrupted photo files that **crash the processing code** every time.
- Order status updates for a **single customer** (for example “processing” then “shipped”) must be applied in the correct order.
- Orders should still be accepted even if the processing servers are **temporarily down**.

Design the solution with SQS. For each point, name the SQS features and explain your reasoning:

1. How do you restructure the flow so customers get a fast response and no orders are lost?
2. Which parts go into a queue, and which parts do **not**?
3. How do you protect against double charging and duplicate processing?
4. Standard or FIFO queue, for which messages, and why?
5. How do you handle the corrupted photo files?
6. How do you handle the 20x spike, and how do you protect the print shop’s 10-request limit?
7. How do you pass 200 photos through the system when messages are small?

#### 1. Restructuring the flow for a fast response and no lost orders

Split the request into a **fast synchronous part** and a **slow asynchronous part**.

- **Synchronous (what the customer waits for):** validate the order, **charge the card**, save the order to the database with status `RECEIVED`, put a message on an SQS queue, and return “Order placed” within a second or two.
- **Asynchronous (queue-driven workers):** resize and watermark photos, generate the PDF proof, email the customer, and send the job to the print shop.

SQS stores messages **redundantly across multiple AZs**, so once the message is accepted, it survives server crashes. The “lose orders” problem disappears because the order is saved and queued **before** the response goes back. If the processing servers are down, messages simply wait in the queue (retention up to 14 days) and are processed when workers return. That meets the requirement that orders are still accepted while processing is down.

One honest caveat: “save the order to the database” and “send the message to the queue” are two separate operations, and one could succeed while the other fails. A common safeguard is the **transactional outbox pattern** (write the message into the database in the same transaction, then a small process publishes it to the queue), or at minimum a reconciliation job that finds orders stuck in `RECEIVED` with no processing started.

#### 2. What goes into a queue, and what does not

**Not queued:** the **card charge**. The customer needs to know immediately whether payment succeeded, and this step needs a direct answer (synchronous request and response). Queueing it would also make “exactly once” harder.

**Queued:**

- Photo processing (the heavy 1 to 3 minute job).
- Proof PDF generation.
- Confirmation email.
- Sending the job to the print shop.
- Order status updates.

#### 3. Protecting against double charging and duplicate processing

Two different problems:

- **Double charging** is prevented **outside the queue**, because the charge isn’t queued. Send an **idempotency key** (such as the order ID) to the payment provider, so a retried request (for example the customer double-clicking, or the browser retrying) can’t create a second charge. Also store the payment status against the order, and check it before charging.
- **Duplicate processing** inside the queue: SQS guarantees at-least-once delivery for Standard queues, so the same message can be delivered twice, and it also reappears if processing outlasts the **visibility timeout**. So:
    - Make every worker **idempotent**: before processing, check the order’s status in the database (for example, “if proof already generated, skip”), or use a conditional update so only one worker can move the order from `RECEIVED` to `PROCESSING`.
    - Set the **visibility timeout longer than the longest processing time** (more than 3 minutes here, so something like 5 to 10 minutes), and extend it with `ChangeMessageVisibility` for unusually large orders.

#### 4. Standard or FIFO, for which messages?

- **Photo processing and proof generation: Standard queue.** Orders are independent of each other, order doesn’t matter, and the festival spike needs the high throughput Standard offers. Duplicates are handled by idempotent workers.
- **Order status updates: FIFO queue**, with the **customer ID (or order ID) as the message group ID**. This guarantees “processing” is applied before “shipped” **for the same customer**, while different customers’ updates are still processed in parallel. Use a **deduplication ID** (for example order ID plus status) so repeated sends are dropped within the 5-minute window.
- The **email** and **print shop** messages can be Standard queues too, since order doesn’t matter there, although the email should be idempotent so customers don’t get duplicates.

An alternative for statuses is to put a version number on each update and let the consumer ignore stale ones, which allows Standard queues everywhere. The FIFO approach is simpler to reason about for this requirement, and the throughput limit of FIFO isn’t a concern at status-update volumes per customer.

#### 5. Handling the corrupted photo files

Use a **dead-letter queue** on the photo-processing queue with a **redrive policy** (for example `maxReceiveCount` of 3). A corrupted order would otherwise crash a worker, reappear, crash another, and loop forever, wasting capacity. After 3 failed attempts, it goes to the **DLQ**. Then:

- Set a **CloudWatch alarm** on the DLQ depth (covered in a later lesson), so someone is alerted.
- Update the order status to something like `NEEDS_ATTENTION` and notify the customer to re-upload the broken photo (a worker can do this when it detects a failure, or a DLQ consumer can).
- Set the DLQ’s **retention period longer** than the source queue’s, so messages don’t expire before anyone examines them.
- Better still, **catch exceptions in the worker and validate files early** (for example, check each image before heavy processing), so the common failure produces a clean “bad photo” result immediately instead of a crash. The DLQ remains the safety net for unexpected failures.

#### 6. Handling the 20x spike and protecting the print shop’s 10-request limit

**For the spike:**

- The queue absorbs it: orders accumulate instead of crashing servers. The web tier only has to do the fast part (charge, save, enqueue).
- Scale the workers with **Auto Scaling driven by backlog per instance** (queue depth divided by the number of workers), or use **Lambda** if the processing fits (see the caveat below). Set a **maximum** on scale-out to protect downstream systems.
- Customers see slightly longer processing times during the peak, which is a good trade-off compared with failures. Show an honest status (“We’re processing your order, you’ll get an email in about X”).

**For the print shop’s limit of 10 concurrent requests:** give the “send to print shop” step **its own queue with a limited number of consumers**. With **Lambda**, set **reserved concurrency (or the SQS event source’s maximum concurrency)** to 10 or a bit less, so no more than 10 calls are in flight. With EC2 workers, run a fixed small pool. The rest wait safely in the queue. The queue works as a **buffer between a fast producer and a slow, limited consumer**, which is a core use case. If the print shop rejects a request anyway (for example, a 429 error), the worker doesn’t delete the message, and it retries after the visibility timeout. Backing off with increasing delays is a good practice here.

A caveat on Lambda for photo processing: Lambda has a **maximum 15-minute runtime** and limited memory and temporary storage, which is fine for 1 to 3 minute jobs in many cases, but if 200 high-resolution photos exceed those limits, use containers or EC2 workers (or split the order into one message per photo, see point 7).

#### 7. Passing 200 photos through the system when messages are small

Use the **claim-check pattern**. SQS messages are limited to 1 MiB (and large payloads are wasteful anyway), so:

- The photos are **uploaded to S3** (most likely already the case, for example via pre-signed upload URLs).
- The queue message contains only **small metadata**: order ID, customer ID, and the **S3 bucket and prefix** (or a list of keys).
- The worker reads the message, downloads photos from S3, processes them, and writes the results (resized images, the PDF) back to S3, updating the order record in the database.

An optional improvement is to **fan out one message per photo** (or per batch of photos) so many workers process a big order in parallel and finish faster. That needs a way to know when all pieces are done (for example, a counter in the database that triggers the PDF step when it reaches 200). This is more complex, so start simple and move to it only if processing time requires it.

#### One queue or several?

| Queue                | Type                                | Why separate                                      |
| -------------------- | ----------------------------------- | ------------------------------------------------- |
| Photo processing     | Standard + DLQ                      | Heavy, scalable, poison-message risk              |
| Email                | Standard (+ DLQ)                    | Cheap and light, shouldn’t wait behind photo jobs |
| Print shop           | Standard (+ DLQ), limited consumers | Concurrency capped at the print shop’s limit      |
| Order status updates | **FIFO**                            | Ordering per customer                             |

Separate queues give each step its **own scaling, visibility timeout, retry and DLQ settings**, and one slow or failing step can’t block the others. A related improvement, which SNS makes possible in our next lesson, is **publishing one “order placed” event that fans out to all these queues**, instead of the web app sending several separate messages.

---

## Amazon SNS (Simple Notification Service)

- **SQS** is a queue: a producer puts a message in, and **one** consumer pulls it out and processes it. Consumers **pull**.
- **Lambda** runs your code in response to events.
- **Producer / consumer**: the side that creates a message, and the side that handles it.
- **Loose coupling**: components communicate through something in between, so each can fail or scale independently.
- **IAM policies**: JSON rules that decide who can do what. **Resource policies** are attached to the resource itself (like a queue or topic).

SQS was the **order rail** in a kitchen, where each ticket is taken by one chef.

SNS is a **school’s public-address announcement system** combined with a **notice-board subscription list**. When the principal makes **one announcement** (“School closes early today”), the system delivers it to **everyone who subscribed**: every classroom speaker, the staff WhatsApp group, the parents’ SMS list and the bus company’s email. The principal doesn’t call each person. They speak once, and the system fans it out.
The principal (publisher) doesn’t know or care who is listening. New listeners can subscribe later without the principal changing anything.

### Definition

**Amazon SNS** is a fully managed **publish/subscribe (pub/sub)** messaging service. A **publisher** sends a message to a **topic**, and SNS **pushes a copy of that message to every subscriber** of that topic.

It handles two kinds of use:

- **Application-to-application (A2A)**: delivering events to SQS queues, Lambda functions, HTTP/S endpoints, Kinesis Data Firehose and more.
- **Application-to-person (A2P)**: delivering notifications to people through **SMS, email and mobile push** notifications.

#### Core concepts

- **Topic**: a named channel that messages are published to (for example `order-placed`). It’s the central object.

- **Publisher**: anything that sends a message to the topic (your app, an AWS service, a CloudWatch alarm).

-**Subscription**: a connection between a topic and an **endpoint**. Each subscription has a **protocol**:
    - **SQS** (queue), **Lambda** (function), **HTTP/HTTPS** (a webhook), **Email / Email-JSON**, **SMS**, **mobile push** (platform applications), **Kinesis Data Firehose**.

- **Message**: up to **256 KiB** (check the current limit, as SQS’s limit was recently raised and SNS’s may follow). For large payloads, store the data in S3 and send a pointer.

- **Message attributes**: key-value metadata attached to a message, used for filtering.

- **Subscription filter policy**: a JSON rule on a subscription saying “only deliver me messages where `eventType` is `order_placed`” or “where `amount` is greater than 1000”. This lets one topic serve many different subscribers without each one receiving everything.

- **Delivery retries**: SNS retries failed deliveries according to a delivery policy (the retry behavior depends on the endpoint type). For SQS and Lambda endpoints, retries are persistent, and for HTTP endpoints they follow a configurable policy.

- **Dead-letter queue for subscriptions**: an SQS queue attached to a **subscription** (not the topic) that captures messages SNS couldn’t deliver to that endpoint.

#### Two topic types

![[Pasted image 20261015201216.png]]

Use FIFO topics when order matters across many subscribers.

### SNS vs SQS: SNS tells everyone, SQS holds work for someone.

![[Pasted image 20261015201309.png]]

#### The most important pattern: SNS + SQS fan-out

Publish **once** to an SNS topic, and subscribe **several SQS queues** to it. Each queue gets its own copy, and each team’s workers process their copy independently, at their own speed, with their own retries and DLQs. You get the **broadcast** of SNS **plus** the **durability and buffering** of SQS. It’s the standard design for event-driven systems.

#### Other things worth knowing

- **Encryption**: server-side encryption with KMS and TLS in transit.

- **Access control**: IAM policies and **topic policies** (needed, for example, to let an S3 bucket or another account publish, or to let SNS write to an SQS queue; the queue also needs a policy allowing the topic).

- **Many AWS services publish to SNS**: CloudWatch alarms (to alert people), S3 event notifications, Auto Scaling events, AWS Budgets, CloudFormation and more.

- **SMS** has costs and country-specific regulations (sender registration, opt-outs), and it’s often **sandboxed** at first in a new account, which is a frequent real-world surprise.

- **Email via SNS** is for **notifications to a few people (like an operations team)**, and **not** for bulk marketing or transactional customer email. Use a dedicated email service for that (Amazon SES).

- It’s priced per request, per delivery (differs by protocol, SMS being the most expensive), with a free tier.

### The industry problem it solves

**Example: an online store**, again.

When an order is placed, many things must happen: the warehouse is told, the invoice is generated, the customer gets an email, the analytics system records a sale, the fraud-detection service scores the order, and (soon) a loyalty-points service awards points.

If the checkout service **calls each of these directly**:

1. Checkout must **know about every downstream system** and be changed each time a new one is added (the loyalty service means a new deployment of checkout).
2. If one downstream is slow or down, **checkout is affected**.
3. Adding or removing a consumer is a risky change to the core system.

With **SNS**, checkout publishes **one** “OrderPlaced” event to a topic. The warehouse, invoice, email, analytics, fraud and loyalty services each **subscribe independently** (usually through their own SQS queue). Adding a new service means **adding a subscription**, with zero change to checkout. Filter policies let each subscriber see only what it needs (for example, fraud checks only for orders over ₹50,000).

A second common use: **operations alerts**. A CloudWatch alarm (“CPU above 90% for 5 minutes”) publishes to an SNS topic, which emails the on-call team, sends an SMS to the manager and triggers a Lambda function that runs automated remediation.

**Where it is a poor fit:** when you need messages **stored** until a consumer is ready (use SQS, or SNS with SQS), when you need **replayable history** (use a streaming service), or when the sender needs a **direct response** (use a synchronous call).

## CODE

Create a topic, subscribe a queue and an email, and publish (CLI):

```bash
# 1. Create the topic
aws sns create-topic --name order-placed

# 2. Subscribe an email address (the person must click the confirmation link)
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:order-placed \
  --protocol email \
  --notification-endpoint ops-team@example.com

# 3. Subscribe an SQS queue (with a filter so it only gets large orders)
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:order-placed \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:ap-south-1:123456789012:fraud-queue \
  --attributes '{"FilterPolicy": "{\"amount\": [{\"numeric\": [\">\", 50000]}]}", "RawMessageDelivery": "true"}'

# 4. Publish a message with attributes
aws sns publish \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:order-placed \
  --message '{"order_id": 8841, "amount": 72000}' \
  --message-attributes '{"amount": {"DataType": "Number", "StringValue": "72000"}}
```

**The queue policy that lets the topic write to the queue (this step is the one people forget):**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "sns.amazonaws.com" },
    "Action": "sqs:SendMessage",
    "Resource": "arn:aws:sqs:ap-south-1:123456789012:fraud-queue",
    "Condition": {
      "ArnEquals": { "aws:SourceArn": "arn:aws:sns:ap-south-1:123456789012:order-placed" }
    }
  }]
}
```

**Publishing from Python (boto3):**

```python
import boto3, json

sns = boto3.client("sns", region_name="ap-south-1")

sns.publish(
    TopicArn="arn:aws:sns:ap-south-1:123456789012:order-placed",
    Message=json.dumps({"order_id": 8841, "amount": 72000}),
    MessageAttributes={
        "amount": {"DataType": "Number", "StringValue": "72000"}
    },
)
```

- Step 1 creates the topic. Step 2 subscribes an email address, which stays **pending** until the recipient confirms. Step 3 subscribes an SQS queue with a **filter policy**, so this queue only receives messages whose `amount` attribute is above 50,000.

- **`RawMessageDelivery: true`** delivers just your message body to the queue. Without it, SQS receives an **SNS envelope (JSON wrapper)** with metadata, and your worker must unwrap it. Knowing about this wrapper saves a lot of debugging.

- The **queue policy** lets SNS write to the queue. Without it, the subscription is created but **messages silently don’t arrive**. This is one of the most common real-world problems.

- The publish call includes a **message attribute**, which is what the filter policy checks. **Filter policies apply to attributes by default** (SNS also supports filtering on the message body if you set the filter scope to `MessageBody`).

- A message with amount 72,000 goes to both the email subscriber and the fraud queue. A message with amount 900 goes only to the email subscriber.

### Interview questions with answers

**Q1. When would you use SNS, when SQS, and why is the answer often “both together”?**

**Answer:** Use **SQS** when work must be **buffered and handled by one consumer** (for example, a pool of workers processing jobs), and use **SNS** when **one event must reach many different receivers** at once. SNS alone has a weakness: it **doesn’t store messages** for a subscriber that’s down or slow beyond its delivery retries, and subscribers can’t control their own pace. SQS alone has the limit that each message goes to a single consumer. Combining them (**SNS topic → multiple SQS queues**) gives you **broadcast plus durable buffering**: every team gets its own copy of the event, and each processes it at its own speed with its own retries and DLQ, and a slow or failed consumer never affects the others.

**Q2. You subscribed an SQS queue to an SNS topic and published a test message. The subscription shows “Confirmed,” but the queue stays empty. What would you check?**

**Answer:** Most likely the **SQS queue policy** doesn’t allow the SNS topic to send messages. The queue needs a policy statement allowing `sqs:SendMessage` from the SNS service, restricted by `aws:SourceArn` to the topic’s ARN. Other checks: the **filter policy** on the subscription might be filtering the message out (a message without the expected attribute is dropped); the message may have been published **to a different topic or Region**; the **queue’s encryption (KMS key) policy** may not allow SNS to use the key if the queue is encrypted with a customer-managed key; and a **DLQ on the subscription** can reveal delivery failures, along with SNS delivery-status logging or CloudWatch metrics such as `NumberOfNotificationsFailed`.

**Q3. Your worker reads messages from the queue that is subscribed to an SNS topic, and the code crashes on `message["order_id"]`, but when you published directly to the queue it worked fine. What’s going on?**

**Answer:** SNS wraps the message in an **envelope**. The queue receives a JSON object with fields such as `Type`, `MessageId`, `TopicArn`, `Message` (a string containing your original payload) and `Timestamp`, and your actual data is a **JSON string inside the `Message` field**. The worker must parse the outer JSON, then parse the inner `Message` string. Alternatively, enable **raw message delivery** on the subscription, so the queue receives your payload exactly as published. Note that with raw delivery, message attributes arrive as SQS message attributes instead of inside the envelope.

**Q4. One topic receives all company events (orders, refunds, signups). The refund team’s queue is being flooded with order messages they don’t need, wasting their processing and money. How do you fix it without creating separate topics?**

**Answer:** Use **subscription filter policies**. Publishers attach a message attribute such as `eventType` (`order`, `refund`, `signup`), and the refund team’s subscription has a filter policy like `{"eventType": ["refund"]}`. SNS then delivers only matching messages to that queue. Filtering happens inside SNS, so the unwanted messages never reach the queue and incur no processing cost. You can also filter on **message body** content (by setting the filter policy scope), and use operators like prefix, numeric ranges and “anything-but”. Separate topics are sometimes the better design when event types are very different or need different access control, but filter policies are the lightweight answer here.

**Q5. A payment event must be delivered to multiple services **in the exact order** it occurred (authorize → capture → refund) for each payment. A standard SNS topic sometimes delivers them out of order. What are the options?**

**Answer:** Use an **SNS FIFO topic** with **FIFO SQS queues** as subscribers. Set the **message group ID** to the payment ID, so ordering is guaranteed within each payment while different payments are processed in parallel, and use a deduplication ID (or content-based deduplication) to drop duplicates in the 5-minute window. The trade-offs: FIFO topics have **lower throughput limits** than standard topics, and they only support **SQS and Lambda** style subscriptions (not email or SMS). Also, SNS FIFO ordering applies per message group, so choose the group ID carefully (too coarse and you lose parallelism, too fine and you lose the ordering you need). A design alternative is to make events self-describing (sequence numbers or timestamps) so consumers can detect and ignore out-of-order or stale events, even on standard topics.

### 6. Scenario question

**Scenario:** You are the cloud engineer at **ShopEasy**, an online marketplace.

- When a customer places an order, **six different systems** must react: the **warehouse** system, the **invoice** generator, the **email/SMS notification** service, the **analytics** pipeline, the **fraud-detection** service, and, soon, a new **loyalty-points** service.
- Right now the checkout service calls each of them directly. Adding the loyalty service would mean changing and redeploying checkout, and **when the invoice service is down, checkout fails**.
- The fraud team only wants orders **above ₹50,000**. The loyalty team only wants orders from **registered members**. Everyone else wants all orders.
- The warehouse system is slow and processes about **5 orders per second**, while peak order volume is **200 per second**. It must **never lose** an order.
- The warehouse team’s workers expect plain order JSON, but today the messages they receive from the test setup have extra fields wrapped around the data, and they ask what went wrong.
- The **operations team** must receive an **email and an SMS** when the order-processing system raises a serious alarm (for example, error rate above 5%), and an **automatic remediation function** should run at the same time.
- **Refund events** must be applied in the correct order for each order (refund requested → approved → paid).

Design the solution with SNS (and whatever else we’ve covered). For each point, name the features and explain your reasoning:

1. How do you restructure the order flow so checkout doesn’t depend on each downstream system?
2. How do you give the fraud and loyalty teams only the orders they need?
3. How do you protect the slow warehouse system from the 200-per-second peak without losing orders?
4. What do you tell the warehouse team about the wrapped message, and what do you change?
5. How do you set up the operations alerting with email, SMS and automatic remediation?
6. How do you handle ordered refund events?
7. What do you do about deliveries that fail repeatedly for one subscriber?

 **what would you need to change to add the loyalty service later, and why is that better than before?**
#### 1. Restructuring the flow so checkout doesn’t depend on downstream systems

Use **SNS fan-out**. Checkout does its essential work (take payment, save the order) and publishes **one `OrderPlaced` event** to an SNS topic. It knows nothing about who is listening.

Each downstream system subscribes **independently**, normally through **its own SQS queue** (SNS + SQS fan-out):

- Warehouse queue, invoice queue, notification queue, analytics queue, fraud queue.
- Each team’s workers pull from their own queue at their own pace.

This fixes the “invoice service down means checkout fails” problem. If invoice is down, its queue simply accumulates messages (retained up to 14 days), and checkout never notices. A slow or failing consumer can’t hurt the others.

Some subscribers can attach differently where it makes sense. For example, analytics might use **Kinesis Data Firehose**, and a light task might use a **Lambda** subscription directly. But SQS is the safe default whenever a consumer needs durability or controls its own speed.

#### 2. Giving fraud and loyalty only the orders they need

Use **subscription filter policies**.

- Checkout publishes with **message attributes**, such as `amount` (Number) and `memberStatus` (String, for example `registered` or `guest`).
- **Fraud queue subscription** filter: `{"amount": [{"numeric": [">", 50000]}]}`.
- **Loyalty queue subscription** filter: `{"memberStatus": ["registered"]}`.
- Warehouse, invoice, notifications and analytics have **no filter**, so they get everything.

Filtering happens inside SNS, so unwanted messages never reach those queues and cost nothing to process. If you’d prefer filtering on fields inside the JSON body instead of attributes, SNS supports a `MessageBody` filter scope, but attributes are the simpler default.

#### 3. Protecting the slow warehouse system from the 200-per-second peak

This is **SQS’s job** (Lesson 31), not SNS’s. SNS pushes messages immediately and **doesn’t buffer for slow consumers**, so a slow warehouse can’t be subscribed directly. Put the **warehouse’s SQS queue** between SNS and the workers:

- SNS delivers to the queue at up to 200 per second, which SQS handles easily.
- The warehouse workers pull at **5 per second**, and the queue grows during the peak and drains afterward. Backlog is fine, because messages are stored durably across AZs for up to 14 days.
- Add a **dead-letter queue** to the warehouse queue (with `maxReceiveCount`) so a broken order doesn’t block everything, plus a CloudWatch alarm on queue age and depth.
- Set the **visibility timeout** longer than the warehouse’s processing time.
- If the backlog is unacceptable for business reasons (for example, orders waiting hours), the real fix is more warehouse capacity, since a queue only smooths the load, it doesn’t create processing speed.

“Never lose an order” is met by SQS’s durability **plus** a delivery failure net on the SNS side (see point 7).

#### 4. The wrapped message the warehouse team sees

SNS wraps the payload in an **envelope**: a JSON object with fields like `Type`, `MessageId`, `TopicArn`, `Timestamp`, and a `Message` field that holds **your original JSON as a string**. Nothing went wrong, that’s the default behavior.

Two options:

- **Enable raw message delivery** on the warehouse queue’s subscription (`RawMessageDelivery: true`). The queue then receives exactly the JSON that checkout published. This is the best fix, and it needs no code change on the warehouse side. (Message attributes then arrive as SQS message attributes.)
- Or change the warehouse code to parse the outer envelope and then parse the inner `Message` string.

Tell them to make this change **per subscription**, since other subscribers might already depend on the envelope.

#### 5. Operations alerting with email, SMS and automatic remediation

- Create a CloudWatch **alarm** on the error-rate metric (above 5%, evaluated over a sensible period) with an **SNS topic `ops-alerts`** as its alarm action. (CloudWatch is next lesson.)
- Subscribe to that topic:
    - **Email** for the on-call team (each recipient must confirm).
    - **SMS** for the manager or on-call phone numbers, keeping in mind that SMS has cost, country regulations, and sandbox restrictions on new accounts.
    - A **Lambda function** that runs the automated remediation (for example restarting a service, scaling out workers, or clearing a stuck process).
- All three fire from **one alarm**. Use a **separate topic from `OrderPlaced`**, so operational alerts and business events aren’t mixed, and so access control can differ.
- Make the Lambda function **idempotent and safe**, since alarms can trigger repeatedly. A Lambda that fixes things automatically should have limits (for example, don’t restart more than N times per hour), or you risk a loop that makes the incident worse.

#### 6. Ordered refund events

Use an **SNS FIFO topic** (`refund-events.fifo`) with **FIFO SQS queues** as subscribers:

- Set the **message group ID to the order ID**, so for each order, “requested → approved → paid” is delivered in order, while different orders are processed in parallel.
- Use a **deduplication ID** (or content-based deduplication), so a retried publish within the 5-minute window isn’t delivered twice.
- Trade-offs to state: FIFO topics have **lower throughput than standard topics**, only support **SQS and Lambda** style subscribers (no email or SMS), and a failing message **blocks the rest of its group** until it’s handled or moved to a DLQ.

As a defense in depth, put a **sequence number or status version** on each event, so consumers can detect and ignore stale or out-of-order updates.

#### 7. Deliveries that fail repeatedly for one subscriber

- Attach a **subscription-level dead-letter queue** (an SQS queue) to that subscription, so messages SNS **can’t deliver** are captured instead of being dropped after retries are exhausted. This helps meet “never lose an order”.
- Set a sensible **delivery retry policy** (most relevant for HTTP/S endpoints).
- Monitor with CloudWatch metrics such as `NumberOfNotificationsFailed`, and alarm on the DLQ depth.
- Common root causes to check: a missing **queue policy** allowing the topic to send, a **KMS key policy** blocking SNS, a subscriber Lambda that’s throttled or erroring, or an endpoint that’s down.
- Once fixed, replay the DLQ messages to the subscriber.

#### Adding the loyalty service later

Only **add a new SQS queue and subscribe it to the existing topic** (with the `registered` filter and the queue policy). **Checkout is not changed or redeployed**, and nothing else is touched. Before, a new consumer meant modifying and redeploying the core checkout code, with its risk and its coupling. Now the publisher doesn’t know who listens, which is the whole point of pub/sub. The same applies to removing a service: delete its subscription.

---
## Amazon CloudWatch

A hospital’s **patient monitoring room**.

- The **vital-signs machines** (heart rate, oxygen, blood pressure) are **metrics**: numbers recorded every few seconds.
- The **doctors’ written notes** are **logs**: detailed records of what happened.
- The **dashboard on the wall** shows all patients’ vitals at a glance (**dashboards**).
- The **alarm that beeps and pages the doctor** when the heart rate drops too low is a **CloudWatch alarm**.
- The nurse who says “if the oxygen level drops, automatically start oxygen” is an **alarm action** (such as Auto Scaling).
Without monitoring, you’d find out a patient was in trouble only when they collapsed.

### Definition

**Amazon CloudWatch** is AWS’s **monitoring and observability service**. It collects **metrics, logs and events** from your AWS resources and applications, lets you **visualize** them, **alarm** on them, and **react** automatically.

#### Metrics

- A **metric** is a time-ordered set of data points, like `CPUUtilization` for an EC2 instance.
- Organized by **namespace** (like `AWS/EC2`, `AWS/SQS`), **metric name**, and **dimensions** (identifiers such as `InstanceId` that slice the metric).
- **Most AWS services publish metrics automatically.** For EC2, the default is **basic monitoring at 5-minute intervals**, and **detailed monitoring** (extra cost) gives **1-minute** data.
- **Important gap:** EC2’s default metrics cover **CPU, network and disk I/O (at the hypervisor level) and status checks**, but **not memory usage or disk space used**. To get those, install the **CloudWatch agent** on the instance.
- **Custom metrics**: your application can publish its own (for example, “orders processed per minute”), using the API (`PutMetricData`) or the agent. High-resolution custom metrics can go down to 1-second granularity.
- Metrics are retained for up to **15 months** (with the resolution reduced over time: 1-minute data is kept for 15 days, 5-minute data for 63 days, and 1-hour data for 15 months).

#### Alarms

An **alarm** watches **one metric** (or a math expression) and changes **state** based on a threshold:
- **OK**: within the threshold.
- **ALARM**: threshold breached.
- **INSUFFICIENT_DATA**: not enough data yet (or data isn’t arriving).

When it changes state, an alarm can trigger **actions**: send an **SNS notification**, run an **Auto Scaling** policy, perform an **EC2 action** (stop, terminate, reboot, recover an instance), or create a Systems Manager OpsItem/incident.

Key settings: **period** (length of each evaluation window), **evaluation periods** and **datapoints to alarm** (for example, “3 out of 5 periods” to avoid alerting on one blip), and **treat missing data** (how to handle gaps). **Composite alarms** combine several alarms with AND/OR logic to reduce noise. **Anomaly detection** alarms use a learned band of expected values instead of a fixed number.

#### Logs

- **CloudWatch Logs** collects and stores log data from applications, servers and AWS services (Lambda, API Gateway, VPC Flow Logs, CloudTrail and more).
- Structure: **log group** (a collection, usually one per application or service) → **log stream** (a sequence from a single source, like one instance or one Lambda container) → **log events** (individual entries).
- **Retention** is set per log group. The **default is “never expire”**, which keeps costing money until you change it, so setting retention is a standard good practice.
- **Metric filters** turn log patterns into metrics (for example, count the lines containing “ERROR”, then alarm on that count).
- **Logs Insights** is a query language for searching and analyzing logs interactively.
- **Subscription filters** stream logs in near real time to other destinations (Lambda, Kinesis, OpenSearch, S3 via Firehose).
- **Export to S3** for cheap long-term storage.
- **Lambda** automatically writes its output to CloudWatch Logs (when its execution role allows it), so `print()` statements in a function show up there.

#### Dashboards

Customizable pages of **graphs and numbers** across metrics, logs and alarms, and across Regions and accounts. Useful for a “single glass” view of a service’s health.

#### Events and EventBridge

**CloudWatch Events** evolved into **Amazon EventBridge**, which reacts to **events** (“an EC2 instance changed state”, “a file was uploaded”, “it’s 2 AM every day”) and routes them to targets like Lambda, SNS or SQS, including **scheduled** (cron) rules. You’ll see both names, and EventBridge is the current service.

#### Other capabilities worth knowing

- **Container Insights, Lambda Insights, Application Insights**: curated monitoring for containers, functions and applications.
- **CloudWatch Synthetics (canaries)**: scripts that act like a user and test your site or API from outside every few minutes.
- **Contributor Insights**: shows top contributors (for example, which IPs send the most requests).
- **Cross-account and cross-Region observability** features for large organizations.

## CloudWatch vs CloudTrail

![[Pasted image 20261019190329.png]]

(CloudTrail belongs to the security and governance part of the course, but know the difference now. They work well together.)

### The industry problem it solves

**Example: a ticket-booking website.**

Without monitoring:

1. The site gets slow on Friday night. Nobody knows until customers tweet about it.
2. A bug starts throwing errors in one service, but the errors are buried in log files on 20 different servers, so engineers have to log in to each one.
3. Servers run out of memory and crash, and no one noticed memory climbing for days.
4. The team can’t tell whether last week’s release made things better or worse.
5. Auto Scaling doesn’t know when to add servers.

With CloudWatch:

- **Metrics and dashboards** show request rate, latency and error rate in one place.
- **Alarms** page the on-call engineer (through SNS) within minutes when latency or error rate crosses a threshold, and trigger Auto Scaling automatically on CPU or queue depth.
- **Centralized logs** from all servers and Lambdas are searchable in one place with Logs Insights (“show all errors from the booking service in the last hour”).
- **Custom metrics** like “failed payments per minute” make business problems as visible as technical ones.
- **Synthetics canaries** detect a broken checkout page even when the servers look healthy.

**Where it is a poor fit:** deep application tracing across microservices is the job of **AWS X-Ray** or similar tools, audit of API activity is **CloudTrail**, and extremely high-volume log analytics can get expensive in CloudWatch compared with other pipelines.

## CODE

Create an alarm that notifies an SNS topic (CLI):
``` BASH
# 1. An SNS topic for alerts (Lesson 32)
aws sns create-topic --name ops-alerts
aws sns subscribe \
  --topic-arn arn:aws:sns:ap-south-1:123456789012:ops-alerts \
  --protocol email --notification-endpoint you@example.com

# 2. An alarm: average CPU above 80% for 3 of the last 5 minutes
aws cloudwatch put-metric-alarm \
  --alarm-name web-high-cpu \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0abc123def4567890 \
  --statistic Average \
  --period 60 \
  --evaluation-periods 5 \
  --datapoints-to-alarm 3 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --treat-missing-data notBreaching \
  --alarm-actions arn:aws:sns:ap-south-1:123456789012:ops-alerts
```

Publish a custom metric and write logs from Python (boto3, introduced last lesson):

```python
import boto3, time

cw = boto3.client("cloudwatch", region_name="ap-south-1")

# Custom business metric
cw.put_metric_data(
    Namespace="TicketApp",
    MetricData=[{
        "MetricName": "FailedPayments",
        "Value": 3,
        "Unit": "Count",
        "Dimensions": [{"Name": "Environment", "Value": "prod"}],
    }],
)

logs = boto3.client("logs", region_name="ap-south-1")
# (the log group and stream must exist first, via create_log_group / create_log_stream)
logs.put_log_events(
    logGroupName="/ticketapp/booking",
    logStreamName="server-1",
    logEvents=[{
        "timestamp": int(time.time() * 1000),   # milliseconds
        "message": "ERROR payment gateway timeout order=8841",
    }],
)
```

Turn log lines into a metric, and query logs:

```bash
# Metric filter: count log lines containing ERROR
aws logs put-metric-filter \
  --log-group-name /ticketapp/booking \
  --filter-name error-count \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=BookingErrors,metricNamespace=TicketApp,metricValue=1

# Set retention so logs don't pile up (and cost) forever
aws logs put-retention-policy \
  --log-group-name /ticketapp/booking --retention-in-days 30
```

- The alarm watches CPU every **60 seconds** and goes to `ALARM` only if **3 of the last 5** data points exceed 80%, which avoids alerting on a single spike. `treat-missing-data notBreaching` means gaps in data don’t trigger a false alarm.

- The **alarm action** is the SNS topic’s ARN (ARNs, as we just covered), so an email goes out. The same topic could also fan out to SMS and Lambda.

- `put_metric_data` publishes your own business number. After that, it can be graphed and alarmed like any built-in metric.

- The **metric filter** converts text logs into a numeric metric (`BookingErrors`), which can then drive an alarm. That is how “too many errors in the logs” becomes an alert.

- The **retention policy** stops logs from being kept forever.

### Interview questions with answers

**Q1. Your EC2 application is crashing because it runs out of memory, but the CloudWatch CPU, network and disk graphs all look normal, and there is no memory graph at all. Why, and what do you do?**

**Answer:** EC2’s default metrics come from the **hypervisor**, which can see CPU, network and disk I/O, but **can’t see inside the operating system**, so **memory usage and disk space used are not published by default**. The fix is to install and configure the **CloudWatch agent** on the instance (or via Systems Manager across many instances), which publishes memory, disk space, swap and other OS-level metrics as **custom metrics** (under the `CWAgent` namespace by default), and also ships log files to CloudWatch Logs. Then create alarms on memory. The instance’s IAM role needs permission to write to CloudWatch (the managed `CloudWatchAgentServerPolicy`).

**Q2. An alarm shows `INSUFFICIENT_DATA` instead of `OK` or `ALARM`. What does it mean and how do you investigate?**

**Answer:** It means CloudWatch **doesn’t have enough data points to evaluate the alarm**. Common causes: the alarm was just created and hasn’t collected data yet; the **metric isn’t being published** (the instance is stopped, the agent crashed, the application stopped sending its custom metric, or the dimensions in the alarm don’t match the dimensions of the actual metric, which is a very common mistake); or the metric is naturally sparse (for example, an error count that’s only published when errors happen). To investigate: check in the Metrics console that **data for exactly that namespace, name and dimensions exists**, check the **period** is not shorter than the metric’s publishing interval, and decide on the right **`treat-missing-data` setting** (`notBreaching`, `breaching`, `ignore` or `missing`). For sparse metrics, `notBreaching` is typical, whereas for a heartbeat metric (where silence is a problem), `breaching` is the right choice because silence should alert.

**Q3. You get paged at 3 AM several times a week by a CPU alarm that goes to ALARM for a minute and then recovers by itself. The team is suffering from alert fatigue. How do you improve it?**

**Answer:** The alarm is too sensitive. First, use **multiple evaluation periods with “M out of N datapoints”** (for example, 4 out of 5 one-minute periods) so brief spikes don’t alert. Second, check the **threshold and statistic**: average over a longer period, or use a percentile as appropriate. Third, consider **anomaly detection** alarms for metrics with daily patterns, or **composite alarms** that only fire when several conditions are true together (for example, high CPU **and** high latency, because high CPU alone with healthy latency may not need a human). Fourth, ask whether it should page a person at all: send **non-urgent alarms to email or a ticket queue** and route only **customer-impacting** alarms to the pager, and prefer **automated remediation** (Auto Scaling) for load-driven CPU. Alerts should be **actionable**, and an alarm nobody acts on should be changed or deleted.

**Q4. Your monthly CloudWatch bill jumped sharply. A review shows Logs ingestion and storage are the largest costs. What are the likely causes and the remedies?**

**Answer:** Likely causes: **log groups with “never expire” retention** accumulating years of data; **verbose DEBUG logging** left on in production; a bug that **logs in a loop** or logs huge payloads; **VPC Flow Logs or other high-volume logs** sent to CloudWatch Logs; and many custom metrics with high-cardinality dimensions (for example, a user ID as a dimension creates a separate billable metric per user). Remedies: **set retention policies** on every log group; **lower the log level** and sample noisy logs; **filter before sending**; use the **Infrequent Access log class** for logs that are rarely queried; **export old logs to S3** (cheaper long-term storage) or send high-volume logs straight to S3 or a cheaper analytics store; avoid high-cardinality dimensions; and use **Cost Explorer or the billing breakdown by usage type** (to be covered in Domain 4) to find which log groups and metrics cost the most.

**Q5. A security engineer asks: “Who deleted that security group yesterday at 2 PM?” A colleague suggests checking CloudWatch metrics. Is that the right tool?**

**Answer:** No. **CloudWatch tells you about performance and health** (metrics, application logs), while **CloudTrail records AWS API activity**: who made which API call, when, from which IP, with which identity. The right place is **CloudTrail event history** (searchable for recent management events, typically the last 90 days) or a **CloudTrail trail** delivering to S3 and optionally to CloudWatch Logs for longer retention and querying. A good setup uses both: CloudTrail logs delivered into CloudWatch Logs with a **metric filter and alarm** for risky actions (like deleting a security group or root account usage), which gives you alerting on audit events.

### Scenario question

**Scenario:** You are the cloud engineer at **FoodRush**, a food-delivery startup.

- The app runs on **EC2 instances in an Auto Scaling group** behind an ALB, writes orders to an **Aurora** database and sends tasks through an **SQS** queue to **Lambda** functions that notify restaurants.
- Last Friday at 8 PM the app became slow for 40 minutes. Nobody noticed until customers complained. Engineers then had to **SSH into 15 servers** to read log files, and found that the servers were running **out of memory**, which no graph had shown.
- The CEO wants to **know about problems before customers do**. The on-call engineer is tired of **false alarms** from short CPU spikes.
- The business team wants a graph of **“orders placed per minute”** and an alert if it **suddenly drops** (which usually means checkout is broken, even if servers look healthy).
- Developers want to **search all application errors** from all servers and Lambdas in one place, and see **how many errors per minute** there are.
- The finance team complains that **logs cost too much** because the logs have been kept forever.
- Separately, the security team wants to be alerted when **someone deletes a security group** or **logs in as the root user**.
- The Auto Scaling group should add instances automatically when the **notification queue backs up**, rather than waiting for CPU to rise.

Design the monitoring setup. For each point, name the CloudWatch features and explain your reasoning:

1. How do you get memory visibility, and what else do you collect from the servers?
2. How do you build alerts that catch real problems without false alarms?
3. How do you create and alarm on “orders per minute”, including the sudden-drop case?
4. How do you centralize and search logs, and turn errors into a graph and alert?
5. How do you control the log costs?
6. How do you set up the security alerts, and which service provides the underlying data?
7. How do you scale on the queue backlog instead of CPU?
8. How do you notice a problem that servers can’t report, such as the checkout page being broken?

Also : **which alarms would page a human at 3 AM, and which would not, and why**

#### 1. Memory visibility and other server data

Install the **CloudWatch agent** on every instance. Bake it into the **AMI or launch template** (or install it via Systems Manager), so every instance the Auto Scaling group launches reports automatically. The instance’s **IAM role** needs the `CloudWatchAgentServerPolicy` managed policy.

The agent publishes **memory used percent, disk space used, swap** and other OS-level metrics under the `CWAgent` namespace. These are the metrics that EC2’s default hypervisor-level metrics can’t see, which is why no graph showed the memory problem.

It can also ship **application and system log files** to CloudWatch Logs, which covers point 4. Also turn on **detailed monitoring (1-minute metrics)** for the instances if the faster resolution is worth the cost.

For the **ALB**, the built-in metrics worth watching are `TargetResponseTime`, `HTTPCode_Target_5XX_Count`, `HTTPCode_ELB_5XX_Count` and `UnHealthyHostCount`. For **Aurora**, watch `CPUUtilization`, `DatabaseConnections` and `FreeableMemory`. These all arrive automatically.

#### 2. Alerts that catch real problems without false alarms

- Alarm on **customer-facing symptoms** first: ALB **latency** (for example p95 or p99 `TargetResponseTime`, not just the average) and **5XX error rate**. These would have caught the Friday slowdown within minutes.
- Use **“M out of N datapoints”** (for example, 4 of 5 one-minute periods) so a short CPU spike doesn’t fire.
- Use **memory and disk alarms** from the agent metrics as **early warnings** (for example, memory above 85% for 10 minutes).
- Use **composite alarms** (high CPU **and** high latency) so a harmless CPU spike with healthy latency doesn’t alert.
- Consider **anomaly detection** for metrics with daily patterns (the lunch and dinner peaks in a food app).
- Send alarm actions to an **SNS topic** (Lesson 32), with separate topics for urgent (pager, SMS) and non-urgent (email, ticket) notifications.

CPU on its own is a weak alarm for a service that scales out automatically. If Auto Scaling handles CPU, a human doesn’t need to hear about it.

#### 3. “Orders per minute” and the sudden-drop alert

- Have the application publish a **custom metric** (`OrdersPlaced`, namespace `FoodRush`, value 1 per order, or aggregated counts) with `PutMetricData`. Alternatively, derive it from logs with a **metric filter** (for example, count log lines containing “ORDER_PLACED”). Use the **Sum statistic over 1-minute periods**. Keep dimensions low-cardinality (for example, environment), not per-user or per-order.
- **Dashboard:** put it on a CloudWatch dashboard as a line graph.
- **Sudden-drop alert**: a fixed threshold is poor, because normal volume changes between 3 AM and 8 PM. Use an **anomaly detection alarm**, which learns the expected daily and weekly band and alarms when the metric falls **below** the lower band. A good alternative is a **metric math** alarm comparing current volume with the same time last week.
- Configure **`treat-missing-data` as `breaching`** for this alarm. If checkout is so broken that no orders (and therefore no metric data points) arrive at all, silence should alarm, not be ignored. Alternatively publish an explicit zero every minute so there are always data points.

This alarm is valuable because it catches a business failure even when every server looks healthy.

#### 4. Centralizing and searching logs, and turning errors into a graph and alert

- The **agent** ships server logs, and **Lambda** automatically writes to its own log group (the execution role needs permission to write logs). Use clear **log group names** per service (for example `/foodrush/api` and `/aws/lambda/notify-restaurant`).
- Write logs as **structured JSON** (with fields like `level`, `orderId`, `service`). This makes Logs Insights queries much more powerful.
- **Search:** use **Logs Insights**, which can query **multiple log groups at once**, for example: filter `level = "ERROR"`, stats count by `bin(1m)`.
- **Errors per minute graph and alert:** create a **metric filter** on the error pattern (for example, `ERROR` or `{ $.level = "ERROR" }` for JSON), publishing a `ErrorCount` metric. Graph it on the dashboard, then create an **alarm** on it (for example, more than 20 errors per minute for 3 of 5 minutes).
- The engineers no longer need to SSH into 15 servers, and they can also trace one `orderId` across all of them.

#### 5. Controlling log costs

- **Set retention on every log group** (for example, 30 days for application logs and 7 days for debug-level logs), as the default is “never expire”.
- Use the **Infrequent Access log class** for logs that are stored but rarely queried.
- **Reduce volume at the source**: lower the log level in production (INFO or WARN, not DEBUG), avoid logging large payloads, and sample noisy logs.
- **Export or stream older logs to S3** (cheap storage, with lifecycle policies to colder classes) if they must be kept for compliance, longer than CloudWatch is economical.
- Watch for **high-cardinality metric dimensions**, which create many billable metrics.
- Use the **billing breakdown** to identify the most expensive log groups. We’ll cover cost tools in Domain 4.

#### 6. Security alerts, and the underlying data source

The data comes from **AWS CloudTrail**, which records API calls, not from CloudWatch itself. The setup:

- Make sure a **CloudTrail trail** exists (covering all Regions) and is **delivered to CloudWatch Logs**.
- Create **metric filters** on that log group:
    - Security group deletion: a filter on event name `DeleteSecurityGroup`.
    - Root login: a filter on `userIdentity.type = "Root"` and event name `ConsoleLogin`.
- Create an **alarm on each metric (count of 1 or more)** that notifies a **security SNS topic**.
- An **EventBridge rule** matching the same CloudTrail events is a common alternative that reacts to events directly, and it can also trigger automated responses (for example, a Lambda that re-creates a deleted rule or revokes a session).
- Root usage should alert **immediately and to a human**, since the root user should almost never be used.

#### 7. Scaling on the queue backlog instead of CPU

Use **Auto Scaling with target tracking** on a **backlog-per-instance** metric, which is the AWS-recommended approach:

- The raw `ApproximateNumberOfMessagesVisible` doesn’t account for how many workers exist or how fast each processes.
- Publish a custom metric: **queue depth ÷ number of running instances** (computed with metric math or a small scheduled job), and set the **target value** to the backlog each instance can clear within your acceptable delay (derived from processing time per message and the delay you can tolerate).
- Auto Scaling then adds instances when the backlog per instance rises, and removes them when it falls.

One caution: in this architecture the notification tasks are consumed by **Lambda**, which already scales with the queue automatically. Instance scaling on queue depth applies only if the queue consumers run on EC2. If the EC2 group is the web tier, scale it on **ALB request count per target** or latency-related metrics instead. Also set **maximum capacity** to protect Aurora from too many connections.

#### 8. Noticing problems that servers can’t report

Use **CloudWatch Synthetics (canaries)**: scheduled scripts that behave like a user and test **from outside** (for example, every minute: load the home page, log in, add an item to the cart, reach checkout) and alarm when the script fails or is slow. This catches problems such as a broken checkout page, an expired TLS certificate, a DNS mistake or a CDN problem, when the servers look healthy and report nothing wrong. It pairs well with the orders-per-minute anomaly alarm, because one checks technical availability and the other checks business results.

#### Which alarms page a human at 3 AM, and which do not

**Page a human (customer impact or security, and needs a person):**

- ALB 5XX error rate or latency above the agreed level for several minutes.
- Orders-per-minute anomaly (a sudden drop).
- Synthetics canary failing (checkout broken).
- Root login, or critical security changes such as deleting a security group in production.
- The DLQ receiving messages (orders or notifications are failing) if those are business-critical.
- Aurora failure or running out of connections or storage.

**Don’t page (send to email, chat or a ticket for business hours):**

- A brief CPU spike (and CPU in general when Auto Scaling handles it).
- Memory or disk **warnings** at moderate levels (early warnings), unless they’re about to cause an outage.
- Queue backlog growth that Auto Scaling or Lambda is already handling.
- Cost anomalies and log volume increases.

The principle: **page only for what is actionable now and affects customers or security**, and let automation handle what automation can. Everything else should be visible but not wake someone up.

---

## AWS CloudFormation

Building a house.

Clicking in the Console is like **giving verbal instructions to a builder, brick by brick**: “put a wall here, now a window there.” If you want an identical second house, you must remember and repeat every instruction, and you’ll probably forget something.

**CloudFormation is the architect’s blueprint.** You write down the **complete design** once: the walls, windows, plumbing and wiring, and how they connect. You hand the blueprint to a construction crew (CloudFormation), and it builds everything in the right order. Want a second identical house? Hand over the same blueprint. Want to add a room? Edit the blueprint and the crew works out what changes are needed. Want to demolish the house? The crew removes everything it built, cleanly.

### Definition

**AWS CloudFormation** is AWS’s **Infrastructure as Code (IaC)** service. You describe the AWS resources you want in a **template** (a text file in JSON or YAML), and CloudFormation **creates, updates and deletes** them for you, in the right order, as a single unit called a **stack**.

The key shift is from **“do these steps”** (imperative) to **“this is what I want to exist”** (declarative). CloudFormation works out how to get there.

## Core concepts

- **Template**: the YAML or JSON file describing resources. It’s just text, so it can be stored in Git, reviewed, versioned and reused.

- **Stack**: the **running instance of a template**: the actual set of resources CloudFormation created and manages together. Create the stack, update it, delete it, and everything goes together.

- **Resource**: one thing in the template (an S3 bucket, an EC2 instance, an SQS queue), identified by a **logical ID** you choose, and a **type** such as `AWS::S3::Bucket`. The real thing it creates has a **physical ID** (the actual bucket name or instance ID).

- **Parameters**: inputs supplied when you create the stack, so one template can build different variations (for example instance size or environment name).

- **Mappings**: fixed lookup tables inside the template (for example, AMI IDs per Region).

- **Conditions**: make parts of the template optional (for example, create a bigger database only in production).

- **Outputs**: values the stack exposes after creation (a bucket name, a URL), which can be **exported** and used by other stacks.

- **Intrinsic functions**: built-in helpers for referencing and composing values:
    - `!Ref` returns a resource’s ID (or a parameter’s value).
    - `!GetAtt` returns an attribute of a resource (such as the ARN).
    - `!Sub` substitutes variables into a string.
    - `!Join`, `!If`, `!FindInMap`, `!ImportValue` and others.

- **Pseudo parameters**: values AWS fills in automatically, such as `AWS::Region`, `AWS::AccountId` and `AWS::StackName`.

**CloudFormation itself is free** for AWS resource types. You pay only for the resources it creates (and for certain third-party or custom resource operations).

## The template sections

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: What this template builds
Parameters:      # inputs
Mappings:        # lookup tables
Conditions:      # optional logic
Resources:       # the only REQUIRED section: what to build
Outputs:         # values to show or export
```

Only **Resources** is mandatory.

#### How it works

1. You write the template.
2. You create a stack from it (Console, CLI or API). The template is uploaded (templates above a size limit must come from S3).
3. CloudFormation **builds a dependency graph** (it knows the queue must exist before the Lambda trigger that references it, because of `!Ref`) and creates resources **in parallel where possible and in order where required**.
4. If **any resource fails** to create, CloudFormation **rolls back** by default: it deletes what it had created so far, leaving no half-built mess. (You can disable rollback for debugging.)
5. The stack ends in a status such as `CREATE_COMPLETE`, or `ROLLBACK_COMPLETE` if it failed.

## Updating safely: change sets

When you modify the template and update the stack, CloudFormation compares the new template with the current state. A **change set** is a **preview** of what would happen: which resources would be added, modified or **replaced** (deleted and recreated). You review it, then execute it. Some property changes **update in place**, some cause **“some interruption”**, and some require **replacement**, which can mean downtime or **data loss** for things like databases. Reading the change set before executing is a key safety habit.

## Protecting your resources

- **DeletionPolicy**: a per-resource setting that controls what happens to the resource when the stack is deleted (or the resource is removed from the template). Values are `Delete` (default for most), `Retain` (keep the resource), and `Snapshot` (take a snapshot first, for supported resources such as RDS and EBS volumes). Use `Retain` or `Snapshot` for databases and other stateful data.
- **UpdateReplacePolicy**: the same idea for resources that are replaced during updates.
- **Stack policy**: a JSON document that blocks updates to specific resources (for example, “no one may update the production database through this stack”).
- **Termination protection**: prevents the whole stack from being deleted by accident.
- **IAM service role**: lets CloudFormation use its own permissions to create resources, so the person running it needs fewer permissions.

## Bigger designs

- **Nested stacks**: a template that includes other templates as reusable pieces (like a standard VPC module).
- **Cross-stack references**: one stack **exports** an output (for example the VPC ID), and another **imports** it (`!ImportValue`), so network, database and application layers can live in separate stacks. An export can’t be removed or changed while another stack imports it.
- **StackSets**: deploy the same stack to **multiple accounts and Regions** at once (common in large organizations).
- **Custom resources**: let a Lambda function run custom logic during stack create, update or delete, for things CloudFormation doesn’t support natively.
- **Helper scripts and `cfn-init`** for configuring EC2 instances, and **wait conditions or creation policies** so CloudFormation waits for an instance to actually finish setting up.
- **CloudFormation Registry** for third-party resource types, and **AWS CDK** (below).

## Where CDK and others fit

**AWS CDK (Cloud Development Kit)** lets you define infrastructure in a real programming language (Python, TypeScript, Java and others) and **generates CloudFormation templates** from it. CDK doesn’t replace CloudFormation, it **builds on it**, and the deployment still happens through CloudFormation stacks. **AWS SAM** is a simplified extension for serverless applications (Lambda, API Gateway, DynamoDB) that also compiles down to CloudFormation. **Terraform** is a popular third-party IaC tool that does a similar job across clouds. For the AWS exam, know that CloudFormation is AWS’s native IaC service, and that CDK uses it underneath.

### The industry problem it solves

**Example: a startup that has grown.** The team built its production environment by clicking in the Console over two years. Now:

1. **Nobody remembers exactly what’s there.** A new engineer can’t tell which security group rules were added by hand, or why.
2. **Setting up a staging environment** that matches production takes weeks of clicking, and it still differs in small, bug-causing ways.
3. **Disaster recovery**: if the Region fails or someone deletes something, rebuilding from memory is slow and error-prone.
4. **Changes are risky and unreviewed**: someone edits a firewall rule at midnight and breaks things, with no record of who changed what or how to undo it.
5. **Cleaning up** an experiment means hunting for every resource that belongs to it. Forgotten resources keep costing money.

With CloudFormation, the infrastructure is **code in Git**:

- **Repeatable**: the same template builds dev, staging and production, differing only by parameters.
- **Reviewable**: infrastructure changes go through pull requests and change sets, with a history of who changed what.
- **Recoverable**: rebuild an entire environment (or a whole new Region) in minutes by creating the stack again.
- **Clean teardown**: delete the stack and everything it created goes away, so experiments don’t leave bills behind.
- **Documentation by definition**: the template **is** the accurate description of what exists.

**Where it is a poor fit:** one-off experiments where clicking is faster, resources created and changed **outside** the template (leading to drift), data-heavy contents that don’t belong in a template (the template defines the database, but doesn’t hold its data), and teams that strongly prefer a general-purpose language (CDK) or multi-cloud tooling (Terraform).

## CODE
A small template: an SQS queue with a dead-letter queue, an SNS topic, and a subscription (YAML)

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Order events - SNS topic fanning out to an SQS queue with a DLQ

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
  MaxReceives:
    Type: Number
    Default: 3

Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  OrdersDLQ:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: !Sub "orders-dlq-${Environment}"
      MessageRetentionPeriod: 1209600          # 14 days

  OrdersQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: !Sub "orders-queue-${Environment}"
      VisibilityTimeout: !If [IsProd, 300, 60]
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt OrdersDLQ.Arn
        maxReceiveCount: !Ref MaxReceives

  OrderTopic:
    Type: AWS::SNS::Topic
    Properties:
      TopicName: !Sub "order-placed-${Environment}"

  QueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues: [!Ref OrdersQueue]
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal: { Service: sns.amazonaws.com }
            Action: sqs:SendMessage
            Resource: !GetAtt OrdersQueue.Arn
            Condition:
              ArnEquals: { "aws:SourceArn": !Ref OrderTopic }

  Subscription:
    Type: AWS::SNS::Subscription
    Properties:
      TopicArn: !Ref OrderTopic
      Protocol: sqs
      Endpoint: !GetAtt OrdersQueue.Arn
      RawMessageDelivery: true

Outputs:
  TopicArn:
    Value: !Ref OrderTopic
    Export:
      Name: !Sub "${AWS::StackName}-TopicArn"
  QueueUrl:
    Value: !Ref OrdersQueue

```

## Deploying and managing it(CLI):

```bash
# Create the stack
aws cloudformation create-stack \
  --stack-name orders-dev \
  --template-body file://orders.yaml \
  --parameters ParameterKey=Environment,ParameterValue=dev

# Watch progress
aws cloudformation describe-stack-events --stack-name orders-dev

# Preview an update safely with a change set
aws cloudformation create-change-set \
  --stack-name orders-dev --change-set-name add-retries \
  --template-body file://orders.yaml \
  --parameters ParameterKey=MaxReceives,ParameterValue=5
aws cloudformation describe-change-set \
  --stack-name orders-dev --change-set-name add-retries
aws cloudformation execute-change-set \
  --stack-name orders-dev --change-set-name add-retries

# Detect manual changes
aws cloudformation detect-stack-drift --stack-name orders-dev

# Delete everything the stack created
aws cloudformation delete-stack --stack-name orders-dev
```

- `Parameters` and `Conditions` let one template build `dev`, `staging` and `prod` versions (a longer visibility timeout in production).

- `!GetAtt OrdersDLQ.Arn` uses the DLQ’s **ARN**, and this reference tells CloudFormation to **create the DLQ before the main queue**. You never write the order yourself, because CloudFormation reads the dependencies from the references. (When there is no reference but order still matters, you add a `DependsOn`.)

- `!Sub "orders-queue-${Environment}"` builds names that differ per environment, so two stacks don’t collide. Hard-coding a queue name would make a **second stack fail**, because names must be unique.

- The **queue policy** and subscription from the SNS lesson are now code, so the “forgot the policy, messages silently don’t arrive” mistake can’t happen in a new environment.

- `Outputs` with `Export` make the topic ARN available to other stacks through `!ImportValue`.

- The **change set** shows exactly what an update would change before anything happens. Here, changing `MaxReceives` modifies the queue’s redrive policy in place, whereas changing certain properties (like a queue’s `FifoQueue` setting or, for some resources, the explicit name) would **replace** it.

- `delete-stack` removes everything cleanly, and this is why CloudFormation is such a good way to avoid forgotten, billing resources.

### Interview questions  with answers

**Q1. You update a stack to change a property on your RDS database, and the update succeeds, but the production database is gone, along with its data. What happened, and how do you prevent it?**

**Answer:** Some property changes require **replacement**, meaning CloudFormation **creates a new resource and deletes the old one**. For a database, the old one (and its data) is deleted unless the old one is protected. To prevent it: always **review a change set** before executing an update, and look for `Replacement: True` on stateful resources. Set **`DeletionPolicy: Snapshot` (or `Retain`) and `UpdateReplacePolicy: Snapshot`** on databases and other stateful resources, so a replacement or deletion takes a snapshot first. Apply a **stack policy** that denies updates to the production database, enable **termination protection**, and keep **automated backups** on. Also avoid hard-coding **explicit physical names** on resources that may need replacement, because a replacement with a custom name fails (the new one can’t be created while the old name still exists), and keep data in a separate stack from application resources so the risky layer changes less often.

**Q2. A teammate fixed a production problem by editing a security group in the Console. Next week a stack update runs and the fix disappears, so production breaks again. Explain why, and what the team should do.**

**Answer:** CloudFormation treats the **template as the source of truth**. The manual console change created **drift**: the real resource no longer matched the template. A later update may overwrite the resource back to the template’s definition (or not touch it, depending on whether that property changed), which makes the outcome unpredictable. The right process: use **drift detection** to find manual changes regularly, **make the same fix in the template** (reviewed in Git) and deploy it through the stack, and restrict console write access in production (through IAM) so changes go through the pipeline. In an emergency, a manual fix is acceptable, but it must be **backported into the template** immediately afterward.

**Q3. Stack creation fails at the 12th resource out of 20. What happens to the 11 resources created so far, how do you find out why it failed, and when would you turn off the default behavior?**

**Answer:** By default CloudFormation **rolls back**: it deletes the resources it created, and the stack ends in `ROLLBACK_COMPLETE` (a stack in that state cannot be updated and must be deleted before you try again). To find the cause, read the stack’s **Events** tab: the failed resource shows a status reason (for example, insufficient permissions, a name collision, a quota limit, or an invalid property). To debug difficult failures, you can create the stack with **rollback disabled** (`--on-failure DO_NOTHING`, or disable rollback in the Console), which leaves the created resources in place so you can inspect them (for example, to read an EC2 instance’s setup logs). Clean up afterward, since those resources cost money. Use this only in development, never for production stacks.

**Q4. You need the same architecture in 15 accounts and 3 Regions. A colleague proposes running the create-stack command 45 times. What’s better?**

**Answer:** Use **CloudFormation StackSets**, which deploys a single template to **multiple accounts and Regions** from one place, with controls for how many deploy at once and how many failures to tolerate. With AWS Organizations, it can use **service-managed permissions** and even **automatically deploy to new accounts** that join an organizational unit. Updates also roll out through the StackSet in one operation. For reusable building blocks inside one stack hierarchy, **nested stacks** or modules help, and for configuration that must differ per account or Region, pass **parameter overrides** per target.

**Q5. What is the difference between CloudFormation and the AWS CDK, and between CloudFormation and just writing a CLI script that creates the resources?**

**Answer:** **CDK is not a replacement for CloudFormation, it generates it.** You write infrastructure in a general-purpose language (loops, functions, classes, type checking), and CDK **synthesizes a CloudFormation template** that is deployed as a normal stack. So you get programming-language convenience with CloudFormation’s deployment features. Against a **CLI script**: a script is **imperative**, running commands in order, and it has no memory of what exists, so it can’t compute a diff for updates, can’t roll back on failure, can’t detect drift, and can’t delete everything it created cleanly, and re-running it often fails with “already exists” errors. CloudFormation is **declarative and stateful**: it tracks the stack, computes changes, orders dependencies, rolls back failures, offers change sets and drift detection, and deletes cleanly. Terraform fills a similar role as CloudFormation, with multi-cloud support and its own state management.

## Scenario question

**Scenario:** You are the cloud engineer at **LearnSphere**, an online-course platform that has been running in AWS for two years, built mostly by clicking in the Console.

- The **production** environment consists of a VPC, an ALB, an Auto Scaling group of EC2 instances, an Aurora database, an SQS queue with a DLQ, an SNS topic, and CloudWatch alarms.
- The CTO wants a **staging** environment that is **identical to production except smaller**, and a **dev** environment for each developer, created in minutes and **deleted** every Friday to save money.
- Last month, an engineer **manually edited a security group in production** to fix an incident, and nobody wrote it down. Last week, someone **deleted the production Aurora database by accident** while “cleaning up”.
- The company is expanding to a **second Region** for disaster recovery, and wants the same setup there.
- The networking team owns the **VPC**, the application team owns the **app**, and the data team owns the **database**. Each team wants to change its own part **without touching the others**, but the app needs to know the VPC and subnet IDs.
- Before every update, the CTO wants to **see exactly what would change**, and she wants **production database data to survive** even in a mistaken update or delete.
- A new developer asks: “Why can’t I just write a Python script with boto3 calls to build all this?”

Design the CloudFormation approach. For each point, name the features and explain your reasoning:

1. How do you describe the existing environment as code, and how do you create staging and dev from the same definition?
2. How do you structure the stacks for the three teams, and how does the app stack get the VPC and subnet IDs?
3. How do you stop accidental deletion of the Aurora database, and make sure data survives a mistaken change?
4. How do you review changes before applying them?
5. How do you handle manual changes like the security group edit?
6. How do you deploy to the second Region?
7. How do you make the Friday dev-environment cleanup safe and automatic?
8. How would you answer the new developer’s boto3 question?

Also : **how would you bring the already-existing, manually created production resources under CloudFormation management without recreating them?**

#### 1. Describing the environment as code, and creating staging and dev from it

Write the architecture as **CloudFormation templates in YAML**, stored in Git.

- Use **Parameters** for what differs between environments: `Environment`, instance type, database instance class, and Auto Scaling min and max.
- Use **Conditions** for production-only features, such as Multi-AZ, extra replicas and deletion protection.
- Keep **one template and one small parameter file per environment**. Don’t copy templates, because copies drift apart, which is the staging-differs-from-prod problem you started with.
- **Staging** is the same template with smaller parameters.
- **Dev** is the same template with a stack name like `dev-<developer>` and tiny sizes. Name resources with `!Sub "...-${Environment}"`, so stacks never collide.

#### 2. Structuring stacks for three teams, and sharing VPC and subnet IDs

Use **layered stacks**, one per ownership boundary:

- **Network stack** (networking team): VPC, subnets, route tables, gateways.
- **Data stack** (data team): Aurora cluster, DB subnet group, database security group.
- **App stack** (application team): ALB, Auto Scaling group, queue, DLQ, topic, alarms.

Each team changes its own stack without touching the others. The **Network stack exports** the VPC ID and subnet IDs as **Outputs with `Export`**, and the app stack reads them with **`!ImportValue`**. Creation order is network, then data, then app. Each team’s IAM permissions can be limited to its own stacks.

One trade-off to know: an export **can’t be changed or deleted while another stack imports it**. Many teams avoid that lock by publishing values to **SSM Parameter Store** and reading them as stack parameters. For reusable pieces (a standard VPC module), **nested stacks** are another option.

#### 3. Preventing accidental deletion of Aurora, and protecting data

Layer several protections, because each covers a different failure:

- **`DeletionProtection: true`** on the Aurora cluster. This blocks **any** deletion, including a manual console delete, until someone deliberately disables it. It directly addresses last week’s accident.
- **`DeletionPolicy: Snapshot`** (or `Retain`) and **`UpdateReplacePolicy: Snapshot`** on the database resources. These apply when **CloudFormation** deletes or replaces the resource, so a mistaken stack update or stack deletion takes a snapshot first. They do **not** stop a manual console deletion, which is why `DeletionProtection` is also needed.
- **Stack termination protection** on the data stack.
- A **stack policy** denying updates that replace or delete the database resource.
- **IAM**: deny `rds:DeleteDBCluster` and `rds:DeleteDBInstance` for everyone except a break-glass role.
- Keep **automated backups** on with a sensible retention period, and keep the database in its **own stack** so the risky app layer changes more often than the data layer.

#### 4. Reviewing changes before applying them

Use **change sets**. A pull request triggers the creation of a change set (or `aws cloudformation deploy --no-execute-changeset`), and the CTO or reviewer reads the **list of Add, Modify and Remove actions**. She specifically looks for **`Replacement: True`** on stateful resources. Only after approval does the pipeline **execute** the change set. Add **`cfn-lint`** in CI to catch template mistakes before they reach AWS, and never deploy to production from a laptop.

#### 5. Handling manual changes like the security group edit

- Run **drift detection** on a schedule (and before each update). It reports resources whose real configuration differs from the template.
- **Put the fix into the template** through a pull request, so the change is reviewed and recorded. In an emergency, a manual fix is acceptable, but it must be **backported into the template** the same day.
- **Reduce the temptation**: give people read-only console access in production, with a **break-glass role** for incidents, so manual changes are rare and audited (by CloudTrail).
- Honest caveat: drift detection doesn’t cover every resource type or property, so it’s a safety net, not a guarantee.

#### 6. Deploying to the second Region

Run the **same templates** in the new Region, after removing Region-specific hard-coding:

- Use `!Ref AWS::Region`, `Fn::GetAZs` and `!Sub` instead of fixed values.
- AMI IDs differ per Region, so use a **Mapping** or an **SSM parameter** that resolves the latest AMI for each Region.
- **IAM is global** and **S3 bucket names are global**, so hard-coded names for those collide. Let CloudFormation generate names, or include the Region in the name.
- For many accounts or Regions at once, use **StackSets**. For two Regions, running the pipeline twice is enough.

CloudFormation builds the **infrastructure** only. It doesn’t copy the **data**. For disaster recovery, add **Aurora Global Database** or cross-Region snapshot copies (Lesson 26), and **Route 53 failover** (Lesson 28). Be clear with the business about which DR style they’re paying for: a fully running second Region, or a smaller “pilot light” that scales up during a disaster.

#### 7. Making the Friday dev cleanup safe and automatic

- Tag every dev stack (`Environment=dev`, `Ephemeral=true`) and name them `dev-<developer>`.
- An **EventBridge scheduled rule** (Friday evening) triggers a **Lambda function** that lists stacks with that tag and calls **`delete-stack`**. Send a summary to an **SNS topic**.
- **Safety**: the Lambda’s IAM role is allowed to delete **only** stacks matching the dev name prefix or tag, so it can never touch production, even if there’s a bug.
- **Avoid `Retain`** on dev resources, so deletion is complete and leaves no billing leftovers. Drive `DeletionProtection` from a parameter so it’s off in dev and on in production.
- Know the common failure: an **S3 bucket that still contains objects** (or a stack whose export is still imported by another stack) makes deletion fail with `DELETE_FAILED`. Plan for it with a cleanup step, and have the cleanup report failures.
- Lower the frequency of surprises by letting developers extend a stack’s life with a tag (for example `KeepUntil`).

#### 8. Answer to the new developer’s boto3 question

You can use boto3 to create resources, but a script is **imperative and has no memory**:

- It doesn’t know what already exists, so re-running it produces “already exists” errors.
- It can’t compute a **diff** for updates, so there’s no change-set-style preview.
- It doesn’t **roll back** a half-finished run, and has no **drift detection**.
- It doesn’t order dependencies for you, and **cleanup** needs you to track every resource you created.

CloudFormation is **declarative and stateful**: you describe what should exist, and it works out creating, ordering, updating, rolling back and deleting. If the developer wants to write **Python**, the right tool is the **AWS CDK**, which lets you define infrastructure in Python and **generates CloudFormation** underneath. Boto3 remains excellent for **application code** that talks to AWS (publishing messages, reading objects) and for small automation, just not for owning infrastructure state.

#### Bringing existing, manually created resources under CloudFormation without recreating them

Use **resource import**:

1. Write a template whose resource definitions **match the current real configuration** exactly, and give every resource to be imported a **`DeletionPolicy: Retain`** (required for import).
2. Create an **import change set**, identifying each existing resource by its real identifier (for example the cluster identifier or bucket name).
3. Execute it. CloudFormation **adopts** the resources into the stack without creating, changing or deleting them.
4. Run **drift detection** straight afterward. If the template didn’t match reality, it will show differences. Fix the template (or the resource) until the stack is clean.

The **IaC generator** can scan the existing account and draft templates to speed up step 1. Do it **layer by layer** (network first, database last), rehearse in a copy of the environment first, and remember that **not every resource type supports import**.

---
## AWS Secrets Manager

- **Secret**: any value that must stay private because it grants access: a database password, an API key, an OAuth token, a private certificate key.

- **IAM roles**: how an AWS service gets permission without storing keys. A Lambda function or EC2 instance **assumes a role** and gets temporary credentials.

- **KMS:** AWS’s Key Management Service. It holds encryption keys, and other services use it to encrypt data. An **AWS managed key** is created and managed for you, and a **customer managed key (CMK)** is one you create and control, with your own key policy.

- **Aurora**: earlier we created a cluster with `--manage-master-user-password`, which makes AWS store the database password in Secrets Manager. Now you’ll see where it goes.

- **CloudFormation**: infrastructure as code, which can create secrets and refer to them.

- **CloudTrail:** the record of who called which AWS API.

