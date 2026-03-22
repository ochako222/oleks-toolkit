# Template: api-axios-config

File: `frontend/src/api/axios.config.ts`

```typescript
import axios, { type AxiosError, type AxiosInstance } from 'axios';
import { store } from 'src/app/store';
import { logout, updateAccessToken } from 'src/app/store/authSlice';

export interface BaseResponse {
  message: string;
  status: string;
}

export class ApiError extends Error {
  readonly status: string;
  readonly statusCode: number;
  readonly originalError: AxiosError;

  private constructor(
    message: string,
    status: string,
    statusCode: number,
    originalError: AxiosError
  ) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.statusCode = statusCode;
    this.originalError = originalError;
  }

  static from(error: unknown): ApiError {
    if (error instanceof ApiError) return error;

    const axiosError = error as AxiosError<{ message?: string }>;
    const message =
      axiosError.response?.data?.message ??
      axiosError.message ??
      'Unknown error';
    const status = axiosError.response?.statusText ?? 'Error';
    const statusCode = axiosError.response?.status ?? 0;

    return new ApiError(message, status, statusCode, axiosError);
  }

  get isUnauthorized(): boolean {
    return this.statusCode === 401;
  }

  get isNotFound(): boolean {
    return this.statusCode === 404;
  }

  get isServerError(): boolean {
    return this.statusCode >= 500;
  }
}

export const apiClient: AxiosInstance = axios.create({
  baseURL: '/api',
  withCredentials: true
});

apiClient.interceptors.request.use((config) => {
  const token = store.getState().auth.access_token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  (res) => res,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const refreshRes = await axios.post(
          '/api/refresh-token',
          {},
          {
            withCredentials: true
          }
        );
        const newToken = refreshRes.data.data.access_token;
        store.dispatch(updateAccessToken(newToken));
        originalRequest.headers.Authorization = `Bearer ${newToken}`;

        return axios(originalRequest);
      } catch (e) {
        store.dispatch(logout());
        window.location.href = '/login';
        return Promise.reject(e);
      }
    }

    return Promise.reject(error);
  }
);
```

## Key Rules

- `BaseResponse` is declared here, not in `index.ts`
- `ApiError` uses `private constructor` — it can ONLY be created via `ApiError.from(error)`
- `ApiError.from()` short-circuits if the error is already an `ApiError` instance
- `apiClient` uses a relative `/api` base URL — never hardcode absolute URLs
- The `_retry` flag prevents infinite refresh loops on persistent 401s
- On refresh failure: dispatch `logout()` and redirect to `/login`
