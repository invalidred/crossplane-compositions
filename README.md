# Crossplane Compositions

Use crossplane compositions to create a custom CRD that hides platform complexities from Dev teams.


## Setup

- Install k3d as [seen here](https://github.com/invalidred/argo-workflow#setup)

- Install Helm
  ```bash
  brew install helm
  ```

- Install opensearch

  ```bash
  helm repo add opensearch https://opensearch-project.github.io/helm-charts/
  helm repo update
  helm install opensearch opensearch/opensearch --values values.yml
  helm install opensearch-dashboards opensearch/opensearch-dashboards --values values.yml
  ```

## General Open Search

- Port forward to open search

  ```bash
  k port-forward svc/opensearch-dashboards 5601:5601
  ```

- Open Dashboard: `http://localhost:5601`

  ```bash
  username: admin
  password: check value value.yml
  ```
- Click on Dev tools
  <img width="380" alt="Screenshot 2024-07-31 at 11 50 34 AM" src="https://github.com/user-attachments/assets/d5bd0365-17cd-4a51-81c5-351e2820f367">



## Lab

Once you're in the dev console follow the exercise [found here](https://github.com/madhusudhankonda/elasticsearch-in-action/blob/9f292ffcb96e6a5736ae80103dc4565f204502e5/kibana_scripts/ch02_getting_started.txt)

