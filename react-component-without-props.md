# Using TypeScript Intersections to extend component's props in React

## 1. Every app has buttons.

So, let's create a Button component, which uses the button html element under the hood.

```tsx
export default function Button({ children }: { children: React.ReactNode }) {
  return <button className="btn">{children}</button>;
}
```

Easy peasy.

But in terms of functionality, our button is very limited. For example, having this implementation, we cannot handle any events, cannot extend the `className`, nor pass any `style`, `disabled`, etc. props to it. It's basically a very dumb component.

There are several possible approaches we can take in order to make it a useful one.

## 2. Props "babysitting" or any type

One way to do it is by "babysitting" every possible button property there is by ourselves. Like this:

```tsx
export default function Button({
  children,
  onClick,
  style,
  disabled,
  // ... etc.
}: {
  children: React.ReactNode
  onClick: (e: MouseEvent) => void }
  style: { [key: string]: any
  disabled: boolean
  // ... etc.
}) {
  return (
    <button
      className="btn"
      onClick={onClick}
      style={style}
    >
      {children}
    </button>
  );
}
```

We can list all of native `button`'s properties in our `Button`' s props, or some of them, but that's very tedious and far from scalable.

Another option is to use the `any` type and tell TypeScript that our component doesn't care about what types are the props it receives. I'm not going into details here, because `any` defeats the purpose of using TypeScript in the first place. And on top of that, using it is a strong sign for being lazy and we're not lazy. :)

So, there's a smarter approach.

## 3. Using `...rest` with TypeScript's intersections.

```tsx
export default function Button({
  children,
  ...rest
}: {
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<"button">) {
  return (
    <button className="btn" {...rest}>
      {children}
    </button>
  );
}
```

Using the JavaScript's `...rest` operator, we collect every possible property the user of the component passes to it in an array, called `rest`.

After that we "spread" the array in the `<button>`, making our Button component act as a "proxy".

One thing that's important to notice is the intersection type `{ ... }` & `React.ComponentPropsWithoutRef<'button'>`.

This is how we tell TypeScript that our Button component may receive any of the native button's properties through the `...rest`.

Sweet. We are happy. TypeScript is happy. 🎉

But we're not constrained only to extending props using only native HTML elements. We can do it with custom ones, too.

Here's an example, where we create an `IconButton`, which composes our `Button` component, but adds the ability to display an icon, next to the button's content. Notice that in this case we should use `typeof Component` to indicate that we're using a custom one.

```tsx
export default function IconButton({
  icon,
  children,
  ...rest
}: {
  icon: React.ReactNode;
} & React.ComponentPropsWithoutRef<typeof Button>) {
  return (
    <Button {...rest}>
      <Icon name="add" />
      {children}
    </Button>
  );
}
```

## 4. Conclusion

Use `React.ComponentPropsWithoutRef<type>` with intersections to compose custom component types. Forget about `any` and props "babysitting". Be smart and reuse whenever possible, whatever possible.

Here's very simple demo: https://codesandbox.io/s/morning-cache-3vzb4?file=/src/App.tsx

Source: [https://dev.to/brslv/using-typescript-intersections-to-extend-component-s-props-in-react-2omg](https://dev.to/brslv/using-typescript-intersections-to-extend-component-s-props-in-react-2omg)
