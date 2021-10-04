import * as k8s from "@pulumi/kubernetes";
import * as pulumi from "@pulumi/pulumi";
import * as docker from "@pulumi/docker";

const appLabels = { app: "nginx" };
const deployment = new k8s.apps.v1.Deployment("nginx", {
    spec: {
        selector: { matchLabels: appLabels },
        replicas: 1,
        template: {
            metadata: { labels: appLabels },
            spec: { containers: [{ name: "nginx", image: "nginx" }] }
        }
    }
});

// About `apply` and `interpolate` => https://www.pulumi.com/docs/intro/concepts/inputs-outputs/#inputs-and-outputs
export const urnExport = deployment.urn.apply( urn => `URN: ` + urn )
export const name = deployment.metadata.name.apply(name => `NAME: ` + name) 


// ########### 
// HELM
// ###########

// Define which K8S we are using
const config = new pulumi.Config();
let cluster = new k8s.Provider("kind-cluster", {
    kubeconfig: "./files/config",
    context: "kind-pulumi-demo",
    cluster: "kind-pulumi-demo",
});

// Deploy latest Armada chart
new k8s.helm.v3.Release("armada", {
    name: "armada-release",
    chart: "armada",
    lint: true,
    cleanupOnFail: true,
    skipAwait: true, // skip waiting green status
    timeout: 120, // seconds
    // version: config.require('armadaVersion'), // if not specified latest will be used
    repositoryOpts: {
        repo: "https://g-research.github.io/charts/",
    },
    values: {
        ingressClass: "nginx",
        clusterIssuer: "dummy-cluster-value",
        hostnames: ["dummy-host-value"],
    },
    // valueYamlFiles: [{}] // Not yet supported e.g docs/quickstart/server-values.yaml
}, {
    provider: cluster,
    customTimeouts: {
        create: "5m"
    },
})

// ########### 
// DOCKER
// ###########

// Run container
new docker.Container("ubuntu", {
    image: "ubuntu:precise",
    attach: true,
    logs: true,
    command: ["cat"],
    entrypoints: ["/bin/bash"],
    rm: true,

});

