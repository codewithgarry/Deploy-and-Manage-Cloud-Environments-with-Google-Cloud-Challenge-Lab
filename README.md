# Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab (GSP314)

![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)
![Lab](https://img.shields.io/badge/Lab-GSP314-34A853)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![Spinnaker](https://img.shields.io/badge/Spinnaker-139BB4?logo=spinnaker&logoColor=white)

Reference solution for the Google Cloud challenge lab **Deploy and Manage Cloud Environments with Google Cloud** (GSP314).

## 📖 Overview

For the fictional "Kraken" team you:

1. Build a production network with Deployment Manager and run the app on GKE.
2. Create a dual-homed admin host with a CPU alert.
3. Verify a Spinnaker continuous delivery pipeline by pushing a tagged release.

## 🧰 Services used

Deployment Manager · Google Kubernetes Engine · Compute Engine · Cloud Monitoring · Cloud Source Repositories · Spinnaker

## ✅ Prerequisites

```bash
gcloud config set compute/zone us-east1-b
```

## 🚀 Solution

### Task 1: Create the production environment

```bash
cd /work/dm
sed -i 's/SET_REGION/us-east1/g' prod-network.yaml
gcloud deployment-manager deployments create prod-network --config=prod-network.yaml

gcloud container clusters create kraken-prod \
  --num-nodes 2 \
  --network kraken-prod-vpc \
  --subnetwork kraken-prod-subnet \
  --zone us-east1-b
gcloud container clusters get-credentials kraken-prod --zone us-east1-b

cd /work/k8s
for f in *.yaml; do kubectl create -f "$f"; done
```

### Task 2: Configure the admin host

Create a VM with one interface in the management subnet and one in production:

```bash
gcloud compute instances create kraken-admin \
  --network-interface="subnet=kraken-mgmt-subnet" \
  --network-interface="subnet=kraken-prod-subnet"
```

Then, in **Cloud Monitoring → Alerting**, create a policy that emails you when `kraken-admin` CPU utilization is above **50% for 1 minute**.

### Task 3: Verify the Spinnaker deployment

Expose the Spinnaker UI (Deck) through Cloud Shell Web Preview:

```bash
gcloud container clusters get-credentials spinnaker-tutorial --zone us-east1-b

DECK_POD=$(kubectl get pods -n default -l "cluster=spin-deck" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward -n default $DECK_POD 8080:9000 >> /dev/null &
```

In Spinnaker, open **Applications → sample → Pipelines**. Run the pipeline if needed and approve the production deployment. Check the production frontend over **http**.

Push a tagged change to trigger a new release:

```bash
gcloud source repos clone sample-app
cd sample-app

git config --global user.email "$(gcloud config get-value account)"
git config --global user.name  "Student"

touch release-note
git add release-note
git commit -m "Trigger release v1.0.1"
git tag v1.0.1
git push --tags
```

---

## ⚠️ Notes

- Resource names, zones, regions and ports are generated per lab session. Always use the values shown in **your** lab instructions.
- These steps were written when the lab was live. Google revises labs over time, so the current version may differ.

## 📌 Disclaimer

This repository is a study reference. Attempt the lab on your own first, then compare approaches. It is not affiliated with or endorsed by Google.

## 👤 Author

**Girish Sharma** · Cloud & DevOps Engineer · [GitHub](https://github.com/codewithgarry) · [LinkedIn](https://www.linkedin.com/in/codewithgarry/)
