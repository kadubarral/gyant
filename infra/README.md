eksctl create cluster -f cluster.yaml

aws ecr create-repository --repository-name gyant

export GITHUB_TOKEN=[your-github-token]
export GITHUB_USER=[your-github-username]

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=gyant-infra \
  --branch=main \
  --path=gyant-cluster \
  --personal

git clone https://github.com/$GITHUB_USER/gyant-infra cd gyant-infra

flux create source git gyant \
  --url=https://github.com/kadubarral/gyant] \
  --branch=main \
  --interval=30s \
  --export > ./gyant-cluster/gyant-source.yaml

flux create kustomization gyant \
 --source=gyant \
 --path="./infra/k8s" \
 --prune=true \
 --validation=client \
 --interval=1h \
 --export > ./gyant-cluster/gyant-sync.yaml

git add -A && git commit -m "add gitops deploy" && git push