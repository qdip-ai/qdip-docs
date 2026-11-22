# QDIP — szybki start dla twórców wtyczek (polski)

[English](quickstart.en.md) · [Українська](quickstart.uk.md) · [Indeks](README.md)

**Zakres:** integracja **SDK-02 remote Worker → Gateway → managed Kernel** zatwierdzona przez operatora. To **nie** jest samodzielna rejestracja publiczna, certyfikacja SDK ani potwierdzone wdrożenie produkcyjne. Dostęp do prywatnego kodu QDIP nie jest wymagany. Worker przygotowuje `DecisionSpec`; Kernel optymalizuje i tworzy manifest.

## 1. Wymagania

1. Za pośrednictwem **istniejącego uwierzytelnionego kanału pomocy** poproś operatora o zatwierdzone organization/application, plugin ID, dostęp do capability `repairs.plan`, adres Gateway i **dwa osobne** klucze: Worker (`plugins:register`) i serwerowy caller (`plugins:read`, `plugins:execute`). Nie deklarujemy publicznego automatycznego wydawania kluczy.
2. Odbierz **zbudowane przez operatora** `qdip-sdk02-bundle-0.1.3.zip` oraz `qdip-client-0.2.0.tgz` wraz z SHA-256 **z zaufanego kanału**. Sprawdź pochodzenie i sumy kontrolne; sam hash nie dowodzi autentyczności. Wheels muszą pasować do wersji Python i OS. Są to **przykładowe wersje**, a nie gwarancja publicznej dostępności.
3. Wymagane: Python 3, Node.js >=20, własny host. Nie używaj kluczy testowych, nie commituj sekretów i nie przekazuj klucza Workera do przeglądarki.

## 2. Napisz własny adapter algorytmu

Utwórz `my_plugin.py` w katalogu Workera. Przykład waliduje wejście i zwraca pełny `DecisionSpec` dla Kernel; **nie** wykonuje CP-SAT ani nie zgłasza gotowego rozwiązania. Zastąp go własnym modelem z zachowaniem kontraktu.

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

## 3. Zweryfikuj artefakty i połącz Workera

Polecenia służą do **zatwierdzonej integracji**, nie do anonimowej rejestracji. Przed uruchomieniem ustaw `SDK02_SHA256` i `SDK03_SHA256` na otrzymane niezależnie 64-znakowe hashe; zatrzymaj się, jeśli brakuje wartości lub kontrola nie przejdzie. Worker musi pozostać aktywny.

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

## 4. Wywołaj Worker z osobnego klienta serwerowego

Najpierw zapisz kod jako `caller/invoke.mjs`, potem wykonaj polecenia. Nazwa pakietu musi odpowiadać dostarczonemu plikowi tarball.

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

## 5. Akceptacja i obsługa błędów

Dla przykładowych danych optimum wynosi **13** (`b` i `c`). HTTP 200 ani samo REGISTER Workera nie oznacza zakończonej integracji. Zapisz wersje SDK, hashe artefaktów, rzeczywisty deployment SHA Gateway, identyfikatory result/manifest/replay, odmowę obcej capability i cofniętego klucza. Dopiero autoryzowany test potwierdza E2E.

- `401/403`: sprawdź klucze i powiązanie organization/application/capability u operatora; nie ponawiaj odwołanego klucza.
- `422`: popraw wejście/schema lub wersję. `429`: respektuj `Retry-After`. `503`: sprawdź Worker i limit. `504`/utracona odpowiedź: wynik może być nieznany, nie duplikuj automatycznie działań.
- Trzymaj klucze tylko na backendzie. Nie publikuj zrzutów środowiska, śladów lub screenshotów z tokenami.

**Otwarte gates:** self-service publication, publiczna dystrybucja pakietów, niezależne potwierdzenie provenance i produkcyjna kwalifikacja bezpieczeństwa **NOT VERIFIED**. W starszym Core istnieje `structured_result`, ale planowane jest jego usunięcie; ten przewodnik używa wyłącznie managed `decision_spec`. Zobacz [zasady bezpiecznej publikacji](../security/publication-safety.md).
