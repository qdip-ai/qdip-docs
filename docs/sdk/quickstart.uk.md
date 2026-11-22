# Швидкий старт розробника плагіна QDIP — українська

[English](quickstart.en.md) · [Polski](quickstart.pl.md) · [Індекс](README.md)

**Межі:** контрольоване підключення **SDK-02 remote Worker → Gateway → managed Kernel** із допуском оператора. Це **не** самостійна публічна реєстрація, не сертифікація SDK і не підтверджене production-розгортання. Доступ до приватного коду QDIP не потрібен. Worker формує `DecisionSpec`; Kernel виконує оптимізацію та створює manifest.

## 1. Умови підключення

1. Через **наявний автентифікований канал підтримки** отримайте від оператора погоджену organization/application, plugin ID, допуск capability `repairs.plan`, Gateway origin і **два окремі** ключі: Worker (`plugins:register`) та серверний caller (`plugins:read`, `plugins:execute`). Публічна автоматична видача ключів не заявляється.
2. Отримайте **зібрані оператором** `qdip-sdk02-bundle-0.1.3.zip` і `qdip-client-0.2.0.tgz` та SHA-256 **через довірений канал**. Перед встановленням перевірте походження та хеші; сам хеш не доводить походження артефакту. Wheel має відповідати Python/OS. Це **приклади зафіксованих версій**, а не доказ публічної доступності.
3. Потрібні Python 3, Node.js >=20 та власний хост. Не використовуйте fixture keys, не комітьте секрети і не передавайте Worker key у браузер.

## 2. Створіть власний адаптер алгоритму

Створіть `my_plugin.py` у робочому каталозі Worker. Приклад перевіряє вхід і повертає повний `DecisionSpec` для Kernel, **не** виконує CP-SAT і не підміняє розв'язок. Замініть тестову модель власною, зберігши контракт.

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

## 3. Перевірте артефакти та підключіть Worker

Ці команди працюють для **погодженої інтеграції**, а не анонімного користувача. Перед запуском встановіть `SDK02_SHA256` і `SDK03_SHA256` з незалежно отриманих 64-символьних хешів; за відсутності або невідповідності зупиніться. Залиште Worker запущеним.

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

## 4. Викликайте Worker окремим серверним клієнтом

Спочатку створіть `caller/invoke.mjs` за наведеним кодом, потім виконайте команди. Назва пакета має відповідати виданому tarball.

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

## 5. Перевірки та помилки

Для цього набору задач оптимум — **13** (`b` і `c`). HTTP 200 або сам факт реєстрації Worker недостатні. Зафіксуйте версії SDK, хеші переданих артефактів, фактичний deployment SHA Gateway, result/manifest/replay, відмову для чужої capability та відкликаного ключа. Лише реальний авторизований запуск підтверджує E2E.

- `401/403`: перевірте ключі, organization/application/capability binding з оператором; не повторюйте запит зі скасованим ключем.
- `422`: виправте дані/schema або версію. `429`: дотримуйтеся `Retry-After`. `503`: перевірте Worker та ліміти. `504`/втрачена відповідь: результат може бути невідомий; не дублюйте side effects автоматично.
- Ключі лише на backend. Не публікуйте дампи середовища, trace payload або скриншоти із токенами.

**Невиконані gates:** self-service publication, публічна дистрибуція пакетів, незалежне підтвердження provenance і production security/release **NOT VERIFIED**. Режим `structured_result` у старішому Core існує, але його видалення заплановане; цей посібник використовує тільки managed `decision_spec`. Див. [правила безпечної публікації](../security/publication-safety.md).
