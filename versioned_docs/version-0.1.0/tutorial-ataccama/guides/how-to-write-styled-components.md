---
sidebar_position: 1
---

# How to write styled components

## Working with styled component props

If you need to pass some properties to a styled component, define an interface and pass it as a generic parameter.

```
import styled from '~/ui-styled'

interface MyButtonProps {
  primary?: boolean
}

export const MyButton = styled.button<MyButtonProps>`
  background: ${p => p.primary ? 'deeppink' : 'white'};
  // other styling
`
```

In case you need to set multiple style rules, based on incoming props wrap these into a `css` fragments:

```
import styled, { css } from '~/ui-styled'

interface MyButtonProps {
  primary?: boolean
}

const primary = css`
 background: deeppink;
 color: white;
`

const normal = css`
 background: white;
 color: black;
`

export const MyButton = styled.button<MyButtonProps>`
  ${p => p.primary ? primary : normal};
  // other styling
`
```

## Styled parts

If component requires some complex styling, all the styled elements should be contained within the `styled.ts` file next to the `index.tsx` file:

```
 /MyButton
   /index.tsx
   /styled.ts
```

Usage within the index file:

```
// always import parts like so to avoid any possible name collisions
import * as S from './styled'

export const Foo = ({ title, children }) => {
  return (
    <S.Container>
      <S.Title>{title}</S.Title>
      <S.Content>{children}</S.Content>
    <S.Container>
  )
}
```

### Naming

Naming convention within the `styled.ts` is pretty simple:

- root node of the component should be named `Container`;
- every other element should be named according to its meaning within the component;

For example, naming for a `List` component might look like so

```
import styled from '~/ui-styled'

const Container = styled.ul`
  // some list styling
`

// note that element is called Item, not ListItem
const Item = styled.li`
  // list item styling
`
```

### Style structure

Since `Container` is the root element of the component it should be free of any styles that affect its outer geometry (e.g. *margin*, _position: absolute_ and so on) to ensure reusability.

If any of the component elements requires an absolute positioning, `Container` should provide `position: relative`, to make sure elements styling isn’t “leaking”.

Apart from what was mentioned above, common sense is your best friend.
