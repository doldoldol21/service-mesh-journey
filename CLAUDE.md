# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A hands-on learning repository for Service Mesh technologies in Kubernetes. This is a documentation-and-experiment repo (not a software application) — content is primarily Korean-language technical notes, OpenTofu IaC, and Kubernetes manifests.

## Repository Structure

- `docs/concepts/` — Numbered concept docs (e.g., `01-envoy-fundamentals.md`). Sequential learning path.
- `infrastructure/` — IaC for lab environments: Hetzner k3s cluster (OpenTofu).
- `experiments/` — Self-contained experiment directories, each with its own README documenting purpose/procedure/results/lessons.
- `notes/` — Troubleshooting logs and debugging notes.

## Environment Context

- **Target cluster**: Hetzner CAX21 × 2 (ARM64, 4 vCPU / 8GB RAM) running k3s
- **IaC**: OpenTofu
- **Observability stack**: Prometheus, Grafana, Kiali, Jaeger
- **Mesh progression**: Istio (sidecar) → Istio (ambient) → Cilium Service Mesh

## Content Conventions

- All documentation is written in Korean.
- Concept docs follow the pattern `NN-topic-name.md` with sequential numbering.
- Experiments are isolated directories with a README covering: 목적(purpose), 절차(procedure), 결과(result), 배운점(lessons learned).
- Commits bundle code/config and documentation together (not separately).

## Learning Roadmap

Week 1: Environment setup + Online Boutique without mesh → Week 2: Istio basics (traffic management, Bookinfo) → Week 3: Chaos Mesh fault injection → Week 4: Envoy config analysis + WASM plugin → Week 5+: Cilium Service Mesh migration.
