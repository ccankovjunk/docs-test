# Internationalization (in Luna 1)

a.k.a. i18n, localization, translation

See [internationalization.md](internationalization.md) for information on internationalizing in Luna2 and LunaUi.

## Quick start

In Luna1 (javascript) files, the `tx` function is used like so:

```
import tx from "luna/common/localization/tx";

DIV({}, [
    tx("Hello, World!"):
])
```

This `tx` function is identical to the `LocalizationService.tx` function used
in Luna Ui. See [internationalization.md](internationalization.md)
for more information on how to use `tx` and for best practices.
