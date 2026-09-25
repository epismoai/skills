# Reuse

Use this guide to discover, evaluate, and apply existing playbook guidance.

## Discover narrowly

Search by the intended outcome, then compare promising candidates' complete
definitions before following one. Pin an immutable version when the work must
be reproducible; later publications should not silently change its guidance.

## Evaluate fit

Check:

- whether the outcome matches the request;
- whether the input satisfies the version's JSON Schema;
- whether step assumptions fit the current context;
- whether hinted resources are available, trusted, and authorized;
- whether expected outputs are sufficient to judge success;
- whether a pinned older version is intentional.

Expected outputs are hints for evaluating fit, not completion criteria.

## Choose the lightest reuse

- **Local reuse:** follow the playbook without creating Epismo execution state.
- **Start a case:** when the result, handoff, or collaboration should persist, start a case from that version. See [Coordinate](./coordinate.md) to start, resume, or continue a case.

When reuse exposes a reusable defect, follow [Improve](./improve.md).
