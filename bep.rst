:BEP: xx
:Title: Extension for Peers to Request and Send Payments
:Version: $Revision$
:Last-Modified: $Date$
:Author:  Andrew Kerry <andrew.kerry@protonmail.com>
:Status:  Proposal
:Type:    Standards Track
:Created: 6-Jul-2021

The purpose of this extension is to allow peers to request and send
payments to other peers.

Handshake
=========

As part of the LTEP handshake [#BEP-10]_, a peer can signal support for this extension
via message "tr_payments":

.. parsed-literal::

    {
      m: {
        tr_payments: *<implementation-dependent local message ID (positive integer)>*,
        ...
      },
      ...
    } 

Request
=======

When a peer requests a piece, the requestee may respond with a message to request payment.

The extension message requesting payment consists of the extension header and the following bencoded payload:

.. parsed-literal::

    {
      bolt11: *Bolt-11 format text in utf-8 describing the amount and method of payment*
    }

The invoice text form must be a [#Bolt #11]_ format invoice. Other formats for the invoice may be supported in future extensions, these would be identifiable by the key in the dictionary. The amount in the invoice must correspond to the price for a piece, i.e. the amount the requestee requires to be transferred before they are willing to send the data for a full piece to the requester. The price per piece, once communicated to a peer, must remain constant for that peer and torrent as long as the peers are connected. The price may vary between torrents and between peers. The price for the last piece must be the same as for other pieces, even if the piece has fewer bytes of data.

Response to payment request
===========================

After a peer has received a request payment, it may respond to the request by transferring funds to the payment requester. After the funds are transferred and received by the payment requester, the piece requester should repeat the request for the piece. The payment requester must then check whether the funds have been received from this peer, and if so, proceed with sending the piece as usual.

The payment requester must keep track of the amount of funds sent by a peer. When data is sent to the peer, the requester should deduct the balance for that peer accordingly. The balance is not shared across torrents. A peer may send funds for more than one piece. If the amount of funds sent by a peer covers the price of sending data, the payment requester must respect the request for data as usual and not request an additional payment.

Tipping
=======

In a case where a peer wishes to send a payment to another peer, however the other peer hasn't requested a payment, a peer may still send an extension message requesting for an invoice such that a payment can be made. The extension message requesting an invoice consists of the extension header and the payload of a bencoded string "tip". Upon receiving the message, the receiver may respond with a message to request payment as described above. The amount in the invoice should be optional such that it can be set by the receiver of the invoice.

Fraud considerations
====================

It is possible for a peer to request payment but not send the piece after the payment. To avoid this, a peer may hold off requesting a payment, or paying an invoice, until some data has been sent. As a result, a peer may receive some data for free, and repeat this for several peers as to receive all data of a torrent without paying even if all peers expect payment, but this will probably negatively affect the download speed. A peer may request payment even for the first piece but in this case it can only upload to peers that trust their peers enough to pay an invoice without having received any data from a peer.

Throughput considerations
=========================

Pausing data transfer due to lack of payment may cause stutter in the throughput. The peer requesting data may choose to send multiple requests to receive multiple invoices, then pay them all and hence batch the payments and the data transfer. This requires the peer that sends funds to trust the other peer to also provide the data. For ideal throughput, clients may internally adjust their perceived level of trust and batch their requests and payments accordingly.

References
==========

.. _`BEP-10`: http://www.bittorrent.org/beps/bep_0010.html
.. _`Bolt #11`: https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md


Copyright
=========

This document has been placed in the public domain.
