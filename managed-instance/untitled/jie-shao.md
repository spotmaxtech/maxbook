# 介绍

Managed Instances are Spotmax’s solution for launching and managing a single compute instance. On the AWS cloud, for a standard single instance workload, an On-Demand EC2 instance is launched. The instance is expected to be highly available, easily manageable, and integrate well with additional services and monitoring tools.  


Managed Instances provide a way to achieve the same functionality, on Spot instances. This provides the benefit of significant cost savings, while the downside of Spot Instances, namely Spot interruptions, are mitigated through the use of advanced persistence features.

With Managed Instance it is possible to maintain state \(i.e Root and Block Device Volume data, Network Interfaces\) across varying instance types, sizes, pricing models \(Spot, RI, On-Demand\), even across different Availability Zones.

For more information proceed to the following sections.

