# Quantum-Service Descriptor Examples for Hybrid 6G--Quantum Infrastructures

This repository contains illustrative YAML descriptor instances for managing quantum services over hybrid 6G--quantum infrastructures. The examples follow the descriptor-based information model proposed in the manuscript:

> Information Model and Management Architecture for Quantum Services over Hybrid 6G--Quantum Infrastructures  
> Oscar Adamuz-Hinojosa, Jonathan Prados-Garzon, Natalia Chinchilla-Romero, and Juan M. Lopez-Soler

The repository is intended to provide machine-readable examples of the descriptor instances discussed in the paper. The descriptors are aligned with the illustrative remote quantum-execution service and feasibility-accounting example presented in Section IV and Table II of the manuscript.

## Scope

The descriptors describe a remote quantum-execution service in which an authorized quantum-enabled user equipment (QUE) transfers four input qubits to a remote quantum processing unit (QPU) through delayed multi-qubit teleportation. The remote-QPU operation is treated as a gated downstream step: it can start only after all requested qubits have been reconstructed at the remote QPU.

The numerical feasibility evidence focuses on the teleportation stage that enables the downstream remote-QPU operation. These values are illustrative and simulation-backed; they should not be interpreted as a general validation claim for all hybrid 6G--quantum deployments.

## Repository contents

```text
.
├── service_descriptor.yaml
├── quantum_resource_descriptor.yaml
├── classical_support_descriptor.yaml
├── workflow_descriptor.yaml
├── deployment_descriptor.yaml
├── feasibility_descriptor.yaml
├── LICENSE
└── README.md
```

## Descriptor overview

| File | Descriptor type | Purpose |
|---|---|---|
| `service_descriptor.yaml` | `service_descriptor` | Captures the tenant-facing quantum-service request, including the operation type, endpoints, service objectives, quantum-resource demand, workflow reference, and policy constraints. |
| `quantum_resource_descriptor.yaml` | `quantum_resource_descriptor` | Captures the quantum-resource context used by planning and admission control, including the selected quantum path, memory state, Bell-pair generation capabilities, swapping support, QPU availability, and validity information. |
| `classical_support_descriptor.yaml` | `classical_support_descriptor` | Captures the 6G/classical-support context, including the selected support path, slice/QoS treatment, latency, jitter, packet loss, synchronization, edge-control reachability, telemetry, and validity information. |
| `workflow_descriptor.yaml` | `service_workflow_descriptor` | Captures the ordered quantum--classical workflow, including Bell-pair generation, heralding, swapping, measurement-result exchange, Pauli correction, retry handling, and remote-QPU execution gating. |
| `deployment_descriptor.yaml` | `service_deployment_descriptor` | Captures one deployable realization of the requested service by binding the service descriptor, the quantum-resource option, the 6G/classical-support option, and the workflow variant. |
| `feasibility_descriptor.yaml` | `feasibility_descriptor` | Captures the explicit feasibility assessment record associated with the evaluated deployment alternative, including fidelity, timing, reliability, classical-support margins, validity, confidence, and admission status. |

## Example service

The example models the following service request:

| Attribute | Value |
|---|---|
| Service type | Remote quantum execution |
| Quantum transfer mechanism | Delayed multi-qubit teleportation |
| Input qubits to teleport | 4 |
| Required end-to-end Bell pairs | 4 |
| Minimum reconstructed-qubit fidelity | 0.85 |
| Teleportation-stage deadline | 1 s |
| Teleportation-stage reliability target | 0.95 |
| Classical-support priority | High priority |
| Remote execution condition | Launch only after all requested qubits are available at the QPU |

## Feasibility evidence represented in the example

The baseline deployment is admitted according to the following feasibility evidence:

| Evidence item | Estimated value | Target or bound | Status |
|---|---:|---:|---|
| Reconstructed-qubit fidelity | 0.963 | >= 0.85 | Admit |
| Teleportation-stage completion time | 368.115 ms | <= 1 s | Admit |
| Link-level Bell-pair generation success | > 0.9999 | >= 0.95 | Admit |
| Teleportation-stage service reliability | > 0.9999 | >= 0.95 | Admit |
| Memory-occupation time | 322.100 ms | <= 10 s coherence time | Admit |
| Critical support latency | 23.000 ms | <= 25 ms | Admit |
| Packet loss | 4.00e-05 | <= 1.00e-03 | Admit |
| Synchronization accuracy | 100 ns | <= 1 us | Admit |
| Final admission decision | Admit | All hard constraints satisfied | Admit |

