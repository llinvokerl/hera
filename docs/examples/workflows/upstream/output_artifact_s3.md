# Output Artifact S3

## Note

This example is a replication of an Argo Workflow example in Hera.
The upstream example can be [found here](https://github.com/argoproj/argo-workflows/blob/main/examples/output-artifact-s3.yaml).




=== "Hera"

    ```python linenums="1"
    from hera.workflows import (
        Container,
        S3Artifact,
        Workflow,
        models as m,
    )

    with Workflow(generate_name="output-artifact-s3-", entrypoint="hello-world-to-file") as w:
        Container(
            name="hello-world-to-file",
            image="busybox",
            command=["sh", "-c"],
            args=["echo hello world | tee /tmp/hello_world.txt"],
            outputs=[
                S3Artifact(
                    name="message",
                    path="/tmp",
                    endpoint="s3.amazonaws.com",
                    bucket="my-bucket",
                    region="us-west-2",
                    key="path/in/bucket/hello_world.txt.tgz",
                    access_key_secret=m.SecretKeySelector(
                        name="my-s3-credentials",
                        key="accessKey",
                    ),
                    secret_key_secret=m.SecretKeySelector(
                        name="my-s3-credentials",
                        key="secretKey",
                    ),
                )
            ],
        )
    ```

=== "YAML"

    ```yaml linenums="1"
    apiVersion: argoproj.io/v1alpha1
    kind: Workflow
    metadata:
      generateName: output-artifact-s3-
    spec:
      entrypoint: hello-world-to-file
      templates:
      - name: hello-world-to-file
        container:
          image: busybox
          args:
          - echo hello world | tee /tmp/hello_world.txt
          command:
          - sh
          - -c
        outputs:
          artifacts:
          - name: message
            path: /tmp
            s3:
              bucket: my-bucket
              endpoint: s3.amazonaws.com
              key: path/in/bucket/hello_world.txt.tgz
              region: us-west-2
              accessKeySecret:
                name: my-s3-credentials
                key: accessKey
              secretKeySecret:
                name: my-s3-credentials
                key: secretKey
    ```

