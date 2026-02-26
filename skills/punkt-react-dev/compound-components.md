# Compound Components

For parent-child component relationships, use React Context:

```tsx
// Parent creates and provides context
const AccordionContext = createContext<{ name?: string }>({})
export const useAccordionContext = () => useContext(AccordionContext)

export const PktAccordion = ({ name, children, ref, ...props }: IPktAccordion) => (
  <AccordionContext.Provider value={{ name }}>
    <div ref={ref} {...props}>{children}</div>
  </AccordionContext.Provider>
)

// Child consumes context
export const PktAccordionItem = ({ name, ref, ...props }: IPktAccordionItem) => {
  const { name: contextName } = useAccordionContext()
  const actualName = name || contextName  // Props override context
  // ...
}
```
