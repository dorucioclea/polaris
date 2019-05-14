<p align="center">
  <img src="/pkg/dashboard/assets/images/logo.png" alt="Polaris Logo" />
</p>

Polaris keeps your cluster sailing smoothly. It runs a variety of checks to ensure that Kubernetes deployments are configured using best practices that will avoid potential problems in the future. The project includes two primary parts:

- A dashboard to display the results of these validations on your existing deployments
- A beta version of a webhook that can prevent poorly configured deployments from reaching your cluster

## Dashboard

The Polaris Dashboard provides an overview of your current deployments in a cluster along with their validation scores. An overall score is provided for a cluster on a 0 - 100 scale. Results are then broken down by namespace and deployment.

<p align="center">
  <img src="/dashboard-screenshot.png" alt="Polaris Dashboard" />
</p>

### Deploying

To deploy Polaris with kubectl:

```
kubectl apply -f https://raw.githubusercontent.com/reactiveops/polaris/master/deploy/dashboard.yaml
```

Polaris can also be deployed with Helm:

```
helm upgrade --install polaris deploy/helm/polaris/ --namespace polaris
```

### Viewing the Dashboard

Once the dashboard is deployed, it can be viewed by using kubectl port-forward:

```
kubectl port-forward --namespace polaris svc/polaris-dashboard 8080:80
```

With the port forwarding in place, you can open http://localhost:8080 in your browser to view the dashboard.

### Using a Binary Release

If you'd prefer to run Polaris locally, binary releases are available on the [releases page](https://github.com/reactiveops/polaris/releases). When running as a binary, Polaris will use your local kubeconfig to connect to a cluster. There are a variety of options available, but the most common usage may be to view the dashboard:

```
polaris --dashboard
```

## Webhook

Polaris includes experimental support for an optional validating webhook. This accepts the same configuration as the dashboard, and can run the same validations. This webhook will reject any deployments that trigger a validation error. This is indicative of the greater goal of Polaris, not just to encourage better configuration through dashboard visibility, but to actually enforce it with this webhook. *Although we are working towards greater stability and better test coverage, we do not currently consider this webhook component production ready.*

Unfortunately we have not found a way to disply warnings as part of `kubectl` output unless we are rejecting a deployment altogether. That means that any checks with a severity of `warning` will still pass webhook validation, and the only evidence of that warning will either be in the Polaris dashboard or the Polaris webhook logs.

### Deploying

The Polaris webhook can be deployed with kubectl:

```
kubectl apply -f https://raw.githubusercontent.com/reactiveops/polaris/master/deploy/webhook.yaml
```

Alternatively, the webhook can be enabled with Helm by setting `webhook.enable` to true:

```
helm upgrade --install polaris deploy/helm/polaris/ --namespace polaris --set webhook.enable=true
```


## Configuration

Polaris supports a wide range of validations covering a number of Kubernetes best practices. Here's a sample configuration file that includes all currently supported checks. The [default configuration](https://github.com/reactiveops/polaris/blob/master/config.yaml) contains a number of those checks. This repository also includes a sample [full configuration file](https://github.com/reactiveops/polaris/blob/master/config-full.yaml) that enables all available checks.

Each check can be assigned a `severity`. Only checks with a severity of `error` or `warning` will be validated. The results of these validations are visible on the dashboard. In the case of the validating webhook, only failures with a severity of `error` will result in a change being rejected.

Polaris validation checks fall into several different categories:

- [Health Checks](docs/health-checks.md)
- [Images](docs/images.md)
- [Networking](docs/networking.md)
- [Resources](docs/resources.md)
- [Security](docs/security.md)

## CLI Options

* `config`: Specify a location for the Polaris config
* `dashboard`: Runs the webserver for Polaris dashboard.
* `dashboard-port`: Port for the dashboard webserver (default 8080)
* `webhook`: Runs the webhook webserver.
* `webhook-port`: Port for the webhook webserver (default 9876)
* `disable-webhook-config-installer`: disable the installer in the webhook server, so it won't install webhook configuration resources during bootstrapping
* `kubeconfig`: Paths to a kubeconfig. Only required if out-of-cluster.

## License
Apache License 2.0