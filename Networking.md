What is networking

## Fully Qualified Domain Name (FQDN)

A Fully Qualified Domain Name (FQDN) consists of three parts - the hostname, domain, and top-level domain (TLD). The format looks like so,

[host name].[domain].[tld]

For the FQDN `www.udacity.com` - `www` is the hostname, `udacity` is the domain, and `com` is the top-level domain.

### types of DNS

| Type of Record  | Maps From     | Maps To    |
| --------------- | ------------- | ---------- |
| A Record        | FQDN          | Ipv4       |
| AAAA Record     | FQDN          | IPv6       |
| PTR Record      | IP address    | FQDN       |
| CNAME Record    | FQDN          | FQDN       |
| NS name service | IP address    | FQDN       |
| NS name service | FQDN          | IP address |
| MX              | Email address | IP address |



# addressing

-  (`0`, `10`, and `127`) are blocks reserved
- `192` block is reserved, but some of it is.
- `224` is set aside for [IP multicast](https://en.wikipedia.org/wiki/IP_multicast).
- `240` was invalid

Switching

Routing

Domain Name System

Load balancing

### Load Balancing Approaches - Summary

Different load balancing approaches provide different benefits. These different approaches are explained below. This is, however, not an exhaustive list, just a list of some of the most common approaches.

- Round Robin - Requests are distributed across the group of servers sequentially.
- BGP Anycast - BGP Anycast allows multiple servers to advertise the same IP address. DNS servers respond to queries with the same IP address. The routing infrastructure of the internet, using BGP, then routes internet traffic to different web servers over the shortest route possible.
- Policy-based DNS load balancing - Uses policies to load balance traffic requests. For example, the IP address of the client’s resolver may be used to determine which server receives the request.
- Dedicated Load Balancing - Enables you to deploy and configure one or more custom load-balancers within a VPC.

# IAC

Infrastructure as code

### ELSA model

[Here](https://medium.dromologue.com/elsa-7188e25beb3) is an article by Justin Arbuckle, VP at CHEF Software, going deeper on [ELSA model](https://medium.dromologue.com/elsa-7188e25beb3), with some examples.