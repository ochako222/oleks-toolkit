# Template: api-service-class

File: `frontend/src/api/<domain>.service.ts`

```typescript
import type { SharedType } from 'src/app/support/types';
import { apiClient, type BaseResponse } from './axios.config';

// ── Response interfaces ────────────────────────────────────────────────────────
// Interfaces only used in this service stay here.
// Interfaces shared across multiple services go in src/app/support/types.ts.

export interface ExampleListItem {
  id: number;
  name: string;
}

interface GetAllExamplesResponse extends BaseResponse {
  data: ExampleListItem[];
  counts: { Active: number; Inactive: number; Total: number };
}

interface GetExampleResponse extends BaseResponse {
  data: SharedType;
}

type UpdateExampleParams = Partial<Pick<SharedType, 'name' | 'status'>>;

// ── Service class ──────────────────────────────────────────────────────────────

class ExampleService {
  getAll = async (): Promise<GetAllExamplesResponse> => {
    const { data } = await apiClient.get('/examples');
    return data;
  };

  getById = async (id: string): Promise<GetExampleResponse> => {
    const { data } = await apiClient.get(`/examples/${id}`);
    return data;
  };

  create = async (params: UpdateExampleParams): Promise<BaseResponse> => {
    const { data } = await apiClient.post('/examples', params);
    return data;
  };

  update = async (id: string, params: UpdateExampleParams): Promise<BaseResponse> => {
    const { data } = await apiClient.post(`/update-example`, { id, ...params });
    return data;
  };
}

export const exampleService = new ExampleService();
```

## Key Rules

- Always import `apiClient` and `BaseResponse` from `'./axios.config'` — never from `'src/api'` or `'index.ts'`
- Use `type` keyword for type-only imports: `import type { Foo } from 'src/app/support/types'`
- All service methods are **arrow functions** assigned to class properties — this ensures correct `this` binding when methods are passed as callbacks to `useApiCall`
- Each method destructures `{ data }` from the Axios response and returns the typed result directly
- Response interfaces extend `BaseResponse` (`message: string; status: string`)
- Keep response interfaces **private** (no `export`) unless consumed outside the service file
- Export **param types** and **list item types** if consumed by components (e.g., form props, table columns)
- Class name: PascalCase + `Service` suffix → `MachinesService`, `BottlesService`
- Singleton export: camelCase → `export const machinesService = new MachinesService()`
- After creating a service, add it to `frontend/src/api/index.ts`:

```typescript
// index.ts
import { exampleService } from './example.service';

export const api = {
  // ...existing services
  exampleService,
};
```
