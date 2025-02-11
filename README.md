# next-navigation-guard

You use Next.js, and you want to show "You have unsaved changes that will be lost." dialog when user leaves page?
This library is just for you!

## Demo

[https://layerxcom.github.io/next-navigation-guard/](https://layerxcom.github.io/next-navigation-guard/)

## How does it work?

- [English Slide](https://speakerdeck.com/ypresto/cancel-next-js-page-navigation-full-throttle)
- [Japanese Slide](https://speakerdeck.com/ypresto/hack-to-prevent-page-navigation-in-next-js)

## Installation

```bash
npm install next-navigation-guard
# or
yarn install next-navigation-guard
# or
pnpm install next-navigation-guard
```

- App Router: app/layout.tsx

  ```tsx
  <html lang="en">
    <body className={`${geistSans.variable} ${geistMono.variable}`}>
      <NavigationGuardProvider>{children}</NavigationGuardProvider>
    </body>
  </html>
  ```

- Page Router: page/_app.tsx

  ```tsx
  export default function MyApp({ Component, pageProps }: AppProps) {
    return (
      <NavigationGuardProvider>
        <Component {...pageProps} />
      </NavigationGuardProvider>
    );
  }
  ```

## Usage

- window.confirm()

  ```tsx
  useNavigationGuard({ enabled: form.changed, confirm: () => window.confirm("You have unsaved changes that will be lost.") })
  ```

- Custom dialog component

  ```tsx
  const navGuard = useNavigationGuard({ enabled: form.changed })

  return (
    <>
      <YourContent />

      <Dialog open={navGuard.active}>
        <DialogText>You have unsaved changes that will be lost.</DialogText>

        <DialogActions>
          <DialogButton onClick={navGuard.reject}>Cancel</DialogButton>
          <DialogButton onClick={navGuard.accept}>Discard</DialogButton>
        </DialogActions>
      </Dialog>
    </>
  )
  ```

  Note that `navGuard.active` is only set for client-side navigation. If the user attempts to close the tab or
  manually navigates to a new URL, the navigation guard will fallback to the browser's default confirmation
  dialog. This is an inherent limitation of modern browsers.

- Hooking into the library's underlying state

  This is an advanced use-case if, for example, you have a special routing setup that prevents you from using
  the built-in NextJS router or Link component.

  ```tsx
  import { NavigationGuardProviderContext } from 'next-navigation-guard';

  function SpecialLink({ href }: { href: string }) {
    const guardMapRef = useContext(NavigationGuardProivderContext);
    let guardNavigation: React.MouseEventHandler<HTMLAnchorElement> | undefined = undefined;
  
    for (const guard of guardMapRef.current.values()) {
      const { enabled, callback } = guard;
      if (!enabled({ to: href, type: 'push' })) continue;
  
      guardNavigation = (e: React.MouseEvent<HTMLAnchorElement>) => {
        e.preventDefault();
        e.stopPropagation();
  
        let confirmed = callback({ to: href, type: 'push' });
        if (typeof confirmed === 'boolean') {
          confirmed = Promise.resolve(confirmed);
        }
  
        void confirmed.then((confirmed) => {
          if (!confirmed) return;
  
          guard.enabled = () => false;
          window.location.href = href;
        });
      };
      break;
    }
  
    return <a href="/" onClick={guardNavigation}>Click Here</a>;
  }
  ```

See working example in example/ directory and its `NavigationGuardToggle` component.
