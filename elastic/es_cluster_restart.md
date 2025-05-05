## How to Make Your Elasticsearch Cluster Restart Automatically: A Lesson Learned the Hard Way

Hey, so picture this: it’s a typical Monday morning, and I’m sipping my coffee, ready to dive into some development work. Little did I know, my Elasticsearch cluster had decided to take an unscheduled nap over the weekend, and I was none the wiser. Yep, it crashed, and since it was my development environment, I didn’t have any fancy monitoring set up to alert me. So, I was blissfully unaware until I openned my Teams and saw 5 chats from coworkers asking about his queries. Cue the mild panic.

Elasticsearch is awesome—it’s the backbone of a lot of my projects—but it can be a bit temperamental if you don’t set it up to handle failures gracefully. In my case, the cluster crashed, and because I hadn’t configured it to restart automatically, it just stayed down. Not ideal, especially when you’re trying to get stuff done.

After a moment of panic and some frantic troubleshooting, I realized I needed to make sure this didn’t happen again. So, I set out to configure Elasticsearch to start automatically when my VM boots up and to restart itself if it ever crashes. Here’s how I did it, and how you can too—because trust me, you don’t want to learn this lesson the hard way like I did.

### Step 1: Enable Elasticsearch to Start on Boot

First things first, I needed to make sure that Elasticsearch would start up automatically whenever my VM booted up. Since I’m running AlmaLinux, which uses `systemd` to manage services, this was pretty straightforward. I just ran:

```bash
sudo systemctl enable elasticsearch
```

This command tells `systemd` to start Elasticsearch automatically every time the VM starts. Simple, but super important.

### Step 2: Configure Elasticsearch to Restart on Failure

Next, I wanted to make sure that if Elasticsearch crashed for any reason, it would try to restart itself. Nobody wants a service that gives up after one hiccup, right? To do this, I needed to tweak the service configuration.
I opened the service file for editing with:

```bash
sudo systemctl edit elasticsearch.service
```

This command opens a text editor where I could add some custom configuration. Under the `[Service]` section, I added these two lines:

```ini
[Service]
Restart=always
RestartSec=10
```

`Restart=always` means that if the service stops unexpectedly (like in a crash), `systemd` will automatically try to restart it.
`RestartSec=10` sets a 10-second delay before attempting the restart, just to give things a moment to settle down.
Once I saved those changes, I was almost done.

### Step 3: Reload systemd to Apply the Changes

After editing the service file, I needed to tell `systemd` to reload its configuration so the changes would take effect. I did that with:

```bash
sudo systemctl daemon-reload
```

This ensures that `systemd` picks up the new restart policy I just added.

### Step 4: Check the Service Status

To make sure everything was set up correctly, I checked the status of the Elasticsearch service:

```bash
sudo systemctl status elasticsearch
```

This showed me that the service was active and running, and importantly, that the restart policy was in place. So far, so good.

### Step 5: Test the Configuration

Just to be thorough (and to calm my nerves), I decided to test the setup. I simulated a crash by killing the Elasticsearch process manually:

```bash
sudo killall java
```

Then, I quickly checked the status again, and sure enough, `systemd` had restarted the service automatically. Phew! Crisis averted.

### A Few Extra Tips

Even though I now had automatic restarts in place, I learned a couple of other things that are worth sharing:

* **Monitoring is still important:** Automatic restarts are great, but it’s still a good idea to set up some basic monitoring to alert you if the service goes down. In my case, since it was a development environment, I didn’t have anything fancy, but even a simple script or a tool like Prometheus could have saved me some trouble.
* **Check the logs:** When your service crashes, always check the logs to figure out why. For Elasticsearch, the logs are usually in `/var/log/elasticsearch/`. In my case, the logs helped me understand that the crash was due to a temporary resource issue, which I’ve since addressed.
* **Make sure your VM has enough resources:** Elasticsearch can be a bit of a resource hog, especially if you’re running other services on the same VM. Make sure your VM has enough CPU, RAM, and disk space to keep everything running smoothly. Otherwise, you might see more crashes than you’d like.

### Wrapping It Up

So, there you have it. By enabling Elasticsearch to start on boot and configuring it to restart automatically on failure, you can save yourself a lot of headaches down the line. Trust me, learning this the hard way on a Monday morning is not fun. But now, my Elasticsearch cluster is way more resilient, and I can focus on development without worrying about unexpected downtime.
If you haven’t already, take a few minutes to set this up on your own system. It’s quick, easy, and could save you from a future panic moment.
