# OCI Helm Charts & Cosign

For demonstrating basics of Flux with Helm/OCI and Cosign.


# Steps

## 0. Prepare an empty cluster

We're using vcluster today, (but nothing about this demo requires a particular cluster or provider)

## 1. Deploy Podinfo based on Helm instructions

The podinfo repo provides [Continuous Delivery](https://github.com/stefanprodan/podinfo#continuous-delivery) instructions.
Follow them to get a minimal Flux on your cluster which only installs Helm and Source controller, and podinfo via Helm.

The podinfo deployment should appear in the `default` namespace. Use `--export` to get the declarative manifests out.
We will convert them to use OCI instead of a traditional Helm Repository in the next step.

## 2. Convert Podinfo HelmRepository to `type: oci`

An example in `solutions/`, or follow the details on your own Helm chart as covered on the
[Flux Blog](https://fluxcd.io/blog/2022/11/verify-the-integrity-of-the-helm-charts-stored-as-oci-artifacts-before-reconciling-them-with-flux/#diy-do-it-yourself-approach)

`tl;dr`: add the `type: oci` field to spec, and convert the URL as follows (to an OCI Repository):

```
flux create source helm podinfo-charts --namespace=default --url=oci://ghcr.io/stefanprodan/charts --interval=10m --export
```

## 3. Confirm that OCI Repository still works

The `sourceRef` in your `HelmRelease` should read as follows now:

```
sourceRef:
  kind: HelmRepository
  name: podinfo-charts
```

## 4. Add (keyless) signature verification

(From the Flux Blog linked above):

```
yq e '.spec.chart.spec|=({"verify": { "provider": "cosign" } } +.)' ./clusters/my-cluster/prometheus-helmrelease.yaml
```

## 5. Check the HelmChart for ready state and condition `SourceVerified`

Apply the changed manifests, give them a second for Flux to begin reconciling, and review them to see if it worked!

## Recap: what we have done

We are now verifying the Helm charts before we apply them in the cluster. We haven't secured the whole pipeline from end-to-end, but... that's progress!
