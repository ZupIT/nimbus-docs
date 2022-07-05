# Specification
This series of documents are the official definition of how [Nimbus](/readme.md) should work as a protocol. Every implementation of Nimbus must
reflect the definitions written in here.

All examples in these documents use JSON or Typescript. We chose to use Typescript due to its easy of writing type definitions. These examples
also use components that are not part of the specification, consider them as components declared by the application.

Most of the components used in the examples, although not part of the specification, are implemented in our
[layout component library](/layout/index.md).

## Topics
- [Components](component.md)
  - [`forEach`](default-components/for-each.md)
  - [`if`, `then`, `else`](default-components/if.md)
- [State](state.md)
- [Operations](operation.md)
  - [Default operations](default-operations.md)
- [Expressions](expression.md)
- [Actions](action.md)
  - [Navigation actions](default-actions/navigation.md)
  - [`condition`](default-actions/condition.md)
  - [`log`](default-actions/log.md)
  - [`openUrl`](default-actions/open-url.md)
  - [`sendRequest`](default-actions/send-request.md)
  - [`setContent`](default-actions/log.md)
  - [`setState`](default-actions/set-state.md)
