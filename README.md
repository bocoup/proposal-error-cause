# Error Cause

This proposal has not yet been presented to TC39 plenary meetings.

## Chaining Errors

Anytime runtime unexpected behavior occurs, an error can be constructed
with a detail message attached to it. Additionally, a stacktrace string will
be generated by JavaScript implementations to indicate where the error was
constructed.

Runtime unexpected behavior can be detected by comparing the actual evaluation
results to expected ones, if not matching, an Error will be constructed and
thrown. Detecting runtime unexpected behavior by catching error thrown by
implementations is another common approach. In that case, wrapping the caught
error instance with a newly constructed error instance is a fairly useful
method.

```js
async function getSolution() {
  const rawResource = await fetch('//domain/resource-a')
    .catch(err => {
      // How to wrap the error properly?
      // 1. throw new Error('Download raw resource failed: ' + err.message);
      // 2. const wrapErr = new Error('Download raw resource failed');
      //    wrapErr.cause = err;
      //    throw wrapErr;
      // 3. throw new CustomError('Download raw resource failed', err);
    })
  const jobResult = doComputationalHeavyJob(rawResource);
  await fetch('//domain/upload', { method: 'POST', body: jobResult });
}

await doJob(); // => TypeError: Failed to fetch
```

If the error were thrown from deep internal methods, the thrown error may not
be straightforward to be easily conducted without proper exception design
pattern. However, if the errors were chained with causes, it can be greatly
helpful to diagnosing unexpected exceptions.

The proposed solution is adding an additional parameter `cause` to the
`Error()` constructor, so that errors can be chained without unnecessary
and overelaborate formalities on wrapping the errors in conditions.

```js
async function doJob() {
  const rawResource = await fetch('//domain/resource-a')
    .catch(err => {
      throw new Error('Download raw resource failed', err);
    });
  const jobResult = doComputationalHeavyJob(rawResource);
  await fetch('//domain/upload', { method: 'POST', body: jobResult })
    .catch(err => {
      throw new Error('Upload job result failed', err);
    });
}

try {
  await doJob();
} catch (e) {
  console.log(e);
  console.log('Caused by', e.cause);
}
// Error: Upload job result failed
// Caused by TypeError: Failed to fetch
```

## Compatibilities

In Firefox, `Error()` constructor can receive two optional additional
positional parameters: `fileName`, `lineNumber`. Those parameters will be
assigned to newly constructed error instances with the name `fileName` and
`lineNumber` respectively.

However, no standard on either ECMAScript or Web were defined on such behavior.