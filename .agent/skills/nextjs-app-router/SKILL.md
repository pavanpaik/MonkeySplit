---
name: "Next.js App Router"
description: "Next.js 14 App Router patterns and conventions"
---

# Next.js App Router Skill

## File Structure
```
app/
├── (auth)/           # Auth route group
├── (app)/            # Authenticated routes
├── api/              # API routes
└── actions/          # Server actions
```

## Server Components (Default)
```typescript
// No 'use client' directive = Server Component
export default async function Page() {
  const data = await db.query.items.findMany();
  return <List items={data} />;
}
```

## Client Components
```typescript
'use client';
import { useState } from 'react';

export function InteractiveComponent() {
  const [state, setState] = useState('');
  return <button onClick={() => setState('clicked')}>{state}</button>;
}
```

## Server Actions
```typescript
// app/actions/example.ts
'use server';
import { z } from 'zod';
import { auth } from '@/lib/auth';
import { revalidatePath } from 'next/cache';

const schema = z.object({ name: z.string().min(1) });

export async function createItem(formData: FormData) {
  const session = await auth();
  if (!session?.user) throw new Error('Unauthorized');
  
  const data = schema.parse({ name: formData.get('name') });
  await db.insert(items).values(data);
  revalidatePath('/items');
  return { success: true };
}
```

## Route Handlers
```typescript
// app/api/items/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(req: NextRequest) {
  return NextResponse.json({ items: [] });
}
```

## Loading & Error States
```typescript
// loading.tsx — Shown during data fetching
export default function Loading() {
  return <Skeleton />;
}

// error.tsx — Shown on error
'use client';
export default function Error({ reset }: { reset: () => void }) {
  return <button onClick={reset}>Retry</button>;
}
```
