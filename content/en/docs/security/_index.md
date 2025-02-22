---
title: "Security"
description: "Security Processes and Guidelines"
weight: 68
---

Kyverno serves an admission controller and is a critical component of the Kubernetes control-plane. It is important to properly secure and monitor Kyverno. This section provides guidance on securing Kyverno and the security processes for the Kyverno project.

## Disclosure Process

Security vulnerabilities are best handled swiftly and discretely with the goal of minimizing the total time users remain vulnerable to exploits.

If you find or suspect a vulnerability, please email the security group at kyverno-security@googlegroups.com with the following information:
- description of the problem
- precise and detailed steps (include screenshots) that created the problem
- the affected version(s)
- any known mitigations

The Kyverno security response team will send an initial acknowledgement of the disclosure in 3-5 working days. Once the vulnerability and mitigation are confirmed, the team will plan to release any necessary changes based on the severity and complexity. Additional details on the security policy and processes are available in the Kyverno [git repo](https://github.com/kyverno/kyverno/blob/main/SECURITY.md).

## Contact Us

To communicate with the Kyverno team, for any questions or discussions, use [Slack](https://slack.k8s.io/#kyverno) or [GitHub](https://github.com/kyverno/kyverno).


## Issues

All security related issues are labeled as `security` and can be viewed with this query:

  [https://github.com/kyverno/kyverno/labels/security](https://github.com/kyverno/kyverno/labels/security)


## Release Artifacts

The Kyverno container images are available at:

  https://github.com/orgs/kyverno/packages?repo_name=kyverno

With each release, the following artifacts are uploaded:
- checksums.txt
- kyverno-cli_v<version_number>_darwin_x86_64.tar.gz
- kyverno-cli_v<version_number>_linux_x86_64.tar.gz
- kyverno-cli_v<version_number>_windows_x86_64.zip
- Source code (zip)
- Source code (tar.gz)


## Verifying Kyverno Container Images

Kyverno container images are signed using Cosign. To verify the container image, download the organization public key (https://github.com/kyverno/kyverno/blob/main/cosign.pub) into a file named `cosign.pub` and then:

1. Install Cosign
2. Configure the Kyverno signature repository:

```sh
export COSIGN_REPOSITORY=ghcr.io/kyverno/signatures
```

3. Verify the image:

```sh
cosign verify -key cosign.pub ghcr.io/kyverno/kyverno:latest
```

If the container image was properly signed, the output should be similar to:

```sh
Verification for kyverno/kyverno:latest --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
  - Any certificates were verified against the Fulcio roots.
[{"critical":{"identity":{"docker-reference":"ghcr.io/kyverno/kyverno"},"image":{"docker-manifest-digest":"sha256:a847df12e2c1cab19af9d1bb34e599cb56cf57639c7d5c958a4bb568c1dad8f6"},"type":"cosign container image signature"},"optional":null}]
```

All three Kyverno images can be verified.

## Fetching the SBOM for Kyverno

An SBOM (Software Bill of Materials) in CycloneDX JSON format is published for each Kyverno release. To download and verify the SBOM for a specific version, install Cosign and run:

```sh
cosign download sbom ghcr.io/kyverno/sbom:latest
```

To save the SBOM to a file, run the following command:

```sh
cosign download sbom ghcr.io/kyverno/sbom:latest > kyverno.sbom.json
```

## Security Scorecard

Kyverno uses [Scorecards by OSSF](https://github.com/ossf/scorecard) to maintain repository-wide security standards. The current OSSF/scorecard score for Kyverno can be found in this [tracker issue](https://github.com/kyverno/kyverno/issues/2617). The Kyverno team is committed to achieving and maintaining a high score. Contributions are welcome.

## Vulnerability Scan Reports

The Kyverno Helm Chart is available via the [ArtifactHub page](https://artifacthub.io/packages/helm/kyverno/kyverno) along with an auto-generated [Security Report](https://artifacthub.io/packages/helm/kyverno/kyverno?modal=security-report) generated by ArtifactHub for all the releases.


## Threat Model

The [Kubernetes SIG Security](https://github.com/kubernetes/community/tree/master/sig-security) team has defined an [Admission Control Threat Model](https://docs.google.com/document/d/1tQTZ6wrD2udbUGL7xhc3SRxqyTvS8FQUlL59KHYHcY8/edit#heading=h.bx94h7ux7k41). It is highly recommended to understand the identified threats and available mitigations.

The following sections discuss related best practices for Kyverno:

### Pod security

The following Pod security best practices are followed for all Kyverno containers:
* `runAsNonRoot` is set to `true`
* `privileged` is set to `false`
* `allowPrivilegeEscalation` is set to `false`
* `readOnlyRootFilesystem` is set to `true`
* all capabilities are dropped
* limits and quotas are configured
* liveness and readiness checks are configured

### RBAC

Use the following command to view all Kyverno roles:
```sh
kubectl get clusterroles,roles -A | grep kyverno
```

The Kyverno RBAC configurations are described in the [installation section](/docs/installation/#roles-and-permissions). 

It is important to limit Kyverno to the required permissions and audit changes in the RBAC roles and role bindings. In particular, the default `kyverno:view` and `kyverno:generate` roles can be customized to match your requirements.

### Webhooks

Use the following command to view all Kyverno roles:
```sh
kubectl get mutatingwebhookconfigurations,validatingwebhookconfigurations | grep kyverno
```

Kyverno creates the following mutating webhook configurations:
- `kyverno-policy-mutating-webhook-cfg`: handles policy changes to index and cache policy sets.
- `kyverno-resource-mutating-webhook-cfg`: handles resource admission requests to apply matching Kyverno mutate policy rules.
- `kyverno-verify-mutating-webhook-cfg`: periodically tests Kyverno webhook configurations 

Kyverno creates the following validating webhook configurations:
- `kyverno-policy-validating-webhook-cfg`: validates Kyverno policies with checks that cannot be performed via schema validation
- `kyverno-resource-validating-webhook-cfg`: handles resource resource admission requests to apply matching Kyverno validate policy rules.

#### Webhook Failure Mode

Kyverno policies are configured to **fail-closed** by default. This setting can be tuned on a [per policy basis](/docs/writing-policies/policy-settings/). Kyverno uses the configured policy set to automatically configure webhooks.

#### Webhook authentication and encryption

By default, Kyverno automatically generates and manage TLS certificates used for mutual authentication with the API server, and encryption of network traffic. To customize, please refer to the details in the [installation section](/docs/installation/#certificate-management).

### Recommended policies

The Kyverno community manages a set of [sample policies](/policies/).

At a minimum, the [Pod Security Standards](/policies/pod-security/) and [best practices](/policies/?policytypes=Best%2520Practices) policy sets are recommended for use.

### Securing policies

Kyverno policies can be used to mutate and generate namespaced and cluster-wide resources. Hence, policies should be treated as critical resources and access to policies should be protected using RBAC.