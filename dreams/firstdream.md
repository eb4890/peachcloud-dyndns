This is my first thoughts on dynamic dns for peach cloud.

User's experience
=====

**Peach cloud owner**: The owner has been playing around with the peachcloud hardware for a bit getting it working locally. They are ready to try and get it available externally. To do so they get a list of pubs that also provide dyndns services and get an invite to one of them.

Having gotten an invite they need to then tell their peachcloud to talk to that pub for dyndns. They do this by picking a pub and choosing a subdomain in the web frontend. Each pub would have a single domain that people could pick a subdomain from. E.g. If mikey as the pub owner decided to use peach.dinosaur.is as the domain, they could ask for eb4890.peach.dinosaur.is.

Once that is setup, they can see a green dot on their status page showing them than dyndns is working.



**Pub operator**: They need to do a few things.

1. Delegate some dns to bind on their pub
2. Install peach-dns
3. Configure bind and peach-dns to use a specific sub-domain e.g. peach.dinosaur.is
4. Allow access to dns and peach-dns(running http on a known port) through firewalls.
5. Manage the people their pub follows.
6. Publish a message on their pub that they provide dyndns with a url to send updates to


Behind the scenes
=====

The peachcloud raspberry pi would periodically look at the pub they are using for dyndns and get the url to send updates to. It would use a client to send an update. The client would first get the external ip of the system by doing a query to a service (maybe the dyndns service itself). That client would look at .ssb/secrets and sign the message with the private key, and encrypt it with the pubs public key. The message format would be something like.

{msg: {'public_key': @XXXX.ed25519, 'name': 'eb4890', 'ip': '24.12.12.12' } 'signature': "xxxxxx" }

It would then send that message to the url.

The server would look at the public_key, make sure it followed the public key (by looking at the social graph from ssb-server) and that the message was signed properly, then update the bind service using 'nsupdate -l'.

Once that was done it would return a status code to the client.

**Miscellaneous/Optional**:

The peachcloud-dyndns server would use letsencrypt for its https. I'm not sure this is useful (as we have alternate authentication methods). But it could be done.

**Technology choices**: 

- Use rust for both the dyndns client and the server. 
- Serde for json parsing
- Dalek-crypt for elliptic curve en/decryption
- Not sure about the web frameworks to use... does anyone have any ideas?
- Bind as default for dns.


