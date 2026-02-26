# Lit Wrapper Components

For components that wrap web components from `punkt-elements`:

```tsx
'use client'
import { createComponent } from '@lit/react'
import { PktIcon as PktEl } from '@oslokommune/punkt-elements'
import React, { FC } from 'react'
import { PktElConstructor, PktElType } from '@/interfaces/IPktElements'

interface IPktIcon extends PktElType {
  name?: string
}

export const PktIcon: FC<IPktIcon> = createComponent({
  tagName: 'pkt-icon',
  elementClass: PktEl as PktElConstructor<HTMLElement>,
  react: React,
  displayName: 'PktIcon',
  events: {},
})
```

Note: Lit wrappers are the one exception where `React` is imported as a value (required by `createComponent`).
