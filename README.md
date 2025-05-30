# Kubectl safe-apply Plugin

A plugin for kubectl that matches a special annotation in applied YAML with the current kubectl context, preventing you from accidentally applying files meant for one context to another.

## Installation

* Make sure you have [uv](https://docs.astral.sh/uv/getting-started/installation/) installed.

### Linux and MacOS

* Put `kubectl-safe_apply.py` somewhere on your path
* Make it executable (`chmod +x /path/to/kubectl-safe_apply.py`)

### Windows

* Put `kubectl-safe_apply.py` and `kubectl-safe_apply.cmd` somewhere on your path.

## Usage

* Try applying a k8s YAML file using `safe-apply`:

    ```bash
    $ kubectl safe-apply -f my-file.yaml
    Attempting to apply Pod 'my-pod' without a safe-apply/require-context annotation. Aborting.
    ```

* Now add the annotation specifying which context is required:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    annotations:
        safe-apply/require-context: my-production-cluster
    name: my-pod
    ...
    ```

* Try safe-applying again:

    ```bash
    $ kubectl safe-apply -f my-file.yaml
    Attempting to apply to incorrect context for Pod 'my-pod'. Current: default, required: my-production-cluster
    ```

* Switch to the correct context (using something like [`kubectx`](https://github.com/ahmetb/kubectx)) and try again:

    ```bash
    $ kubectx my-production-cluster
    $ kubectl safe-apply -f my-file.yaml
    pod/my-pod configured
    ```

## Notes

* All arguments to `kubectl safe-apply` are passed through as-is to `kubectl apply`, assuming the context validation succeeds.

* If any objects in the YAML file don't have the correct context annotation, nothing will be applied.

* The `safe-apply/require-context` annotation **must** be present for the apply to succeed, as demonstrated above. This means you can't accidentally apply a normal YAML file to an important cluster just by forgetting to add the annotation. If you're using `safe-apply`, you can trust that every object being applied explicitly names the context you're applying it to.

* You don't have to use stateful context selection. `--cluster` will also work:

    ```bash
    $ kubectx default
    $ kubectl safe-apply --context my-production-cluster -f my-file.yaml
    pod/my-pod configured
    ```


* If it's safe to apply a particular YAML file to multiple clusters, you can specify them as a list in the annotation:


    ```yaml
    ...
    annotations:
        safe-apply/require-context: 
        - my-production-cluster
        - another-cluster
    ...
    ```
