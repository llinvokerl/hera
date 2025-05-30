# Synchronization Tmpl Level Legacy

## Note

This example is a replication of an Argo Workflow example in Hera.
The upstream example can be [found here](https://github.com/argoproj/argo-workflows/blob/main/examples/synchronization-tmpl-level-legacy.yaml).




=== "Hera"

    ```python linenums="1"
    from hera.workflows import (
        Container,
        Steps,
        Workflow,
        models as m,
    )

    with Workflow(
        generate_name="synchronization-tmpl-level-",
        entrypoint="synchronization-tmpl-level-example",
    ) as w:
        acquire_lock = Container(
            name="acquire-lock",
            image="alpine:latest",
            command=["sh", "-c"],
            args=["sleep 10; echo acquired lock"],
            synchronization=m.Synchronization(
                semaphore=m.SemaphoreRef(config_map_key_ref=m.ConfigMapKeySelector(name="my-config", key="template"))
            ),
        )
        with Steps(name="synchronization-tmpl-level-example") as s:
            acquire_lock(
                name="synchronization-acquire-lock",
                arguments={"seconds": "{{item}}"},
                with_param='["1","2","3","4","5"]',
            )
    ```

=== "YAML"

    ```yaml linenums="1"
    apiVersion: argoproj.io/v1alpha1
    kind: Workflow
    metadata:
      generateName: synchronization-tmpl-level-
    spec:
      entrypoint: synchronization-tmpl-level-example
      templates:
      - name: acquire-lock
        container:
          image: alpine:latest
          args:
          - sleep 10; echo acquired lock
          command:
          - sh
          - -c
        synchronization:
          semaphore:
            configMapKeyRef:
              name: my-config
              key: template
      - name: synchronization-tmpl-level-example
        steps:
        - - name: synchronization-acquire-lock
            template: acquire-lock
            withParam: '["1","2","3","4","5"]'
            arguments:
              parameters:
              - name: seconds
                value: '{{item}}'
    ```

