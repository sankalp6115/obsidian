So, i got an acer tablet from beloved government, but it was locked down very deeply.
Most of the useful functionalities were locked.

To make it a server , it needed to have tailscale,
the way i was aware to do so, was to download an app, tailscale and then connect it to the other devices.
But the tab was locked and didnt allow vpn service required by tailscale app.

At 20/10/2026 00:12

But after few months, i had the idea to try out the tailscale-cli.
I tried and it worked.
Now the tab is also part of my homelab.
It can host services on its own.

Another problem this solved, was that running paper_boy on diff device. then tab required its main IP as it cant access tailscale network, but now that paper boy is running on tab itself, i can just go on 
http://localhost:3002 for paper_boy on the tab and it works delightfully.


