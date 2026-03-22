---
name: frontend-portal-pattern
description: >
  Code patterns for React portals using createPortal. Covers portal root setup,
  BasePortal primitive, portal-based modal, toast/notification system, and tooltip.
  Used by the frontend-portal-builder skill.
type: template
---

# Frontend Portal Pattern

Reference code for building React portal-based overlay UI. All patterns follow the project's TypeScript + Ant Design + SCSS conventions (tabs, single quotes, semicolons, `src/` path alias).

---

## 1. Portal Root — `index.html`

Add a dedicated `#portal-root` div **after** `#root`. This is the single mount point for all portal overlays.

```html
<body>
  <div id="root"></div>
  <div id="portal-root"></div>
</body>
```

---

## 2. BasePortal — `src/app/components/BaseComponents/BasePortal.tsx`

The low-level primitive. All portal-based components use this.

```tsx
import { type ReactNode } from 'react';
import { createPortal } from 'react-dom';

interface BasePortalProps {
  children: ReactNode;
  target?: Element | DocumentFragment;
}

export const BasePortal = ({ children, target }: BasePortalProps) => {
  const portalRoot = target ?? document.getElementById('portal-root');

  if (!portalRoot) {
    console.warn('[BasePortal] #portal-root not found in DOM — portal skipped.');
    return null;
  }

  return createPortal(children, portalRoot);
};
```

---

## 3. Portal Modal — `src/app/components/BaseComponents/PortalModal.tsx`

Custom modal built on `BasePortal`. Use when Ant Design's `Modal` is not sufficient (e.g. full-screen overlays, deeply nested stacking contexts).

```tsx
import { type KeyboardEvent, type ReactNode, useEffect } from 'react';
import { BasePortal } from 'src/app/components/BaseComponents/BasePortal';
import 'src/app/styles/_portal-modal.scss';

interface PortalModalProps {
  open: boolean;
  onClose: () => void;
  children: ReactNode;
  title?: string;
  className?: string;
}

export const PortalModal = ({
  open,
  onClose,
  children,
  title,
  className,
}: PortalModalProps) => {
  // Lock body scroll while open
  useEffect(() => {
    if (open) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = '';
    }
    return () => {
      document.body.style.overflow = '';
    };
  }, [open]);

  // Close on Escape key
  useEffect(() => {
    if (!open) return;
    const handleKey = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };
    document.addEventListener('keydown', handleKey as unknown as EventListener);
    return () =>
      document.removeEventListener(
        'keydown',
        handleKey as unknown as EventListener,
      );
  }, [open, onClose]);

  if (!open) return null;

  return (
    <BasePortal>
      <div
        className="portal-modal-overlay"
        role="presentation"
        onClick={onClose}
      >
        <div
          className={`portal-modal-content${className ? ` ${className}` : ''}`}
          role="dialog"
          aria-modal="true"
          aria-label={title}
          onClick={(e) => e.stopPropagation()}
        >
          {title && <div className="portal-modal-title">{title}</div>}
          {children}
        </div>
      </div>
    </BasePortal>
  );
};
```

---

## 4. Toast System

### Context — `src/app/context/ToastContext.tsx`

```tsx
import {
  type ReactNode,
  createContext,
  useCallback,
  useContext,
  useId,
  useState,
} from 'react';
import { BasePortal } from 'src/app/components/BaseComponents/BasePortal';
import 'src/app/styles/_portal-toast.scss';

type ToastType = 'success' | 'error' | 'warning' | 'info';

interface Toast {
  id: string;
  message: string;
  type: ToastType;
}

interface ToastContextValue {
  showToast: (message: string, type?: ToastType) => void;
}

const ToastContext = createContext<ToastContextValue | null>(null);

export const ToastProvider = ({ children }: { children: ReactNode }) => {
  const [toasts, setToasts] = useState<Toast[]>([]);
  const baseId = useId();

  const showToast = useCallback(
    (message: string, type: ToastType = 'info') => {
      const id = `${baseId}-${Date.now()}`;
      setToasts((prev) => [...prev, { id, message, type }]);
      setTimeout(() => {
        setToasts((prev) => prev.filter((t) => t.id !== id));
      }, 4000);
    },
    [baseId],
  );

  return (
    <ToastContext.Provider value={{ showToast }}>
      {children}
      <BasePortal>
        <div className="portal-toast-container" role="status" aria-live="polite">
          {toasts.map((toast) => (
            <div key={toast.id} className={`portal-toast portal-toast--${toast.type}`}>
              {toast.message}
            </div>
          ))}
        </div>
      </BasePortal>
    </ToastContext.Provider>
  );
};

export const useToast = (): ToastContextValue => {
  const ctx = useContext(ToastContext);
  if (!ctx) throw new Error('useToast must be used inside <ToastProvider>');
  return ctx;
};
```

