# Suspend Template Outputs

## Note

This example is a replication of an Argo Workflow example in Hera.
The upstream example can be [found here](https://github.com/argoproj/argo-workflows/blob/main/examples/suspend-template-outputs.yaml).




=== "Hera"

    ```python linenums="1"
    from hera.workflows import Container, Step, Steps, Workflow
    from hera.workflows.models import Outputs, Parameter, SuppliedValueFrom, SuspendTemplate, Template, ValueFrom

    with Workflow(
        name="suspend-outputs",
        entrypoint="suspend",
    ) as w:
        print_message = Container(
            name="print-message",
            image="busybox",
            command=["echo"],
            inputs=[Parameter(name="message")],
            args=["{{inputs.parameters.message}}"],
        )

        with Steps(name="suspend"):
            Step(
                name="approve",
                template=Template(
                    name="approve",
                    suspend=SuspendTemplate(),
                    outputs=Outputs(
                        parameters=[
                            Parameter(
                                name="message",
                                value_from=ValueFrom(supplied=SuppliedValueFrom()),
                            )
                        ]
                    ),
                ),
            )
            print_message(name="release", arguments={"message": "{{steps.approve.outputs.parameters.message}}"})
    ```

=== "YAML"

    ```yaml linenums="1"
    apiVersion: argoproj.io/v1alpha1
    kind: Workflow
    metadata:
      name: suspend-outputs
    spec:
      entrypoint: suspend
      templates:
      - name: print-message
        container:
          image: busybox
          args:
          - '{{inputs.parameters.message}}'
          command:
          - echo
        inputs:
          parameters:
          - name: message
      - name: suspend
        steps:
        - - name: approve
            template: approve
        - - name: release
            template: print-message
            arguments:
              parameters:
              - name: message
                value: '{{steps.approve.outputs.parameters.message}}'
      - name: approve
        outputs:
          parameters:
          - name: message
            valueFrom:
              supplied: {}
        suspend: {}
    ```

