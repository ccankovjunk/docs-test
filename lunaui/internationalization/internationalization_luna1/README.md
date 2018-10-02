# Internationalization in Luna1

a.k.a. i18n, localization, translation

See [internationalization.md](https://github.com/ccankovjunk/docs-test/tree/ec64d54dbe01f87d81c1fb709ba61401e6e9fe8b/lunaui/internationalization/README.md) for information on internationalizing in Luna2 and LunaUi.

## Quick start

In Luna1 \(javascript\) files, the `tx` function is used like so:

```text
import tx from "luna/common/localization/tx";

DIV({}, [
    tx("Hello, World!"):
])
```

This `tx` function is identical to the `LocalizationService.tx` function used in Luna Ui. See [internationalization.md](https://github.com/ccankovjunk/docs-test/tree/ec64d54dbe01f87d81c1fb709ba61401e6e9fe8b/lunaui/internationalization/README.md) for more information on how to use `tx` and for best practices.

