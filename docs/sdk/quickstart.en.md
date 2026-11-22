# QDIP plugin developer quickstart — English

[Українська](quickstart.uk.md) · [Polski](quickstart.pl.md) · [Index](README.md)

**Scope:** operator-assisted, approved **SDK-02 remote Worker → Gateway → managed Kernel** integration. This is **not** self-service registration, public release certification or a production deployment claim. It does not require access to QDIP private source repositories. Your Worker transforms domain input into a `DecisionSpec`; QDIP Kernel owns optimization and the execution manifest.

## 1. Prerequisites / Умови / Wymagania

1. Ask the QDIP platform operator through an **existing authenticated support channel** for an approved organization/application, plugin ID, `repairs.plan` capability admission, Gateway origin, and **two separate** scoped credentials: Worker (`plugins:register`) and server-side caller (`plugins:read`, `plugins:execute`). No public key-issuance endpoint is promised.
2. Obtain the *operator-built* `qdip-sdk02-bundle-0.1.3.zip` and `qdip-client-0.2.0.tgz` plus their SHA-256 checksums **over a trusted channel**. Verify the artifact origins and checksums before installing; hashes alone are not origin attestation. A compatible Python wheel bundle is required. These are **example pinned artifact versions**, not proof of public availability.
3. Use Python 3 with a compatible wheel bundle, Node.js >=20 and a private development host. Do not use fixture credentials, embed keys in source files or run Worker credentials in a browser.

## 2. Implement your own algorithm adapter

Create `my_plugin.py` in the Worker working directory. This minimal **model-building adapter** validates input and returns a complete Kernel `DecisionSpec`, rather than running CP-SAT or returning a fabricated solution. Replace the sample with your own model while preserving the contract.

```python
# my_plugin.py — runs on your own Worker host
import re

def make_spec(payload: dict, decision_id: str) -> dict:
    capacity = payload.get("capacity_hours")
    jobs = payload.get("jobs")
    if type(capacity) is not int or not 0 <= capacity <= 100:
        raise ValueError("Invalid capacity_hours")
    if not isinstance(jobs, list) or not 1 <= len(jobs) <= 8:
        raise ValueError("Expected 1–8 jobs")
    variables, objective, budget, names = [], [], [], set()
    for job in jobs:
        if not isinstance(job, dict) or set(job) != {"id", "hours", "priority"}:
            raise ValueError("Invalid job")
        name, hours, priority = job["id"], job["hours"], job["priority"]
        if not isinstance(name, str) or not re.fullmatch(r"[a-z][a-z0-9_]{0,15}", name) or name in names:
            raise ValueError("Duplicate or invalid job id")
        if type(hours) is not int or not 1 <= hours <= 100:
            raise ValueError("Invalid hours")
        if type(priority) is not int or not 0 <= priority <= 100:
            raise ValueError("Invalid priority")
        names.add(name)
        variable = "job_" + name
        variables.append({"id": variable, "kind": "BOOLEAN"})
        objective.append({"variable_id": variable, "coefficient": str(priority)})
        budget.append({"variable_id": variable, "coefficient": str(hours)})
    return {
        "semantic_version": "1.0.0", "decision_id": decision_id,
        "variables": variables,
        "objective": {"sense": "MAXIMIZE", "terms": objective},
        "constraints": [{"id": "hours_budget", "relation": "LE",
                         "terms": budget, "rhs": str(capacity)}],
    }
```

## 3. Verify and install approved artifacts; connect the Worker

The following commands are for a **controlled approved integration**, not an unauthenticated public account. Set `SDK02_SHA256` and `SDK03_SHA256` to the independently supplied 64-hex values before executing; stop if they are absent or wrong. Keep the Worker running.

```bash
# On your own Linux/macOS development machine, using a trusted operator handoff:
printf '%s  %s\n' "$SDK02_SHA256" qdip-sdk02-bundle-0.1.3.zip | sha256sum -c -
printf '%s  %s\n' "$SDK03_SHA256" qdip-client-0.2.0.tgz | sha256sum -c -
unzip -q qdip-sdk02-bundle-0.1.3.zip -d sdk02
python3 -m venv .venv-qdip
source .venv-qdip/bin/activate
python -m pip install --no-index --find-links sdk02/wheels qdip-remote-sdk==0.1.3

# Worker key is scoped to this plugin, organization, application and capability.
export QDIP_REMOTE_URL='wss://<operator-issued-host>/api/v1/remote-plugins/connect'
export QDIP_PLUGIN_ID='<operator-issued-plugin-id>'
read -r -s -p 'Worker key: ' QDIP_REMOTE_API_KEY; printf '\n'
export QDIP_REMOTE_API_KEY
qdip-worker --handler my_plugin:make_spec --plugin-id "$QDIP_PLUGIN_ID" --capability-id repairs.plan
```

## 4. Invoke with a separate server-side caller

Create `caller/invoke.mjs` from the code below *before* running the commands. Ensure the installed package name matches the issued tarball.

```javascript
// caller/invoke.mjs — server-side only, never inside a browser bundle
import { QdipClient, isActionable } from '@qdip/client';
const sdk = new QdipClient({
  baseUrl: process.env.QDIP_GATEWAY_URL,
  apiKey: process.env.QDIP_API_KEY,
});
const result = await sdk.plugins.invoke({
  capabilityId: 'repairs.plan', capabilityVersion: '1',
  input: { capacity_hours: 4, jobs: [
    { id: 'a', hours: 3, priority: 9 },
    { id: 'b', hours: 2, priority: 7 },
    { id: 'c', hours: 2, priority: 6 },
  ] },
  timeoutSeconds: 10, deadlineMs: 45000,
});
if (!isActionable(result) || String(result.execution_result.objective_value) !== '13') {
  throw new Error('Expected actionable managed Kernel result');
}
console.log(result.execution_result.execution_id);
```

```bash
# In a separate terminal, from the same developer workspace:
mkdir caller && cd caller
npm init -y
npm pkg set type=module
npm install ../qdip-client-0.2.0.tgz
export QDIP_GATEWAY_URL='https://<operator-issued-host>/'
read -r -s -p 'Caller key: ' QDIP_API_KEY; printf '\n'
export QDIP_API_KEY
node invoke.mjs
```

## 5. Acceptance and failure behavior

For the example input the optimal objective is **13** (`b` and `c`). An HTTP 200 or Worker registration alone is not acceptance. Record the Worker/SDK versions, issued artifact digests, real Gateway deployment revision, result/manifest/replay identifiers, wrong-capability denial and revoked-key denial. Only a real authorized run can establish integration E2E.

- `401/403`: verify credentials, organization/application/capability binding with the operator. Do not repeatedly retry a revoked key.
- `422`: fix input/schema or capability version. `429`: respect `Retry-After`. `503`: check Worker connectivity/capacity. `504`/lost response: outcome may be unknown, do not automatically duplicate side effects.
- Keep the caller key and Worker key on backend hosts. Do not publish environment dumps, trace payloads or screenshots containing tokens.

**Pending platform gates:** third-party self-service publication, public package registry distribution, independent artifact provenance and production security/release qualification are **NOT VERIFIED**. A separate `structured_result` mode currently exists in older Core sources but its removal is proposed; this quickstart intentionally uses only the managed `decision_spec` path. See [safe-publication policy](../security/publication-safety.md).
