# Layout
The Nimbus specification doesn't declare any UI component, for this reason we need additional libraries to add some basic functionalities to our
Server Driven UI application. This library adds layout components to Nimbus, i.e. components that help organize other components in the screen.

# Installing

## AndroidAdd to your gradle file, under dependencies:
```
implementation "br.com.zup.nimbus:nimbus-layout-compose:$VERSION"
```

where `$VERSION` is the latest version.

## iOS
todo: publish

# Image Provider
This class is used to provide local images to the component "layout:image". See the examples below of how to implement and set it up.

## Android
todo

## iOS
todo

# Components
All components of this lib are prefixed by `layout:`. We don't have yet a full documentation for the component library, but everything you need can be found in our specification: https://gist.github.com/Tiagoperes/694516d05bf989cb0a222dd03c8f4537

In the specification, every exported type represent a component.
