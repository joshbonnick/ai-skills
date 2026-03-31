# Performance Best Practices

There are mainly two aspects that developers usually consider regarding the performance of any computer programs: the
time and the memory consumption, both of which obviously depends on the specs of the hardware deck.gl is ultimately
running on.

On 2015 MacBook Pros with dual graphics cards, most basic layers (like ScatterplotLayer) renders fluidly at 60 FPS
during pan and zoom operations up to about 1M (one million) data items, with framerates dropping into low double
digits (10-20FPS) when the data sets approach 10M items.

Even if interactivity is not an issue, browser limitations on how big chunks of contiguous memory can be allocated (e.g.
Chrome caps individual allocations at 1GB) will cause most layers to crash during GPU buffer generation somewhere
between 10M and 100M items. You would need to break up your data into chunks and use multiple deck.gl layers to get past
this limit.

Modern phones (recent iPhones and higher-end Android phones) are surprisingly capable in terms of rendering performance,
but are considerably more sensitive to memory pressure than laptops, resulting in browser restarts or page reloads. They
also tend to load data significantly slower than desktop computers, so some tuning is usually needed to ensure a good
overall user experience on mobile.

### Layer Update Performance

Layer update happens when the layer is first created, or when some layer props change. During an update, deck.gl may
load necessary resources (e.g. image textures), generate GPU buffers, and upload them to the GPU, all of which may take
some time to complete, depending on the number of items in your data prop. Therefore, the key to performant deck.gl
applications is to minimize layer updates wherever possible.

#### Minimize data changes

When the data prop changes, the layer will recalculate all of its GPU buffers. The time required for this is
proportional to the number of items in your data prop. This step is the most expensive operation that a layer does -
also on CPU - potentially affecting the responsiveness of the application. It may take multiple seconds for
multi-million item layers, and if your data prop is updated frequently (e.g. animations), "stutter" can be visible even
for layers with just a few thousand items.

Some good places to check for performance improvements are:

##### Avoid unnecessary shallow change in data prop

The layer does a shallow comparison between renders to determine if it needs to regenerate buffers. If nothing has
changed, make sure you supply the same data object every time you render. If the data object has to change shallowly
for some reason, consider using the dataComparator prop to supply a custom comparison logic.

###### Bad practice

```typescript
function render(settings: Settings) {
    const layers = [
        new ScatterplotLayer<DataType>({
            // `filter` creates a new array every time `render` is called, even if minTime/maxTime have not changed
            data: DATA.filter(d => d.time >= settings.minTime && d.time <= settings.maxTime),
            getPosition: (d: DataType) => d.position,
            getRadius: settings.radius
        })
    ];

    deckInstance.setProps({layers});
}
```

###### Good practice

```typescript
let filteredData: DataType[];
let lastSettings: Settings;

function render(settings: Settings) {
    if (!lastSettings ||
        settings.minTime !== lastSettings.minTime ||
        settings.maxTime !== lastSettings.maxTime) {
        filteredData = DATA.filter(d => d.time >= settings.minTime && d.time <= settings.maxTime);
    }
    lastSettings = settings;

    const layers = [
        new ScatterplotLayer<DataType>({
            data: filteredData,
            getPosition: (d: DataType) => d.position,
            getRadius: settings.radius
        })
    ];

    deckInstance.setProps({layers});
}
```

##### Use updateTriggers

So data has indeed changed. Do we have an entirely new collection of objects? Or did just certain fields changed in each
row? Remember that changing data will update all buffers, so if, for example, object positions have not changed, it will
be a waste of time to recalculate them.

##### Favor layer visibility over addition and removal

Removing a layer will lose all of its internal states, including generated buffers. If the layer is added back later,
all the GPU resources need to be regenerated again. In the use cases where layers need to be toggled frequently (e.g.
via a control panel), there might be a significant perf penalty:

###### Bad practice

```typescript
function render(layerVisibility: {
    circles: boolean;
    labels: boolean;
}) {
    const layers = [
        // when visibility goes from on to off to on, this layer will be completely removed and then regenerated
        layerVisibility.circles && new ScatterplotLayer({
            id: 'circles'
            // ...
        }),
        layerVisibility.labels && new TextLayer({
            id: 'labels',
            //...
        })
    ];

    deckInstance.setProps({layers});
}
```

The visible rop is a cheap way to temporarily hide a layer:

###### Good practice

```typescript
function render(layerVisibility: {
    circles: boolean;
    labels: boolean;
}) {
    const layers = [
        // when visibility is off, this layer's internal states will be retained in memory, making turning it back on instant
        new ScatterplotLayer({
            id: 'circles',
            visible: layerVisibility.circles,
            // ...
        }),
        new TextLayer({
            id: 'labels',
            visible: layerVisibility.labels,
            // ...
        })
    ];

    deckInstance.setProps({layers});
}
```

##### Optimize Accessors

99% of the CPU time that deck.gl spends in updating buffers is calling the accessors you supply to the layer. Since they
are called on every data object, any performance issue in the accessors is amplified by the size of your data.

##### Favor constants over callback functions

Most accessors accept constant values as well as functions. Constant props are extremely cheap to update in comparison.
Use `ScatterplotLayer` as an example, the following two prop settings yield exactly the same visual outcome:

- `getFillColor: [255, 0, 0, 128]` - deck.gl uploads 4 numbers to the GPU.
- `getFillColor: d => [255, 0, 0, 128]` - deck.gl first builds a typed array of `4 * data.length` elements, call the
  accessor `data.length` times to fill it, then upload it to the GPU.

Aside from accessors, most layers also offer one or more `*Scale` props that are uniform multipliers on top of the
per-object value. Always consider using them before invoking the accessors.

##### Common Issues

A couple of particular things to watch out for that tend to have a big impact on performance:

* If not needed disable Retina/High DPI rendering. It generates 4x the number of pixels (fragments) and can have a big
  performance impact that depends on which computer or monitor is being used. This feature can be controlled using
  `useDevicePixels` prop of `DeckGL` component and it is on by default.
* Avoid using luma.gl debug mode in production. It queries the GPU error status after each operation which has a big
  impact on performance.

##### Smaller considerations:

* Enabling picking can have a small performance penalty so make sure the `pickable` property is `false` in layers that
  do not need picking (this is the default value).
