# Terraform Exercise

## Outline

Here we have some terraform to build a simple VPC network, for now we have
just one instance running the web server Nginx in its default
configuration, serving up the default welcome page. This would be run with
the following command:

    terraform init && terraform apply -var-file=dublin.tfvars

We want this to be extended. You're tasked with making the alterations
detailed below. After completing each stage, a test to show the things are
still working would be to run the following command. Expect to see the
Nginx welcome page HTML.

    terraform output nginx_domain | xargs curl

## Exercises

1. We want to be able to run the same stack closer to our customers in the
US. Please build the same stack in the `us-east-1` (Virginia) region,
leaving the existing one in place too.  Feel free to modify the code and or
structure as much as needed in order to do this. You'll need to consider
terraform state: each stack should have its own state, but don't feel you
need to go as far as setting up remote state. As for a CIDR for the new
VPC, use whatever you feel like, providing that it is compliant with
RFC-1918 and does not overlap with the Dublin network.

2. Virginia has several availability zones: we want to use 4. However we
still want to run the stack in Ireland using the 3 AZs there. Modify the
Virgina stack to span 4 AZs. Do this however you like, but consider that we
would like to see re-use of code.

3. The EC2 instance running Nginx went down over the weekend. It's been
decided that we need a solution that is more resilient than just a single
instance. Please implement a solution that you'd be confident would
continue to run in the event one instance goes down.

4. We are looking to improve the security and segregation of our network.
We've decided we would like private subnets that are not addressable on the
internet. Modify the VPC to meet this requirement. The private subnets
should still have egress internet connectivity.

5. Describe any general changes or improvements that you would make to this
   Terraform.

## Expected Outcomes

<details>
<summary>Expected Outcomes</summary>

1. Add a Virginia TFVars file, with the region changed to `us-east-1`. Create a Terraform workspace for the Virginia deployment, in order to have a separate TF file with little change. Or, go the module route, and have separate folders with module implementations with different variable definitions. Set VPC CIDRs to e.g. `10.10.11.0/24`, the key differential being a change in the second or third octet in order to avoid overlap with the Dublin region.

2. With the workspaces solution, could use a ternary to set a count for subnets in different AZs, according to whether the Virginia region is set. Would be good to see comment on how this is not a scalable approach, and if other regions were to be added, a map would be a better data structure to configure per-region. Would need modifying of the TF to make subnets and route table associations also work by count. Could be done dynamically with the `aws_availability_zones` `data` source, in order to index AZs by the count. Similarly for the module approach, the count can be added as a tfvar, which could equally apply to a workspace approach.

3. Extend the count to also apply to EC2 instances, and set the subnet ID dynamically by indexing with the count. Add an ALB to handle round-robin traffic distribution across instances in different subnets / AZs, with healthchecking to ensure that dead instances are not routed to.

4. Set `associate_public_ip_address` to `false` for the instance(s). Add or update the security group to prevent public access: restrict to e.g. the VPC CIDR. Add a public subnet with a public NAT gateway, and add appropriate route table entries for private subnet(s) to reach the NAT gateway, and for the NAT gateway to reach the internet gateway.

5. Modularisation, ALB, ECS, custom domain support, refined security groups, disabling default security groups, HTTPS-only, removing SSH access, session manager access.

</details>