# Step 1: Install the Tor Browser

The Tor Browser is available from the Tor Project's website for Windows, macOS, and Linux/Unix systems. It's even usable on Android devices, but we'll be focusing on the computer versions. Just download and install the version that's right for you. If using Linux, such as Kali, you can install Tor more easily from the terminal using apt:

<code>$ sudo apt install torbrowser-launcher</code>

# Step 2: Browse Hidden Services

After Tor installs, you can open "Tor Browser" from your applications, and it will automatically connect to the Tor network. On your first run, it may ask you to "Connect" to Tor or "Configure" it. Choose the former unless you're using a proxy or are in a region that bans Tor.

We can view our route through the Tor network by clicking the site's name in the URL bar where the (i) button is. On older versions, you can click on the drop-down arrow next to the onion icon in the upper left of the window. In the case of visiting an .onion site, we can only see the last relays and the "Onion site" listed in the circuit information.
How to Host Your Own Tor Hidden Service with a Custom Onion Address

# Step 3: Host a Server

The first step in configuring a Tor server will be setting up a way to serve HTTP content, just the same as a regular web server might. While we might choose to run a conventional web server at 0.0.0.0 so that it becomes accessible to the internet as a whole by its IP, we can bind our local server environment to 127.0.0.1 to ensure that it will be available only locally and through Tor.

On a system where we can call a Python module directly, we might choose to use the http.server module. First, we need to be in the directory that contains the content we would like to host. If you just want to test it out with the bare minimum, create a new directory, and change into it.

<code>~$ mkdir tor_service</code>
<code>~$ cd tor_service</code>
<code>~/tor_service$</code>


Now, we can run a server directly from the command line. Using Python 3 and http.server, we can use the following string to bind to 127.0.0.1 and launch a server on port 8080.

<code>
~/tor_service$ python3 -m http.server --bind 127.0.0.1 8080

