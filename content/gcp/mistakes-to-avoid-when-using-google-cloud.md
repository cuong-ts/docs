# Mistakes to avoid when using Google Cloud



Google Cloud Platform is a suite of public cloud computing services offered by Google. The platform includes a range of hosted services for compute, storage and application development that run on Google hardware. Google Cloud Platform services can be used by software developers, cloud administrators and other enterprise IT professionals.

### Avoid excessive use of default service accounts and primitive roles

**IAM** stands for **identity and Access management**[\[1\]](https://cloud.google.com/iam/docs/overview), and it allows you to manage access control by defining who \(identity\) has what access \(role\) to which resources. Cloud IAM has many predefined roles. Most of these roles provide appropriate access to suit your needs and the most common use cases. Well, not always.

When you enable or use some Google Cloud services, they create user-managed service accounts that allow them to deploy jobs, that access other Google Cloud resources. These accounts are known as default service accounts. Convenient, right? Unfortunately, it is not as bright as you'd imagine, the default service account practically always use primitive roles that has too extensive rights, i.e. Editor.  
![](https://storage.googleapis.com/gweb-cloudblog-publish/images/zGbQytQs1Ez.max-700x700.png)

Permissions matrix

Source: https://storage.googleapis.com/gweb-cloudblog-publish/images/zGbQytQs1Ez.max-700x700.png

"Primitive roles" like Owner and Editor grant wide-ranging access to all project resources. Your newly launched service now has access to all resources in your project. This is totally safe, right? \(No, no it's not\).![Is it?](https://sysdogs.com/static/6b91b4ad2f058a2e43d2d3e69beb5f49/4b401/thisisfine.jpg)

Is it?

**Always use the minimum required, limit permissions to only those you need:**

* As a first step, we strongly suggest disabling the automatic addition of permissions for the default service accounts[\[2\]](https://cloud.google.com/resource-manager/docs/organization-policy/restricting-service-accounts#disable_service_account_default_grants)
* Use a custom service account. Create a separate service account for each logical element in your infrastructure, such as an instance group or application. This will reduce the risk of abuse of rights, and will give you greater control over the rights of individual component.
* Granulation will allow precise adjustment of entitlements to a specific service - the minimum required.
* Use custom roles. Create a role that contains only the required permissions. Your application usually has to read something from the bucket, it doesn't need to manage all the buckets in the project. Permissions allow users to perform specific actions on Google Cloud resources[\[3\]](https://cloud.google.com/iam/docs/custom-roles-permissions-support), and usually look like this:

```text
service.resource.verb
---

compute.instances.list
compute instances.stop
storage.objects.get
storage.objects.list
```

A basic example of a custom role could look like this:![Is it?](https://sysdogs.com/static/6b91b4ad2f058a2e43d2d3e69beb5f49/4b401/thisisfine.jpg)

Is it?

Additional tip - Searching for the appropriate permissions can often be inconvenient, when working with legacy code. The recommendations engine may be helpful. Using separate accounts, Recommender will check which roles have been used for the last 90 days and propose an appropriate set of permissions. [\[4\]](https://cloud.google.com/iam/docs/recommender-overview)

### Flawed VPC design

**VPC** - Virtual Private Cloud. You can think of a VPC network the same way you'd think of a physical network, except that it is virtualized within Google Cloud. Within a VPC you create and manage typical network resources such as subnets, firewall rules, routes and more.  


As with the previous point, when creating a project, GCP generates a default setup, with basic settings and a few firewall rules. Unfortunately, the pre-populated rules in the default network allow for a wide range of insecurities:

> * **default-allow-internal** - Allows ingress connections for all protocols and ports among instances in the network. This rule has the second-to-lowest priority of 65534, and it effectively permits incoming connections to VM instances from others in the same network. This rule allows traffic in 10.128.0.0/9 \(from 10.128.0.1 to 10.255.255.254\), a range that covers all subnets in the network.
> * **default-allow-ssh** - Allows ingress connections on TCP destination port 22 from any source to any instance in the network. This rule has a priority of 65534.
> * **default-allow-rdp** - Allows ingress connections on TCP destination port 3389 from any source to any instance in the network. This rule has a priority of 65534, and it enables connections to instances running the Microsoft Remote Desktop Protocol \(RDP\).
> * **default-allow-icmp** - Allows ingress ICMP traffic from any source to any instance in the network. This rule has a priority of 65534, and it enables tools such as ping.

These defaults are predictable, permissive, and inflict unnecessary risk towards your environment. For your safety, we suggest deleting these rules and making your own, depending on your needs. Follow the minimum required rule.  


**Subnets** are also created automatically, in each region, with predefined IP ranges. It is neither necessary nor convenient. You don't have to use every region in the world, you don't want to have the same IP ranges in every VPC. Why? If you plan to connect multiple VPC using for example VPC Network Peering or VPN, you may encounter a problem with overlapping IP addresses.  


Consider VPC network design early. In many cases, the project grows very quickly and you can forget about such things... Unfortunately, this approach in network configuration causes more problems than you can imagine. Sometimes needed changes will require resource recreation in a different region or an entirely different implementation. It's worth rethinking your network architecture beforehand, there are some good rules for it:

* Naming convention - make your network resource names understandable and maintainable.
* Group services into separate subnets.
* Think about allocation/reservation of IP scopes for multiple VPCs - to prevent overlapping.
* Consider advantages and disadvantages of different connection or routing propagation methods[\[5\]](https://cloud.google.com/solutions/best-practices-vpc-design#choose-method), in some cases this can have a huge impact on the network architecture.

![](https://cloud.google.com/architecture/images/vpc-bps-single-project-single-vpc.svg)

Single project, single VPC network

Source: https://cloud.google.com/architecture/images/vpc-bps-single-project-single-vpc.svg

### Neglecting network features

By default, all VMs in GCP are assigned a public IP address and are therefore accessible directly from the internet if there are firewall rules that allows it \(such as the default ones\). External IP address also allows external communication - access to internet, depending on the environment it is required or not. A safe and convenient approach is to use a NAT gateway, in this case **Cloud NAT** service. There are several benefits associated with it, especially in production environments:

* Security - You easily mitigate the problem of exposing services publicly.
* Availability - Cloud NAT is a distributed, software-defined managed service. It doesn't depend on any VMs in your project or a single physical gateway device.
* Scalabillity - You can scale the number of NAT IP addresses that it uses.
* Fixed external IP addresses - An example of why you might need fixed outbound IP addresses is the case in which a third party allows requests from specific external IP addresses.

OK, we have abandoned external IP addresses in favor of the NAT service, but how do we get to communicate with the instance that has no external IP address? In this configuration, we can only use another instance on the same network or VPN connection[\[6\]](https://cloud.google.com/solutions/connecting-securely#vpn)

**Bastion host**

Another secure setup proposal is commonly called `bastion host` - an external endpoint that allows your clients to SSH from the public internet. With this set-up, your application can be safely kept away from public eyes. You can use the bastion in several ways, the most common one is called "Jump server". By using `ssh` forwarding you can get to the target machine. [\[7\]](https://cloud.google.com/compute/docs/instances/connecting-advanced#bastion_host). Remember that you should also secure the bastion's `ssh`, to avoid trouble - our article about Shell security should prove itself useful here.[\[8\]](https://sysdogs.com/articles/how-to-secure-ssh)

![](https://cloud.google.com/solutions/images/bastion.png)

Bastion

Source: https://cloud.google.com/solutions/images/bastion.png

The second approach that is often used, is to set up a VPN server such as OpenVPN/Pritunl on the bastion. This allows the configuration and adaptation of access through these services, for example:

* External auth integration
* 2FA - Two-factor authentication
* Roles/polices to access specific subnets

### Conclusion - what to do?

* Apply the principle of least privilege to service accounts.
* Remove the insecure defaults of VPCs.
* Consider VPC network design early.
* You can provision your VMs without public IPs, reducing the attack surface.
* Use NAT GW for outbound traffic.
* Securely connect to VM instances using bastion.

REF: [https://sysdogs.com/articles/common-mistakes-to-avoid-gcp](https://sysdogs.com/articles/common-mistakes-to-avoid-gcp)

