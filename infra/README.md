# GYANT Challenge App - Infrastructure 

## Requiriments
- aws-cli
- eksctl

```sh
$ eksctl create cluster -f cluster.yaml

$ TRUST="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::627938768582:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"

$ echo '{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": "eks:Describe*", "Resource": "*" } ] }' > /tmp/iam-role-policy

$ aws iam create-role --role-name CodeBuildKubectlRole --assume-role-policy-document "$TRUST" --output text --query 'Role.Arn'

$ aws iam put-role-policy --role-name CodeBuildKubectlRole --policy-name eks-describe --policy-document file:///tmp/iam-role-policy

$ aws iam attach-role-policy --role-name CodeBuildKubectlRole --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess

$ aws iam attach-role-policy --role-name CodeBuildKubectlRole --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildAdminAccess

$ ROLE="    - rolearn: arn:aws:iam::${ACCOUNT_ID}:role/CodeBuildKubectlRole\n      username: build\n      groups:\n        - system:masters"

$ kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml

$ kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"
```

## Access application

Application is running on http://a2241f451f9a24a1ea5d0423a5ff4dea-148851880.eu-west-3.elb.amazonaws.com but to have access is necessary to add your public IP address at "gyant.yaml"

```
  loadBalancerSourceRanges:
  - "213.22.0.0/16"
```

## Horizontal Pod Autoscale

## CloudWatch Monitoring
```sh
$ kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml

$ kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-serviceaccount.yaml

$ curl -O https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-configmap.yaml

$ kubectl apply -f cwagent-configmap.yaml

$ kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml

$ kubectl get pods -n amazon-cloudwatch
```