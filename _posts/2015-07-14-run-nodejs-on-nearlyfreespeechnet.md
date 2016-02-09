---
layout: post
title: "Run node.js on NearlyFreeSpeech.Net"
description: ""
category:
tags: [nodejs, nearlyfreespeech]
---

NearlyFreeSpeech.net (NFSN) is a very inexpensive web host, DNS provider and domain registrar. NFSN [added support](https://blog.nearlyfreespeech.net/2014/09/24/more-power-more-control-more-insight-less-cost/) in 2014 for NodeJS, Django, and tons of other languages through persistent processes. This guide, [based on the Django tutorial provided by NFSN](https://blog.nearlyfreespeech.net/2014/11/17/how-to-django-on-nearlyfreespeech-net/), will demonstrate the setup I used for creating a node.js daemon.

---

NFSN Configuration
------------------

If you're creating a new website, select the *Custom* domain type:

![Choose custom domain option for new site](/assets/images/nfsn-node/custom-domain.png)

If you have an exisiting website, the domain can by changed to *Custom*:

1. Select your domain under **Sites**
    ![Site selection](/assets/images/nfsn-node/custom-change1.png)
2. Go to the **Config Information** section and on click the **Edit** button on *Server Type*
    ![Config information](/assets/images/nfsn-node/custom-change2.png)
3. Select the *Custom* domain type:
    ![Choose custom domain](/assets/images/nfsn-node/custom-change3.png)

---

Project Setup
-------------

Next, use SSH to setup your node service on NFSN'S server. Your username, password, and SSH hostname can be found in your site's information:

![SSH information](/assets/images/nfsn-node/site-information.png)

<ol>
<li>Connect to your server with SSH. See <a href="#ssh-key-setup">this section to set up a SSH key on NFSN</a>.

<pre>
> <strong>ssh mopsled_mycustomdomain@ssh.phx.nearlyfreespeech.net</strong>
mopsled_mycustomdomain@ssh.phx.nearlyfreespeech.net's password:
[mycustomdomain /home/public]$</pre></li>

<li>Go into the <code>/home/protected</code> folder to set up your node website. This will allow <code>node.js</code> to run as the <code>web</code> user.

<pre>[mycustomdomain /home/public]$ <strong>cd /home/protected</strong></pre></li>

<li>Upload or clone your node.js code into this folder. In this example, I'm going to use the <a href="https://github.com/heroku/node-js-sample">node-js-sample</a> repository.

<pre>[mycustomdomain /home/protected]$ <strong>git clone https://github.com/heroku/node-js-sample.git</strong>
[mycustomdomain /home/protected]$ <strong>cd node-js-sample</strong></pre></li>

<li>Install <code>npm</code> dependencies for your project

<pre>[mycustomdomain /home/protected/node-js-sample]$ <strong>npm install</strong></pre></li>

<li>Create <code>run.sh</code> script for running your node process. A NFSN daemon is given a file to run. This file will encapsulate the <code>npm run start</code> command and any other setup your process needs. See <a href="#setting-environment-variables">environment variable setup</a> in the <em>Additional Details</em> if your project requires environment variables.

<pre>[mycustomdomain /home/protected/node-js-sample]$ <strong>nano run.sh</strong></pre>

In <code>run.sh</code>, type the command that starts your node.js process:

<pre>npm run start</pre>

<strong><code>&lt;CTRL&gt;+X</code></strong>, <strong><code>Y</code></strong>, <strong><code>&lt;Enter&gt;</code></strong> to save and exit from <code>nano</code>.<br><br></li>

<li>Test running your startup script:

<pre><code>[mycustomdomain /home/protected/node-js-sample]$ <strong>chmod +x run.sh</strong>
[mycustomdomain /home/protected/node-js-sample]$ <strong>./run.sh</strong>

> node-js-sample@0.2.0 start /home/protected/node-js-sample
> node index.js

Node app is running at localhost:5000</code></pre></li>
</ol>

Make sure your node.js service works from `run.sh` before continuing to the next step. Press `CTRL`+`X` to kill your service on SSH before the next section.

Note: It's okay if your node app runs at a high-level port like `5000`, `8080`, or `10080`. NFSN doesn't give root permissions to allow for low-level port access, but instead uses *Proxies* to give your server access to low level ports. See [Configure Proxy](#configure-proxy) later in the guide.

Configure NearlyFreeSpeech.net Daemon
-------------------------------------

1. Select your domain under **Sites**
    ![Site selection](/assets/images/nfsn-node/daemon1.png)
2. Go to the **Daemons** section and on click the **Add a Daemon** button
    ![Add a daemon button](/assets/images/nfsn-node/daemon2.png)
3. Change configuration:
    - `Tag: node` *Note: This tag is used to locate logs for your process*
    - `Command Line: `**`/home/protected/<NODE_FOLDER>/run.sh`**
    - `Working Directory: `**`/home/protected/<NODE_FOLDER>`**
    - `Run Daemon As: `**`web`**

    ![Daemon configuration](/assets/images/nfsn-node/daemon3.png)

4. Click the **Add Daemon** button

---

<h2 id="configure-proxy">Configure Proxy</h2>

1. Go back to your site configuration

2. Go to the **Proxies** section and on click the **Add a Proxy** button
    ![Add a proxy button](/assets/images/nfsn-node/proxy1.png)

3. Configure proxy:

    - `Protocol: `**`http`**
    - `Base URI: `**`/`**
    - `Document Root: `**`/`**
    - `Target Port: `**`5000`** *Note: Change to your node port if different*
    - `Direct: `**`☑ Bypass Apache Entirely`** *(Checked)*

    ![](/assets/images/nfsn-node/proxy2.png)

4. Click the **Add Proxy** button

Check your work! Go to your domain to see if your daemon and proxy are working.

![](/assets/images/nfsn-node/hello-world.png)

---

Troubleshooting
---------------

If you website isn't working yet, you can use SSH to consult your process's logs. Output from the daemon process is stored in `/home/logs/daemon_node.log`.

<pre>mycustomdomain /home/protected/node-js-sample]$ <strong>cd /home/logs</strong>
[mycustomdomain /home/logs]$ <strong>ls</strong>
daemon_node.log
[mycustomdomain /home/logs]$ <strong>tail -f daemon_node.log</strong>

> node-js-sample@0.2.0 start /home/protected/node-js-sample
> node index.js

Node app is running at localhost:5000</pre>

To restart your daemon process, go to your site's configuration and click **Stop** on your daemon:

![](/assets/images/nfsn-node/daemon-stop.png)

Afterward, click **Start** on your daemon to start it again:

![](/assets/images/nfsn-node/daemon-start.png)

---

Additional Details
------------------

<h3 id="ssh-key-setup">Set up an SSH key on NearlyFreeSpeech.net</h3>

Setting up an SSH key isn't difficult, but uses the NFSN web interface instead of the filesystem. See [NearlyFreeSpeech.net's reasoning here](https://faq.nearlyfreespeech.net/section/uploading/sshkeys#sshkeys).

1. Go the **Profile** tab

2. Click **Add SSH Key** in the sidebar

    ![](/assets/images/nfsn-node/ssh1.png)

3. Paste your key into the input box, and click **Add SSH Key**

<h3 id="setting-environment-variables">Setting environment variables on NearlyFreeSpeech.net Daemon</h3>

NFSN doesn't have a environment variable editor like heroku or other cloud services, but environment variables can easily be set in the `run.sh` script file.

For example, if my node process required the `PORT`, `AUTH_KEY` and `NZ` (timezone) environment variables set, my `run.sh` script would look like this:

    PORT="8080" AUTH_KEY="XXX_XXXXXXXXXX" NZ="America/Denver" npm run start