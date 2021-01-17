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