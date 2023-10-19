# Mercury

## Motivations

Currently in WebF, it is required that the runtime be initialized within the top level of the `Scaffold` of an active app, because it is assumed that the UI is primarily being built with web. However, this imposes a quite large restriction on the usecase of this library; it cannot be used to simply provide a full JavaScript runtime.

The existing alternatives for javascript runtimes in Flutter do not provide the same W3C compatibility that this library does, making this a good potential option for those seeking to drive a javascript backend.

## Design

### Polyfills
- `console`
- `Event` (`EventTarget`, `CustomEvent`)
- `fetch`
- `URLSearchParams`
- `URL`
- `setTimeout`, `clearTimeout`, `setInterval`
- `WebSocket` 
- `XHR`

### Features
- `MercuryRuntimeChannel` / `mercury` Runtime Channel (dart<->javascript bridge)
  ```dart
  Mercury(
    bundlePath:'assets/bundle.js',
    runtimeChannel: MercuryRuntimeChannel(),
  )
  ```
  ```js
  mercury.runtimeChannel
    .invokeMethod('somemethod', 'parameter 1', ['parameter 2'], {
        value: 'parmeter 3',
    })
    .then(result => {
        console.log('data from native', result);
    })
    .catch(err => {
        console.log('some error occured', err);
    });
  ```

### Extensions
- Add default domain for web requests
- Add built-in events
- Add to the global
- Add polyfills