Serving HTTP on 127.0.0.1 port 8080 (http://127.0.0.1:8080/) ...
</code>

Either of these will serve as enough for a test server, but for any larger or more permanent project a full hosting stack will be more useful. Nginx provides a server backend that can be more suitably hardened and secured against the potential threats against a hidden service, although Apache or other HTTP server software can certainly work. Just be sure that it's bound to 127.0.0.1 to prevent discovery through services such as Shodan.


If you have Fing network scanner, you can confirm this is working by running the following in a new terminal window.

<code>~$ fing -s 127.0.0.1 -o text,console</code>

You should see output like the image below if your server is running. And just like that, a Fing scan shows us our server.

<code>
Serving HTTP on 127.0.0.1 port 8080 (http://127.0.0.1:8080/) ...

04:19:03 > Service scan on: 127.0.0.1

04:19:03 > Service scanning started.
04:19:09 > Detected service:  5432 (postgresql)
04:19:09 > Detected service:  8080 (http-proxy)
04:19:11 > Service scan completed in 7.720 seconds.

-------------------------------------------------------------------------
| Scan result for localhost (127.0.0.1)                                 |
|-----------------------------------------------------------------------|
|  Port | Service         | Description                                 |
|-----------------------------------------------------------------------|
|  5432 | postgresql      | PostgreSQL database server                  |
|  8080 | http-proxy      | Common HTTP proxy/second web server port    |
-------------------------------------------------------------------------
</code>

To ensure that our server is functional, we'll want to test our local address (127.0.0.1) or "localhost" in a web browser by opening it as an address followed by a port number, as seen below.

    <code>http://localhost:8080</code>

To make testing the server easier, it may be useful to create an "index.html" file in the directory from which the server is being run. In a new terminal, in your Tor directory, create the file.

<code>~/tor_service$ touch index.hmtl</code>

Next, use nano or another text editor to add some HTML to the file.

<code>~/tor_service$ nano index.html</code>

Something as simple as this will work:

<code>
<html>
<body>
Null Byte
</body>
</html>
</code>

Exit with Control-X, type Y, then Enter to save the file in nano. The result will look like the following image if it's working properly.
How to Host Your Own Tor Hidden Service with a Custom Onion Address

With our local server environment configured and available at 127.0.0.1:8080, we can now start to link our server to the Tor network.
# Step 4: Create a Hidden Service

First, we will need to install or confirm that the Tor service/daemon is installed. The standalone Tor service is an active separate of the Tor Browser package. For Linux/Unix, it's available here. On Ubuntu or Debian-based distros with apt package management, the following command should work assuming Tor is in the distro's repositories.

<code>
~/tor_service$ sudo apt install tor

[sudo] password for kali:
Reading package lists... Done
Building dependency tree
Reading state information... Done
tor is already the newest version (0.4.3.5-1).
tor set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 568 not upgraded.
</code>

To confirm the location of our Tor installation and configuration, we can use whereis.

<code>
~/tor_service$ whereis tor

tor: /usr/bin/tor /usr/sbin/tor /etc/tor /usr/share/tor /usr/share/man/man1/tor.1.gz
</code>

This will show us a few of the directories which Tor uses for configuration. We're looking for our "torrc" file, which is most likely in /etc/tor. We can move to that directory with cd, as we run the command below.

<code>
~/tor_service$ cd /etc/tor/
~/etc/tor$
</code>

Finally, confirm that "torrc" is present by simply running ls.

<code>
~/etc/tor$ ls

torrc  torsocks.conf
</code>

If the torrc file is present, we will want to edit it. We can use Vim, emacs, or simply GNU nano to edit the file. To edit the file in nano, simply run the following in the terminal. If you're root, you can skip the sudo.

<code>~/etc/tor$ sudo nano torrc</code>

In the file, we're looking for the section highlighted below. To find it quickly, use Control-W to search for "location-hidden," hit Enter, and you should jump right to it.

<code>
############### This section is just for location-hidden services ###

## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.

#HiddenServiceDir /var/lib/tor/hidden_service/
#HiddenServicePort 80 127.0.0.1:80

#HiddenServiceDir /var/lib/tor/other_hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServicePort 22 127.0.0.1:22
</code>

To direct Tor to our hidden service, we'll want to un-comment two lines.

<code>
#HiddenServiceDir /var/lib/tor/hidden_service/
#HiddenServicePort 80 127.0.0.1:80
</code>

To do this, we simply remove the "#" symbols at the beginning of those two lines. While we're here, we need to correct the port on which Tor looks for our server. If we're using port 8080, we'll want to correct the line from port 80 to port 8080 for the "#HiddenServicePort" line.

<code>
############### This section is just for location-hidden services ###

## Once you have configured a hidden service, you can look at the
## contents of the file ".../hidden_service/hostname" for the address
## to tell people.
##
## HiddenServicePort x y:z says to redirect requests on port x to the
## address y:z.

HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:8080

#HiddenServiceDir /var/lib/tor/other_hidden_service/
#HiddenServicePort 80 127.0.0.1:80
#HiddenServicePort 22 127.0.0.1:22
</code>

Write the changes with Control-X, type Y, then Enter to exit.

# Step 5: Test the Tor Service

With the changes written to our torrc file and a server running at 127.0.0.1:8080, making our server accessible over Tor is as simple as starting the Tor service. We can do this from the command line by typing the following.

<code>
~/etc/tor$ sudo tor

Jul 10 19:20:44.992 [notice] Tor 0.4.3.5 running on Linux with Libevent 2.1.11-stable, OpenSSL 1.1.1g, Zlib 1.2.11, Liblzma 5.2.4, and Libzstd 1.4.4.
Jul 10 19:20:44.992 [notice] Tor can't help you if you use it wrong! Learn how to be safe at https://www.torproject.org/download/download#warning
Jul 10 19:20:44.992 [notice] Read configuration file "/etc/tor/torrc".
Jul 10 19:20:44.994 [notice] Opening Socks listener on 127.0.0.1:9050
Jul 10 19:20:44.994 [notice] Opened Socks listener on 127.0.0.1:9050
Jul 10 19:20:44.000 [notice] Parsing GEOIP IPv4 file /usr/share/tor/geoip.
Jul 10 19:20:45.000 [notice] Parsing GEOIP IPv6 file /usr/share/tor/geoip6.
Jul 10 19:20:45.000 [warn] You are running Tor as root. You don't need to, and you probably shouldn't.
Jul 10 19:20:45.000 [notice] Bootstrapped 0% (starting): Starting
Jul 10 19:20:45.000 [notice] Starting with guard context "default"
Jul 10 19:20:46.000 [notice] Bootstrapped 5% (conn): Connecting to a relay
Jul 10 19:20:46.000 [notice] Bootstrapped 10% (conn_done): Connected to a relay
Jul 10 19:20:46.000 [notice] Bootstrapped 14% (handshake): Handshaking with a relay
Jul 10 19:20:46.000 [notice] Bootstrapped 15% (handshake_done): Handshake with a relay done
Jul 10 19:20:46.000 [notice] Bootstrapped 20% (onehop_create): Establishing an encrypted directory connection
Jul 10 19:20:46.000 [notice] Bootstrapped 25% (requesting_status): Asking for networkstatus consensus
Jul 10 19:20:46.000 [notice] Bootstrapped 30% (loading_status): Loading networkstatus consensus
Jul 10 19:20:47.000 [notice] I learned some more directory information, but not enough to build a circuit: We have no usable consensus.
Jul 10 19:20:47.000 [notice] Bootstrapped 40% (loading_keys): Loading authority key certs
Jul 10 19:20:47.000 [notice] The current consensus has no exit nodes. Tor can only build internal paths, such as paths to onion services.
Jul 10 19:20:47.000 [notice] Bootstrapped 45% (requesting_descriptors): Asking for relay descriptors
Jul 10 19:20:47.000 [notice] I learned some more directory information, but not enough to build a circuit: We need more microdescriptors: we have 0/6477, and can only build 0% of likely paths. (We have 0% of guards bw, 0% of midpoint bw, and 0% of end bw (no exits in consensus, using mid) = 0% of path bw.)
Jul 10 19:20:47.000 [notice] Bootstrapped 50% (loading_descriptors): Loading relay descriptors
Jul 10 19:20:48.000 [notice] The current consensus contains exit nodes. Tor can build exit and internal paths.
Jul 10 19:20:49.000 [notice] Bootstrapped 56% (loading_descriptors): Loading relay descriptors
Jul 10 19:20:50.000 [notice] Bootstrapped 62% (loading_descriptors): Loading relay descriptors
Jul 10 19:20:50.000 [notice] Bootstrapped 67% (loading_descriptors): Loading relay descriptors
Jul 10 19:20:50.000 [notice] Bootstrapped 75% (enough_dirinfo): Loaded enough directory info to build circuits
Jul 10 19:20:50.000 [notice] Bootstrapped 80% (ap_conn): Connecting to a relay to build circuits
Jul 10 19:20:50.000 [notice] Bootstrapped 85% (ap_conn_done): Connected to a relay to build circuits
Jul 10 19:20:50.000 [notice] Bootstrapped 89% (ap_handshake): Finishing handshake with a relay to build circuits
Jul 10 19:20:50.000 [notice] Bootstrapped 90% (ap_handshake_done): Handshake finished with a relay to build circuits
Jul 10 19:20:50.000 [notice] Bootstrapped 95% (circuit_create): Establishing a Tor circuit
Jul 10 19:20:50.000 [notice] Bootstrapped 100% (done): Done
</code>

Upon starting Tor for the first time with our new configuration, an .onion address will be generated automatically. This information will be stored in "/var/lib/tor/hidden_service" (or another directory if specified in the torrc file). First, in a new directory, if you're not a root user, get root permissions.

<code>
~$ sudo cd /var/lib/tor/hidden_service
[sudo] password for kali:
root@kali:/home/kali# kali
</code>

Then, we can move to the directory in a new terminal window with cd.

<code>
root@kali:/home/kali# cd /var/lib/tor/hidden_service
root@kali:/var/lib/tor/hidden_service#
</code>

Next, run ls to ensure that both the "hostname" and "private_key" files are in the directory

<code>
root@kali:/var/lib/tor/hidden_service# ls

hostname  private_key
</code>

Then, we can view our newly generated address by running cat.

<code>
root@kali:/var/lib/tor/hidden_service# cat hostname

4zkl7lz6w67gp46k4qmqahz4ydjewzihlymwy4vrfn2mnqd.onion
</code>

The string ending in .onion is our new hidden service address! While this one was automatically generated, we'll be able to customize it later to our preference.

We can test that our service is accessible by opening it in Tor Browser. If the address resolves to your server, you've successfully hosted a hidden service!