### Usage

```tsx
// Wrap at app root (e.g. main.tsx or App.tsx)
<ToastProvider>
  <App />
</ToastProvider>

// In any component
const { showToast } = useToast();
showToast('Saved successfully', 'success');
showToast('Something went wrong', 'error');
```

---

## 5. Portal Tooltip — `src/app/components/BaseComponents/PortalTooltip.tsx`

Positioned tooltip using `useLayoutEffect` + `getBoundingClientRect`. Use when Ant Design's `Tooltip` cannot be used (e.g. inside scroll containers with `overflow: hidden`).

```tsx
import {
  type ReactNode,
  useCallback,
  useLayoutEffect,
  useRef,
  useState,
} from 'react';
import { BasePortal } from 'src/app/components/BaseComponents/BasePortal';
import 'src/app/styles/_portal-tooltip.scss';

interface PortalTooltipProps {
  content: ReactNode;
  children: ReactNode;
}

interface TooltipPosition {
  top: number;
  left: number;
}

export const PortalTooltip = ({ content, children }: PortalTooltipProps) => {
  const triggerRef = useRef<HTMLSpanElement>(null);
  const tooltipRef = useRef<HTMLDivElement>(null);
  const [visible, setVisible] = useState(false);
  const [pos, setPos] = useState<TooltipPosition>({ top: 0, left: 0 });

  const updatePosition = useCallback(() => {
    if (!triggerRef.current || !tooltipRef.current) return;
    const triggerRect = triggerRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();
    setPos({
      top: triggerRect.top - tooltipRect.height - 8 + window.scrollY,
      left:
        triggerRect.left +
        triggerRect.width / 2 -
        tooltipRect.width / 2 +
        window.scrollX,
    });
  }, []);

  useLayoutEffect(() => {
    if (visible) updatePosition();
  }, [visible, updatePosition]);

  return (
    <>
      <span
        ref={triggerRef}
        onMouseEnter={() => setVisible(true)}
        onMouseLeave={() => setVisible(false)}
        onFocus={() => setVisible(true)}
        onBlur={() => setVisible(false)}
      >
        {children}
      </span>
      {visible && (
        <BasePortal>
          <div
            ref={tooltipRef}
            className="portal-tooltip"
            role="tooltip"
            style={{ top: pos.top, left: pos.left }}
          >
            {content}
          </div>
        </BasePortal>
      )}
    </>
  );
};
```

---

## 6. Integration in `main.tsx`

Wrap providers at the root. `ToastProvider` must be inside `Redux Provider` so it can access the store if needed.

```tsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import App from './App';
import { ToastProvider } from './app/context/ToastContext';
import { store } from './app/store';

createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter>
      <Provider store={store}>
        <ToastProvider>
          <App />
        </ToastProvider>
      </Provider>
    </BrowserRouter>
  </React.StrictMode>,
);
```

---

## 7. Decision: Custom Portal vs Ant Design Built-in

| Scenario | Use |
|----------|-----|
| Standard modal with form, confirm, title/footer | **Ant Design `Modal`** (already uses portals internally) |
| Full-screen overlay, complex stacking context, custom animation | **`PortalModal`** with `BasePortal` |
| Standard tooltip on non-clipped element | **Ant Design `Tooltip`** |
| Tooltip inside `overflow: hidden` scroll container | **`PortalTooltip`** with `BasePortal` |
| App-wide toast/notification from any component | **`ToastProvider` + `useToast`** |
| One-off inline notification within a component | **Ant Design `Alert`** |

---

## Key Rules

- **Always use `#portal-root`** — never render portals into `document.body` directly; use `BasePortal`
- **Stop event propagation on modal content** — prevent overlay click from bubbling to portal content
- **Lock body scroll** when a modal is open (`document.body.style.overflow = 'hidden'`)
- **Handle Escape key** for keyboard accessibility
- **Use `role="dialog"` + `aria-modal="true"`** on custom modals; `role="tooltip"` on tooltips; `role="status"` + `aria-live="polite"` on toast containers
- **No inline `style={{}}`** on portal components — all styles go in dedicated SCSS files under `src/app/styles/`
- **Colors from `_variables.scss`** — never hardcode hex values in portal components
