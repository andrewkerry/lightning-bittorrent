Usage
=====

If you're interested in testing a prototype for sending and receiving payments with your Bittorrent peers then this page is for you.

WARNING: all the related code is highly EXPERIMENTAL and there is a risk of LOSING YOUR FUNDS. Use a testnet or a very little amount of money to try it out. The Bittorrent client will have full access to your funds and a bug or a malicious peer could cause you to lose your funds.

Prerequisites
-------------

At the moment there is no support available in an upstream release so you need to compile the code yourself.

You will also need to be running your own Lightning node. Currently c-lightning and lnd are supported.

The clients for which Lightning support exists are Transmission and qBittorrent. The combination of Transmission+lnd currently isn't supported.

The code has only been tested on Mac OS but Linux should work as well. Windows most probably won't work out of the box (sorry). I'm not 100% sure the configuration code for Linux is implemented correctly in Transmission. qBittorrent configuration should work well for both Mac OS and Linux.

So you will need to either:

* Compile Transmission
* Have c-lightning running locally

Or:

* Compile libtorrent (used by qBittorrent)
* Compile qBittorrent
* Have either lnd or c-lightning running locally

(lnd supports remote connections so technically it doesn't need to be running locally, however remote connections to lnd haven't been tested, and also the SSL certificate checking isn't implemented so running locally is probably the safest option.)

Transmission + c-lightning
--------------------------

For c-lightning, no modifications are necessary. Simply run it as per their instructions (https://github.com/ElementsProject/lightning).

For Transmission, follow these steps:

1. Get the code at: https://github.com/andrewkerry/transmission/tree/lightning
2. Compile it as per the instructions (using cmake).
3. Configure it to use c-lightning, see below for details.
4. Use Transmission as you normally would. When issues arise see the troubleshooting section below.

Configuration
~~~~~~~~~~~~~

Transmission can be configured for Lightning using the default configuration file. On Mac OS, this is at ~/Library/Preferences/org.m0k.transmission.plist. On Linux it's probably $HOME/.config/transmission-daemon/settings.json (according to the documentation at https://github.com/transmission/transmission/wiki/Configuration-Files).

For editing the Mac OS plist file, either you need tools to convert it to XML and back, or open it using Xcode. For Linux (assuming it works) you would edit the settings.json file. The key you need to insert is "LightningRPCName" (on Mac OS) or "lightning_rpc_name" (on Linux) (without quotation marks). The value is the path to the RPC file that lightningd is listening on, e.g. "/tmp/l3-testnet/testnet/lightning-rpc" (without quotation marks).

In the same file you can set the keys "LightningRequestingPricePerPieceMsat" and "LightningWillingToPayPricePerPieceMsat" (Mac OS) or "willing_to_pay_price_per_piece" and "requesting_price_per_piece" (Linux), unit is msatoshi per piece. To improve throughput Transmission currently will pay a peer for up to 16 pieces upfront. It's currently not possible to change the requested price on a torrent-by-torrent basis.

qBittorrent + c-lightning or lnd
--------------------------------

As above, no modifications are needed for the Lightning node but it needs to be running. For qBittorrent:

1. Get the libtorrent code at: https://github.com/andrewkerry/libtorrent/tree/lightning
2. Compile it. The Boost build system must be used because of changes made in the Jamfile. An example command (after b2 setup): b2 logging=on variant=debug crypto=openssl install --prefix=out (crypto=openssl isn't required for c-lightning, however it is required if you're running lnd.)
3. Get the qBittorrent code at: https://github.com/andrewkerry/qBittorrent/tree/lightning
4. Compile it (using make).
5. Run it, but point the LD_LIBRARY_PATH to your libtorrent build. E.g. LD_LIBRARY_PATH=/home/me/libtorrent/out/lib ./qBittorrent (on Mac OS: DYLD_LIBRARY_PATH=/Users/me/libtorrent/out/lib src/qbittorrent.app/Contents/MacOS/qbittorrent)
6. Configure it to use either c-lightning or lnd, see details below.
7. Use qBittorrent as you normally would. When issues arise see the troubleshooting section below.

Configuration
~~~~~~~~~~~~~

The configuration is done by editing a .ini file (Mac OS) or a .conf file (Linux), specifically ~/.config/qBittorrent/qBittorrent.ini (or .conf).

The first field to add is "Lightning\Node" under the "[Preferences]" section. Valid values are either "lnd" or "clightning". There are examples below.

If you're using lightning-c you only need to set the RPC file path under the "Lightning\Param1" key.

For lnd you need four parameters, namely: path to the admin.macaroon file, path to the tls.cert file (currently ignored), hostname and port to connect to.

Independently of the node you can configure the payment limits (maximum price to pay per torrent and requested total price per torrent). Currently these can only be set globally (as opposed to on a torrent-by-torrent basis).

E.g. for lightning-c a valid configuration would look like:

.. code-block::

  [Preferences]
  ...
  Lightning\Node=clightning
  Lightning\Param1=/tmp/l2-testnet/testnet/lightning-rpc
  Lightning\RequestingPricePerTorrentMsat=100000
  Lightning\WillingToPayPricePerTorrentMsat=100000

For lnd an example configuration is:

.. code-block::

  [Preferences]
  ...
  Lightning\Node=lnd
  Lightning\Param1=/Users/user/Library/Application Support/Lnd/data/chain/bitcoin/testnet/admin.macaroon
  Lightning\Param2=/Users/user/Library/Application Support/Lnd/tls.cert
  Lightning\Param3=127.0.0.1
  Lightning\Param4=8080
  Lightning\RequestingPricePerTorrentMsat=100000
  Lightning\WillingToPayPricePerTorrentMsat=100000

Troubleshooting
---------------

Currently there's no UI to show whether a peer is accepting and receiving payments or not. The best way to find out what's happening is enabling debug logging in the client, and reviewing the logs.

If you know your peer is asking for a payment and you are willing to pay, and have configured your client accordingly but aren't able to download from the peer, or alternatively know your peer is willing to pay and you're asking for a payment, yet unable to upload, check these items:

* Did the extended handshake take place, with both peers signaling support for payments?
* Are both peers able to connect to their Lightning nodes?
* Are both Lightning nodes on the same network (testnet vs. mainnet), and connected via channels?
* Is the uploading peer able to create and send an invoice? You can also check this by reviewing the node status.
* Is the downloading peer able to receive and pay an invoice? You can also check this by reviewing the node status.
* Is the amount requested within the bounds of the channels and the configured maximum amount the peer is willing to pay?
* Is the amount above minimum transfer amount? Lnd seems to have a default minimum of 1000 millisatoshi per transfer.
