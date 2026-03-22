# Template: use-api-call-hook

File: `frontend/src/hooks/useApiCall.ts`

```typescript
import { useCallback, useRef, useState } from 'react';
import { ApiError } from '../api/axios.config';

interface ApiCallState<T> {
  data: T | null;
  error: ApiError | null;
  isLoading: boolean;
}

const initialState = <T>(): ApiCallState<T> => ({
  data: null,
  error: null,
  isLoading: false
});

export function useApiCall<T, Args extends unknown[]>(
  apiFn: (...args: Args) => Promise<T>
) {
  const fnRef = useRef(apiFn);
  fnRef.current = apiFn;

  const [state, setState] = useState<ApiCallState<T>>(initialState);

  const execute = useCallback(async (...args: Args): Promise<T | null> => {
    setState({ ...initialState(), isLoading: true });
    try {
      const data = await fnRef.current(...args);
      setState({ ...initialState(), data });
      return data;
    } catch (err) {
      setState({ ...initialState(), error: ApiError.from(err) });
      return null;
    }
  }, []);

  const reset = useCallback(() => setState(initialState()), []);

  return { ...state, execute, reset };
}
```

## Key Rules

- `initialState` is a **factory function** `<T>(): ApiCallState<T>` — called on each state update to avoid shared object mutation
- `fnRef` pattern: store `apiFn` in a ref and update `fnRef.current = apiFn` on every render — this keeps `execute` stable (`useCallback` with `[]` deps) while always calling the latest version of `apiFn`
- `execute` has zero dependencies (`[]`) because it reads from `fnRef.current` at call time, not from the closure
- `execute` spreads `initialState()` before each state update to reset other fields cleanly
- `execute` returns `T | null` — returns `null` on error (caller can ignore or handle)
- Errors are always wrapped via `ApiError.from(err)` — never raw unknown errors
- `reset()` clears all state back to initial — use on unmount or navigation

## Usage in a Component

```typescript
import { useApiCall } from 'src/hooks/useApiCall';
import { api } from 'src/api';

// Basic usage
const { execute, data, isLoading, error } = useApiCall(api.machinesService.getAllMachines);

// With rename for clarity
const { execute: fetchMachine, data: machineData, isLoading } = useApiCall(
  api.machinesService.getMachineDetailsByStoreId
);

// Call in effect or event handler
useEffect(() => {
  fetchMachine(storeId);
}, [storeId]);

// Call with reset on unmount
useEffect(() => {
  return () => reset();
}, []);
```
