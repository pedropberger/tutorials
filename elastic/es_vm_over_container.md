## Why I Deploy Elasticsearch on VMs Instead of Docker or Kubernetes (And I’m Okay With That)

Let me start by saying—I **love containers**. Docker is amazing, Kubernetes is powerful. But when it comes to deploying **Elasticsearch in production**, I still stick with good ol’ **virtual machines**. Yeah, VMs. No fancy orchestrator. No Helm charts. Just raw compute.

Before you come at me with YAML files, let me walk you through **why** this decision actually makes a lot of sense—especially in real-world, high-stakes environments like the one I deal with.

### 1. Elasticsearch Hates Being Bossed Around

Elasticsearch is a **stateful beast**. It wants to manage its own memory, storage, and cluster lifecycle. Stick it in a container, and suddenly it’s got some orchestrator trying to reschedule it, restart it, or move it mid-operation. Not cool.

With VMs, I give each node its own space, tuned and tailored for what Elasticsearch actually needs. No surprises. No rescheduling nightmares.

### 2. Predictability = Stability

On VMs, I know **exactly** what I’m getting. CPU, RAM, storage—I define it all upfront and monitor it closely. That kind of predictability makes it a lot easier to troubleshoot and optimize.

Kubernetes environments are more dynamic by design, which is great for stateless microservices, but Elasticsearch? It’s more like that heavyweight backend service that just wants peace and quiet to do its job.

### 3. Simpler Networking (Seriously)

Getting a basic Elasticsearch cluster talking across nodes on raw VMs is **way easier** than dealing with Kubernetes networking policies, persistent volumes, service discovery, sidecars, and so on.

Sometimes simpler is better. Especially when something goes wrong and you want to SSH into a box and figure it out without digging through 50 layers of abstraction.

### 4. Disk I/O and Performance

Elasticsearch is **very sensitive to disk performance**, especially for indexing-heavy workloads. Running it in containers can sometimes introduce storage layers that reduce IOPS or introduce latency—depending on how your volumes are mounted or what storage class is used in Kubernetes.

With VMs, I can mount dedicated SSDs or NVMe drives directly and configure disk settings specifically for Elasticsearch. No intermediary storage plugins. Just fast, reliable I/O.

### 5. Easier Maintenance in Smaller Teams

Let’s be honest—Kubernetes adds a **ton of complexity**. You need to manage manifests, operators, sidecars, storage classes, ingress, rolling updates, and more. It’s powerful, but it’s also a lot to maintain, especially for a **small ops/dev team** or when you're wearing multiple hats (like I do).

With VMs, maintenance is more straightforward. I update the cluster nodes manually or via Ansible scripts, snapshot data to S3, monitor health via Elastic’s APIs, and I can sleep well knowing that nothing’s going to roll out automatically while I’m not looking.

### 6. It Just Works (for Now)

Could I move to Kubernetes later? Sure. If I had a dedicated SRE team, a rock-solid operator setup, and the time to refactor everything. But right now? VMs are battle-tested, stable, and they do the job **really well**.

Production is about what works. And for my Elasticsearch stack, **VMs work**.

---

### Final Thoughts

I’m not saying containers are bad for Elasticsearch. For development or quick PoCs, Docker is awesome. And if you’ve got a mature K8s setup with operators like ECK (Elastic Cloud on K8s), then you might be living the dream.

But for me, in production, where Elasticsearch is critical infrastructure—I keep it simple, stable, and under control. And that means giving each node its own VM and letting it do what it does best.

By the way, [here you can follow my step by step](https://github.com/pedropberger/tutorials/blob/main/elastic/es_basic_security_cluster.md) to deploy a ElasticSearch micro cluster using VMs with basic security.

---

If you're curious about how I set up my VMs, tune Elasticsearch settings, or monitor the cluster, let me know—I’ve got notes, scripts, and scars to share.
