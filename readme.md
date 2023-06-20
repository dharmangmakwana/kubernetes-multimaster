
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=dm-kubernetes  --approve

eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster dm-kubernetes \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
 --region=us-east-1 \
  --role-name AmazonEKS_EBS_CSI_DriverRole


eksctl create addon  --region=us-east-1  --name aws-ebs-csi-driver --cluster dm-kubernetes --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole --force

192.168.162.94:6379 192.168.247.151:6379 192.168.196.71:6379 192.168.130.209:6379 192.168.237.108:6379 192.168.134.223:6379 



#!/bin/bash
# Find ClusterIPs of Redis nodes
export REDIS_NODES=$(kubectl get pods  -l app=redis-cluster -n redis -o json | jq -r '.items | map(.status.podIP) | join(":6379 ")'):6379
# Activate the Redis cluster
kubectl exec -it redis-cluster-0 -n redis -- redis-cli --cluster create --cluster-replicas 1 ${REDIS_NODES}
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1
# Check if all went well
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -n redis -- redis-cli role; echo; done
view raw