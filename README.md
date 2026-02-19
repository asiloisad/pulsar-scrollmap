# scrollmap

Show markers on the scroll bar. Core package providing scrollmap infrastructure for text editors and custom panes.

![demo](https://github.com/asiloisad/pulsar-scrollmap/blob/master/assets/demo.png?raw=true)

## Features

- **Layer system**: Multiple packages can add markers to the scrollbar.
- **3-column layout**: Markers positioned in left, center, or right columns.
- **Cross-platform**: Automatically adapts to scrollbar width on Windows, macOS, and Linux.
- **Toggle panel**: Enable/disable layers individually.
- **Simplemap API**: Support for non-editor panes like PDF viewer.
- **Extensible**: Other packages provide layers via the `scrollmap` service.

## Installation

To install `scrollmap` search for [scrollmap](https://web.pulsar-edit.dev/packages/scrollmap) in the Install pane of the Pulsar settings or run `ppm install scrollmap`. Alternatively, you can run `ppm install asiloisad/pulsar-scrollmap` to install a package directly from the GitHub repository.

## Provided Service `scrollmap`

Allows other packages to add custom marker layers to the scrollbar. Each layer provider returns a descriptor with initialization and item-fetching callbacks.

In your `package.json`:

```json
{
  "providedServices": {
    "scrollmap": {
      "versions": { "1.0.0": "provideScrollmap" }
    }
  }
}
```

In your main module:

```javascript
provideScrollmap() {
  return {
    name: "mylayer",
    description: "My layer description",
    position: "left",
    initialize: ({ editor, disposables, update }) => {
      disposables.add(
        editor.onDidStopChanging(update),
      );
    },
    getItems: ({ editor }) => {
      return [
        { row: 10 },                  // basic marker
        { row: 20, cls: "special" },  // with extra class
      ];
    },
  };
}
```

### Provider properties

| Property | Type | Description |
| --- | --- | --- |
| `name` | string | Layer name (CSS class: `marker-{name}`) |
| `description` | string | Layer description shown in toggle panel (optional) |
| `position` | string | Position class e.g. `left`, `right` (optional) |
| `timer` | number | Throttle interval in ms (default: 50) |
| `initialize` | function | `({ editor, cache, disposables, update }) => void` - set up layer |
| `getItems` | function | `({ editor, cache }) => items[]` - return markers to render |

### Marker item properties

| Property | Type | Description |
| --- | --- | --- |
| `row` | number | Screen row for the marker |
| `cls` | string | Additional CSS class (optional) |

## Provided Service `simplemap`

Provides a scrollbar widget for non-editor panes (like PDF viewer). Consumers receive a `Simplemap` constructor to create standalone scrollbar markers.

In your `package.json`:

```json
{
  "consumedServices": {
    "simplemap": {
      "versions": { "1.0.0": "consumeSimplemap" }
    }
  }
}
```

In your main module:

```javascript
consumeSimplemap(Simplemap) {
  const simplemap = new Simplemap();
  simplemap.setItems([
    { percent: 10, cls: "marker-h1" },
    { percent: 50, cls: "marker-h2" },
  ]);
  container.appendChild(simplemap.element);
  return new Disposable(() => simplemap.destroy());
}
```

## Customization

The scrollbar width is measured automatically and stored as CSS variables on `:root`:

| Variable | Description |
| --- | --- |
| `--scrollbar-width` | Measured scrollbar width (e.g. 10px on Windows, 8px on Linux) |
| `--scrollbar-bottom` | Bottom offset for horizontal scrollbar (0px for overlay scrollbars) |

Marker widths use percentages (40% center, 20% sides) to scale proportionally across platforms.

The style can be adjusted according to user preferences in the `styles.less` file:

- e.g. change marker width and opacity:

```less
.scrollmap .marker {
  width: 6px;
  opacity: 0.8;
}
```

- e.g. style specific layers:

```less
.scrollmap .marker-mylayer {
  background-color: @text-color-info;
}
```

## Contributing

Got ideas to make this package better, found a bug, or want to help add new features? Just drop your thoughts on GitHub — any feedback's welcome!
