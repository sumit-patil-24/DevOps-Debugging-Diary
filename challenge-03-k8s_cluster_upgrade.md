
# Upgrade production grade k8s cluster with zero downtime 

- this is a common task as a devops engineer
- once in every 3 month k8s comes with a new version
- k8s only supports latest 3 versions. (e.g., if 1.32 is put , then 1.32, 1.31, 1.30 will be supported).
- we need to understand the prerequisite to start the cluster upgrade and the process of upgrading k8s cluster.

# Prerequisites:
1. cordon nodes/make nodes unschedulable during cluster upgrades:
2. make sure you go through the release nodes:
3. k8s upgrades are irriversible
4. Test the upgrade in lower environment first
5. your control plane and data plane should have same k8s version
6. if you are using cluster autoscaler, you need to make sure that it matches with the version of control plane
7. At least 5 avaliable ip addresses within the subnet you provided during k8s cluster creation.
8. kubelet should match the version of control plane

    This are the major prerequisites sometimes according to your organization, k8s cluster, avaliability zones, regions prerequisites might change.

# Process of upgrading the k8s cluster 
1. upgrading control plane:- (takes 30 mins)
2. Upgrade nodegroup/nodes:
3. Upgrade
4. Addons
