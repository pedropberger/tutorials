# Basic Security Elastic Search Cluster Tutorial

This tutorial is an adaptation of my step-by-step guide to deploying a small Elastiscsearch cluster with basic security. The nodes will authenticate each other using certificates, but you will only be able to manipulate the indexes with a username and password. The installation of Kibana is not included in this tutorial.

We'll set up a 3-node cluster where each node can be a master and hold data. Remember, **this is a *basic* secure setup**. Real-world production might need more bells and whistles (dedicated master nodes, coordinating nodes, more robust security, backups, monitoring, etc.), but this gets you started!

**Assumptions:**

*   You have 3 VMs ready (let's call them VM1, VM2, VM3).
*   They are running a `dnf`-based Linux distro (like Fedora, RHEL, CentOS stream).
*   They can reach each other over the network (check firewalls *between* VMs if needed).
*   You have `sudo` access on all of them.
*   You know the IP addresses for each VM. We'll use placeholders like `YOUR_NODE_1_IP`, `YOUR_NODE_2_IP`, `YOUR_NODE_3_IP`. **Replace these!**

Let's go!

---

### Phase 1: Setting Up Each Node (Repeat with tweaks for VM2 & VM3)

We'll do VM1 first, then the others are mostly rinse-and-repeat with minor changes.

**On VM1 (Your `master-node-1`)**

1.  **Install Java:** Elasticsearch needs Java. Let's grab OpenJDK 17.
    ```bash
    sudo dnf install java-17-openjdk-devel -y
    ```
    Check if it worked:
    ```bash
    java -version
    ```
    You should see Java 17 output. Cool.

2.  **Add Elasticsearch Repo:** We need to tell `dnf` where to find Elasticsearch.
    Import the GPG key first (like a stamp of approval):
    ```bash
    sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
    ```
    Now, create the repo file. This neat `tee` command writes the config directly:
    ```bash
    sudo tee /etc/yum.repos.d/elasticsearch.repo << EOF
    [elasticsearch]
    name=Elasticsearch repository for 8.x packages
    baseurl=https://artifacts.elastic.co/packages/8.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    EOF
    ```

3.  **Find Your IP:** You'll need this for the config later.
    ```bash
    hostname -I
    ```
    Note down the main IP address for VM1 (e.g., `10.0.101.49`). This is `YOUR_NODE_1_IP`.

4.  **Install Elasticsearch:** Let's get the specific version you used.
    ```bash
    sudo dnf install elasticsearch-8.17.3-1.x86_64 -y
    ```
    **>>> SUPER IMPORTANT <<<**
    During the install, it will spit out a password for the `elastic` superuser. **COPY THIS PASSWORD AND SAVE IT SOMEWHERE SAFE!** You *will* need it. If you miss it, you'll have to reset it later (which is a pain).

5.  **(Optional) Backup the Config:** Always good practice before messing with configs.
    ```bash
    sudo mkdir -p /opt/bkp
    sudo cp /etc/elasticsearch/elasticsearch.yml /opt/bkp/elasticsearch.yml.bak
    ```

6.  **Configure Elasticsearch (`elasticsearch.yml`):** Time to tell this node about the cluster. Your `sed` commands are slick for automating this!
    *Remember to replace the example IPs (`10.0.101.49`, `10.0.101.50`, `10.0.101.51`) with the *actual* IPs of your three VMs.*

    ```bash
    # Set cluster name
    sudo sed -i '/cluster.name:/c\cluster.name: elastic-cluster-dev' /etc/elasticsearch/elasticsearch.yml

    # Set node name (unique for this VM)
    sudo sed -i '/node.name:/c\node.name: master-node-1' /etc/elasticsearch/elasticsearch.yml

    # Bind to all network interfaces (0.0.0.0 makes it listen on all available IPs)
    # Fine for internal networks, but be careful if any interface is public!
    sudo sed -i '/network.host:/c\network.host: 0.0.0.0' /etc/elasticsearch/elasticsearch.yml

    # Tell it where to find other potential masters (use YOUR actual IPs!)
    sudo sed -i '/discovery.seed_hosts:/c\discovery.seed_hosts: ["YOUR_NODE_1_IP", "YOUR_NODE_2_IP", "YOUR_NODE_3_IP"]' /etc/elasticsearch/elasticsearch.yml

    # Define the initial set of master-eligible nodes for the *first* time the cluster forms
    # Remove any existing/commented line first, then add the new one
    sudo sed -i '/^ *#* *cluster\.initial_master_nodes:/d' /etc/elasticsearch/elasticsearch.yml
    sudo sed -i '$ a cluster.initial_master_nodes: ["master-node-1", "master-node-2", "master-node-3"]' /etc/elasticsearch/elasticsearch.yml

    # Define roles for this node (can be master, can hold data)
    sudo sh -c 'echo "node.roles: [master, data]" >> /etc/elasticsearch/elasticsearch.yml'
    ```

7.  **Initial Security State (Important Note):** Your commands mention disabling security (`xpack.security.enabled: false` and `xpack.security.transport.ssl.enabled: false`) using `vi`. However, Elasticsearch 8+ has security **enabled by default**, and it's generally recommended to keep it that way. Your later steps *do* set up certificates, which *requires* security to be enabled.

    *   `xpack.security.enabled: true` (This is the default, you usually don't need to add it unless it was explicitly set to `false` before). This enables authentication (like the `elastic` user/password) and authorization.
    *   `xpack.security.transport.ssl.enabled: true` (You'll enable this *later* after creating certs). This encrypts communication *between* the nodes.

    **For now, let's assume you *don't* manually disable `xpack.security.enabled`.** We will enable `xpack.security.transport.ssl.enabled` in Phase 2. If you open the file (`sudo vi /etc/elasticsearch/elasticsearch.yml`), just ensure these security lines aren't set to `false`. If they are commented out or missing, that's fine (defaults are `true`).

8.  **Configure Firewall:** Open the necessary ports. Elasticsearch uses 9200 for HTTP traffic (REST API) and 9300 for internal node-to-node communication.
    ```bash
    sudo firewall-cmd --add-port=9200/tcp --permanent
    sudo firewall-cmd --add-port=9300/tcp --permanent
    sudo firewall-cmd --reload
    ```

**Phew! Node 1 basics done.** Now for the other two.

---

**On VM2 (Your `master-node-2`)**

Repeat steps 1-8 from above, but with these key differences in **Step 6 (Configure Elasticsearch)**:

*   Change the `node.name`:
    ```bash
    sudo sed -i '/node.name:/c\node.name: master-node-2' /etc/elasticsearch/elasticsearch.yml
    ```
*   Make sure `discovery.seed_hosts` and `cluster.initial_master_nodes` use the **same correct IPs and node names** as on VM1. Your `sed` commands should already handle this if you run them again, just double-check the IPs you put in the `discovery.seed_hosts` line!
*   Don't forget to run the firewall commands (Step 8) on this machine too!
*   **Remember to grab the `elastic` password generated during the install on *this* node too!** It *should* be the same if installed close together, but best to check. You'll only really need the first one you saved, though.

**On VM3 (Your `master-node-3`)**

Repeat steps 1-8 again, changing **Step 6 (Configure Elasticsearch)**:

*   Change the `node.name`:
    ```bash
    sudo sed -i '/node.name:/c\node.name: master-node-3' /etc/elasticsearch/elasticsearch.yml
    ```
*   Again, ensure `discovery.seed_hosts` and `cluster.initial_master_nodes` are consistent across all nodes.
*   Run the firewall commands (Step 8).
*   Note the `elastic` password just in case (again, you likely only need the first one).

---

### Phase 2: Initial Startup (Let's see if they talk!)

**On ALL THREE NODES (VM1, VM2, VM3):**

1.  **Start and Enable Elasticsearch:**
    ```bash
    sudo systemctl start elasticsearch
    sudo systemctl enable elasticsearch # Makes it start on boot
    ```

2.  **Check Status:**
    ```bash
    sudo systemctl status elasticsearch
    ```
    Look for `active (running)`. If not, check the logs (`sudo journalctl -u elasticsearch`).

3.  **Basic Cluster Check (Run from any node):**
    Use the `elastic` username and the password you saved earlier. The `-k` flag ignores certificate checks (since we haven't fully set up HTTPS certs for the HTTP layer yet, only basic auth is on).
    ```bash
    # Replace YOUR_SAVED_ELASTIC_PASSWORD with the actual password!
    curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD http://localhost:9200
    ```
    You should get a JSON response with cluster info. Now check the cluster health:
    ```bash
    curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD http://localhost:9200/_cluster/health?pretty
    ```
    Ideally, the `status` should be `green`, and `number_of_nodes` should be `3`. It might be `yellow` initially if it's still sorting itself out, which is okay for now.

    Let's see the nodes:
    ```bash
    curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD "http://localhost:9200/_cat/nodes?v"
    ```
    You should see all three nodes (`master-node-1`, `master-node-2`, `master-node-3`) listed.

If they all see each other, great! If not, double-check IPs in `elasticsearch.yml`, firewall rules, and network connectivity between the VMs.

---

### Phase 3: Securing Node-to-Node Communication (Transport Layer TLS)

Okay, the nodes can talk, but that chatter on port 9300 isn't encrypted yet. Let's fix that. We'll generate certificates on Node 1 and distribute them.

**On VM1 (master-node-1):**

1.  **Navigate to Elasticsearch Directory:**
    ```bash
    cd /usr/share/elasticsearch/
    ```

2.  **Create Certificate Authority (CA):** This is like the main stamp maker.
    ```bash
    ./bin/elasticsearch-certutil ca
    ```
    It will ask for an output filename (press Enter for default: `elastic-stack-ca.p12`) and a password. **Choose a strong password and remember it!** Let's call this the `CA_PASSWORD`.

3.  **Generate Node Certificates:** Use the CA to create certs for the nodes.
    ```bash
    ./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
    ```
    It will ask for the CA password (`CA_PASSWORD`) you just created. Then, it asks for an output filename (press Enter for default: `elastic-certificates.p12`) and a password for *this* certificate file. **Choose another strong password and remember it!** Let's call this the `CERT_PASSWORD`. This single file contains the necessary certs for all nodes in this basic setup.

4.  **Copy Certificate to Config Directory:**
    ```bash
    sudo cp /usr/share/elasticsearch/elastic-certificates.p12 /etc/elasticsearch/certs/
    ```
    *(Note: The default install might already create a `certs` directory. If not, `sudo mkdir /etc/elasticsearch/certs` first)*

5.  **Set Certificate Permissions:** We need to make sure Elasticsearch can read it, but keep it secure.
    ```bash
    cd /etc/elasticsearch/
    sudo chown root:elasticsearch certs/elastic-certificates.p12
    sudo chmod 660 certs/elastic-certificates.p12
    ```
    Check the permissions:
    ```bash
    sudo ls -l certs/
    ```
    You should see something like `-rw-rw----. 1 root elasticsearch ... elastic-certificates.p12`.

6.  **Store Certificate Password Securely:** Instead of putting the password in `elasticsearch.yml`, we add it to the Elasticsearch keystore.
    ```bash
    cd /usr/share/elasticsearch/

    # Add the password for the node certificate file (CERT_PASSWORD)
    # It will prompt you to enter the CERT_PASSWORD twice
    sudo ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    sudo ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ```
    *(Yes, we add the same password for both keystore and truststore settings when using a single cert file like this).*

7.  **Copy Certificate to Other Nodes:** Use `scp` to securely copy the generated `elastic-certificates.p12` file to VM2 and VM3. Replace `YOUR_USER` with your username on the remote VMs and adjust hostnames/IPs as needed.
    ```bash
    # Example using hostname - adjust YOUR_USER@HOSTNAME as needed
    sudo scp /etc/elasticsearch/certs/elastic-certificates.p12 YOUR_USER@master-node-2:/tmp/elastic-certificates.p12
    sudo scp /etc/elasticsearch/certs/elastic-certificates.p12 YOUR_USER@master-node-3:/tmp/elastic-certificates.p12

    # OR Example using IP - adjust YOUR_USER@IP as needed
    # sudo scp /etc/elasticsearch/certs/elastic-certificates.p12 YOUR_USER@YOUR_NODE_2_IP:/tmp/elastic-certificates.p12
    # sudo scp /etc/elasticsearch/certs/elastic-certificates.p12 YOUR_USER@YOUR_NODE_3_IP:/tmp/elastic-certificates.p12
    ```
    You'll likely be prompted for your user's password on the destination VMs.

**On VM2 and VM3:**

1.  **Move Certificate and Set Permissions:**
    ```bash
    # Move from /tmp to the correct location
    sudo cp /tmp/elastic-certificates.p12 /etc/elasticsearch/certs/elastic-certificates.p12

    # Set ownership and permissions (crucial!)
    sudo chown root:elasticsearch /etc/elasticsearch/certs/elastic-certificates.p12
    sudo chmod 660 /etc/elasticsearch/certs/elastic-certificates.p12

    # Clean up the temp file
    rm /tmp/elastic-certificates.p12
    ```

2.  **Add Certificate Password to Keystore:** Repeat the keystore commands on VM2 and VM3. Use the **same `CERT_PASSWORD`** you created on VM1.
    ```bash
    # It will prompt you for the CERT_PASSWORD
    sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ```

**On ALL THREE NODES (VM1, VM2, VM3):**

1.  **Enable Transport SSL in `elasticsearch.yml`:** Now we tell Elasticsearch to actually *use* these certificates for node-to-node communication. Edit the config file:
    ```bash
    sudo vi /etc/elasticsearch/elasticsearch.yml
    ```
    Add these lines at the end (or uncomment and configure if they exist):
    ```yaml
    # Enable transport SSL
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.verification_mode: certificate
    xpack.security.transport.ssl.client_authentication: required
    xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
    xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
    ```
    Save and close the file (`:wq` in vi).

---

### Phase 4: Restart and Final Verification

**On ALL THREE NODES (VM1, VM2, VM3):**

1.  **Restart Elasticsearch:** Apply the new security settings.
    ```bash
    sudo systemctl stop elasticsearch
    sudo systemctl start elasticsearch
    ```

2.  **Check Status:** Give it a moment to start up, then check:
    ```bash
    sudo systemctl status elasticsearch
    ```
    Again, look for `active (running)`. Check logs (`sudo journalctl -u elasticsearch`) if there are issues. Problems here often relate to incorrect certificate passwords in the keystore or permission issues on the cert file.

**Final Test (Run from any node):**

Let's run those checks again. Use the `elastic` user and the password you saved way back at the beginning (`YOUR_SAVED_ELASTIC_PASSWORD`).

```bash
# Basic check
curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD http://localhost:9200

# Cluster health check
curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD http://localhost:9200/_cluster/health?pretty

# Node list check
curl -k -u elastic:YOUR_SAVED_ELASTIC_PASSWORD "http://localhost:9200/_cat/nodes?v"
```

**Expected Result:**

You should get successful JSON responses. The `_cluster/health` status should ideally be `green` with `number_of_nodes: 3`. The `_cat/nodes` command should list all three of your nodes.

If you see all three nodes and the cluster health is green (or maybe yellow if you have no data yet, which is fine), then **Congratulations!** You've successfully set up a basic, multi-node Elasticsearch cluster with encrypted transport between the nodes using your command list as a base!

---

Remember, this is just the start. You'd typically want Kibana for visualization, maybe Logstash or Beats for data ingestion, proper backups, monitoring, and potentially more advanced security configurations for a true production system. But hey, you've got the core cluster running securely! Good job!