For constraints expressed as `estimated_value <= bound`, the YAML files retain the margin convention used in the manuscript table, where a negative margin indicates that the estimated value is below the bound. Additional positive `slack_*` fields are included where useful for runtime monitoring and threshold-based adaptation.

## Descriptor relationships

The descriptors are linked through explicit identifiers and references:

```text
service_descriptor.yaml
        |
        | service_ref: svc-01
        v
quantum_resource_descriptor.yaml
classical_support_descriptor.yaml
workflow_descriptor.yaml
        |
        | input_descriptor_refs
        v
deployment_descriptor.yaml
        |
        | deployment_ref: depl-01
        v
feasibility_descriptor.yaml
```

The intended lifecycle is:

1. A tenant request is formalized as a `service_descriptor`.
2. Current quantum-resource and 6G/classical-support conditions are represented through the `quantum_resource_descriptor` and `classical_support_descriptor`.
3. The ordered quantum--classical workflow is represented through a `service_workflow_descriptor`.
4. Deployment alternatives bind the service request, quantum-resource option, 6G/classical-support option, and service workflow.
5. Admission control evaluates each deployment alternative and stores the explicit feasibility assessment record in a `feasibility_descriptor`.
6. Runtime telemetry can refresh descriptor state, update feasibility-status flags, and trigger bounded adaptation or reassessment when admitted margins become stale, marginal, or violated.

## Classical-support latency values

Critical quantum-control traffic is aligned with the manuscript example:

```yaml
target_mean_one_way_latency_us: 10000
target_p95_one_way_latency_us: 23000
max_one_way_latency_us: 25000
```

The feasibility evidence uses the 23 ms high-percentile latency value against a 25 ms one-way latency bound. Orchestration traffic uses a looser envelope:

```yaml
target_mean_one_way_latency_us: 20000
target_p95_one_way_latency_us: 30000
max_one_way_latency_us: 50000
```

These envelopes avoid inconsistent statistics such as a mean latency larger than the p95 latency or the maximum admitted bound.

## Basic validation

The YAML files are intended to be directly parseable by standard YAML parsers. For a basic syntax check, run:

```bash
python - <<'PY'
from pathlib import Path
import yaml

for path in sorted(Path('.').glob('*.yaml')):
    with path.open('r', encoding='utf-8') as f:
        yaml.safe_load(f)
    print(f'OK: {path}')
PY
```

Optional style validation can be performed with `yamllint`:

```bash
yamllint .
```

These checks validate YAML syntax and style only. They do not validate cross-descriptor consistency, physical feasibility, or compliance with a future standardized schema.

## Notes on interpretation

- The descriptor structure is illustrative and follows the information categories proposed in the manuscript.
- The examples are not a normative schema specification.
- Identifiers such as `svc-01`, `qpath-01`, `depl-01`, and `feas-01` are local example identifiers.
- The numerical evidence is aligned with the manuscript example and corresponds to a simulation-backed feasibility assessment for the teleportation stage.
- The downstream remote-QPU operation is represented as an execution-gated step; the numerical feasibility accounting focuses on the teleportation stage that enables this operation.
- The descriptors are designed to be extensible toward schema-based validation, descriptor versioning, telemetry refresh, and cross-domain orchestration.

## Suggested citation

If you use these descriptor examples, please cite the associated manuscript:

```bibtex
@article{AdamuzHinojosa2026QuantumServiceDescriptors,
  author  = {Oscar Adamuz-Hinojosa and Jonathan Prados-Garzon and Natalia Chinchilla-Romero and Juan M. Lopez-Soler},
  title   = {{Information Model and Management Architecture for Quantum Services over Hybrid 6G--Quantum Infrastructures}},
  journal = {Submitted manuscript},
  year    = {2026}
}
```
